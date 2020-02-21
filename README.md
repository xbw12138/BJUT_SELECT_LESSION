# 北工大研究生抢课助手

## 研究生管理平台
[http://webrecdoc.bjut.edu.cn/pyxx/login.html](http://webrecdoc.bjut.edu.cn/pyxx/login.html)

## 使用方法

在代码中修改对应的配置，
填写学号、密码、钉钉通知、课程（课程代码，星期几的课）
钉钉通知没有可以不设置

## 代码

```
# -*- coding: utf-8 -*- 
import requests
import re
import time
import _thread


def request_by_curl(webhook, data_string):
    headers = {
        "Content-Type": "application/json;charset=utf-8"
    }
    resp = requests.post(url=webhook, headers=headers, data=data_string, timeout=10, verify=True)
    return resp


def push(details, webhook):
    message = "抢课成功\n" + details
    data = {'msgtype': 'text', 'text': {'content': message}}
    data_string = json.dumps(data)
    request_by_curl(webhook, data_string)


def getCookie(username, password):
    url = "http://webrecdoc.bjut.edu.cn/pyxx/MyService.ashx?callback=jQuery17204863061010193702_1534667792635&username=" + username + "&password=" + password + "&_=1534667800673"
    res = requests.get(url)
    cookies = requests.utils.dict_from_cookiejar(res.cookies)
    return cookies['ASP.NET_SessionId']


def getSelectKe(cookie):
    url = "http://webrecdoc.bjut.edu.cn/pyxx/pygl/pyjhxk.aspx"
    header = {
        'Referer': 'http://webrecdoc.bjut.edu.cn/pyxx/leftmenu.aspx',
        'Cookie': 'ASP.NET_SessionId=' + cookie
    }
    res = requests.get(url, headers=header)
    return res.text


def observeKe(cookie, content, kechengcode, keyword):
    isMatchedinput = re.findall(r'<input type="hidden" name="(.*?)" id="(.*?)" value="(.*?)" />', content)
    __VIEWSTATE = isMatchedinput[0][2]
    __VIEWSTATEGENERATOR = isMatchedinput[1][2]
    isMatchedtr = re.findall(r'<tr (.*?)>(.*?)</tr>', content, re.S)
    if isMatchedtr:
        for tr in isMatchedtr:
            if kechengcode in tr[1] and keyword in tr[1]:
                isMatchedtd = re.findall(r'<td (.*?)>(.*?)</td>', tr[1], re.S)
                if isMatchedtd:
                    if int(isMatchedtd[3][1]) > int(isMatchedtd[4][1]):
                        print("有课啦～～～" + kechengcode)
                        if isMatchedinput:
                            isMatchedbutton = re.findall(r';(.*?)&', isMatchedtd[8][1], re.S)
                            if isMatchedbutton:
                                __EVENTTARGET = isMatchedbutton[0]
                                postData = {
                                    '__VIEWSTATEGENERATOR': __VIEWSTATEGENERATOR,
                                    '__EVENTTARGET': __EVENTTARGET,
                                    '__EVENTARGUMENT': '',
                                    '__VIEWSTATE': __VIEWSTATE
                                }
                                header = {
                                    'Referer': 'http://webrecdoc.bjut.edu.cn/pyxx/pygl/pyjhxk.aspx',
                                    'Cookie': 'ASP.NET_SessionId=' + cookie
                                }
                                url = 'http://webrecdoc.bjut.edu.cn/pyxx/pygl/pyjhxk.aspx'
                                res = requests.post(url, headers=header, data=postData)
                                print(res.text)
                                return True
                    else:
                        print("再等待～～" + kechengcode)
                        return False


def selectKebiao(username, password, webhook, kechengcode, keyword):
    cookie = getCookie(username, password)
    count = 1
    if cookie:
        while 1:
            content = getSelectKe(cookie)
            kebiao = observeKe(cookie, content, kechengcode, keyword)
            if kebiao:
                push(kechengcode + " -- " + keyword, webhook)
                break
            time.sleep(1)
            count = count + 1
        print("共抢课" + str(count) + "次")
    else:
        return "not found"


if __name__ == "__main__":
    username = "S201861847"  # 学号
    password = "123456"  # 密码
    webhook = "https://oapi.dingtalk.com/robot/send?access_token="  # 钉钉通知
    # 课程代码，周几的课
    lession = [
        ["2180616028", "星期二"],
        ["2140256011", "星期五"],
        ["2140256010", "星期二"],
        ["5140072003", "星期一"],
        ["2140256004", "星期五"],
        ["2140256016", "星期四"]
    ]

    for item in lession:
        try:
            _thread.start_new_thread(selectKebiao, (username, password, webhook, item[0], item[1],))
        except:
            print("Error: unable to start thread")

    while 1:
        pass

```

