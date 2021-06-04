#### `CURL`
详情参考 [curl](https://catonmat.net/cookbooks/curl)

`get`请求
```
curl https://catonmat.net
```
请求之后保存响应信息到文件
```
curl -o response.txt https://catonmat.net
```
发送一个空参数的 `post` 请求
```
curl -X POST https://catonmat.net
```
 `Form` 表单数据的 `POST` 请求
```
curl -d 'login=emma&password=123' -X POST https://google.com/login
```
`JSON` 数据的 `POST` 请求
```
curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type:application/json' https://google.com/login
```
自己搭建了一个 `SpringBoot` 项目，尝试用 curl 发送 POST 请求，但是调不通，报 `SEC_E_INVALID_TOKEN` 。
```
 url -H 'Content-type:application/json' -d '{"id":1,"name":"kangpan"}' https://localhost:8080/
curl: (35) schannel: next InitializeSecurityContext failed: SEC_E_INVALID_TOKEN (0x80090308) - 给函数提供的标志无效
```
<!--more-->
查看本机 `curl`,猜想是不是使用了某种协议，需要权限才能访问
```
curl -V
curl 7.55.1 (Windows) libcurl/7.55.1 WinSSL
Release-Date: [unreleased]
Protocols: dict file ftp ftps http https imap imaps pop3 pop3s smtp smtps telnet tftp
Features: AsynchDNS IPv6 Largefile SSPI Kerberos SPNEGO NTLM SSL
```
#### `HTTPie` 安装步骤
没有找到解决办法，那么就换一个工具吧 `httpie` [https://github.com/jkbr/httpie](https://github.com/jkbr/httpie)
查看文档 [https://httpie.io/docs#installation](https://httpie.io/docs#installation)
```
Windows, etc.
A universal installation method (that works on Windows, Mac OS X, Linux, …, and always provides the latest version) is to use pip:

# Make sure we have an up-to-date version of pip and setuptools:
python -m pip install --upgrade pip setuptools

python -m pip install --upgrade httpie
(If pip installation fails for some reason, you can try easy_install httpie as a fallback.)

Python version
Python version 3.6 or greater is required.
```
大致是说，需要安装 `Python`，配置`Python`环境变量，再使用 `Python` 安装 `pip`，设置 `pip`环境变量，再使用 `pip` 安装 `httpie`
进入 `Python` 官方下载地址 [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)
选择安装的文件 `Windows installer (64-bit)`
#### `Python` 安装
```
Files
Version	Operating System	Description	MD5 Sum	File Size	GPG
Gzipped source tarball	Source release		cc8507b3799ed4d8baa7534cd8d5b35f	25411523	SIG
XZ compressed source tarball	Source release		2a3dba5fc75b695c45cf1806156e1a97	18900304	SIG
macOS 64-bit Intel installer	Mac OS X	for macOS 10.9 and later	2b974bfd787f941fb8f80b5b8084e569	29866341	SIG
macOS 64-bit universal2 installer	Mac OS X	for macOS 10.9 and later, including macOS 11 Big Sur on Apple Silicon (experimental)	9aa68872b9582c6c71151d5dd4f5ebca	37648771	SIG
Windows embeddable package (32-bit)	Windows		b4bd8ec0891891158000c6844222014d	7580762	SIG
Windows embeddable package (64-bit)	Windows		5c34eb7e79cfe8a92bf56b5168a459f4	8419530	SIG
Windows help file	Windows		aaacfe224768b5e4aa7583c12af68fb0	8859759	SIG
Windows installer (32-bit)	Windows		b790fdaff648f757bf0f233e4d05c053	27222976	SIG
Windows installer (64-bit)	Windows	Recommended	ebc65aaa142b1d6de450ce241c50e61c	28323440	SIG
```
安装之后设置环境变量 打开`CMD`窗口，输入命令，`C:\Python\Python39` 安装的路径
```
path=%path%;C:\Python\Python39
```
查看环境变量是否设置成功，`CMD` 输入 `python`
```
Python 3.9.4 (tags/v3.9.4:1f2e308, Apr  6 2021, 13:40:21) [MSC v.1928 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
#### `pip` 安装
接下来使用 `python` 安装 `pip`，安装详情参考 [https://pip.pypa.io/en/stable/installing/](https://pip.pypa.io/en/stable/installing/)
```
Do I need to install pip?
pip is already installed if you are using Python 2 >=2.7.9 or Python 3 >=3.4 downloaded from python.org or if you are working in a Virtual Environment created by virtualenv or venv. Just make sure to upgrade pip.

Use the following command to check whether pip is installed:


Unix/macOS

Windows
C:\> py -m pip --version
pip X.Y.Z from ...\site-packages\pip (python X.Y)
```
找到 `C:\Python39\Scripts` 路径，设置 `pip` 环境变量
```
 py -m pip --version
 pip 20.2.3 from C:\Python\Python39\lib\site-packages\pip (python 3.9)
```
查看版本
```
pip -V
pip 20.2.3 from c:\python\python39\lib\site-packages\pip (python 3.9)
```
`pip` 使用详解
```
pip安装包 :pip install 所需安装包名字
pip查看已安装的包: pip show --files 安装包名字
pip检查哪些包需要更新:pip list --outdate
pip升级包:pip install --upgrade 安装包名字
pip卸载安装包:pip uninstall  安装包名字
```
####  `httpie` 安装
```
pip install --upgrade httpie
Collecting httpie
  Downloading httpie-2.4.0-py3-none-any.whl (74 kB)
     |████████████████████████████████| 74 kB 297 kB/s
     
     WARNING: You are using pip version 20.2.3; however, version 21.1.1 is available.
You should consider upgrading via the 'c:\python\python39\python.exe -m pip install --upgrade p
ip' command.
```
这里警告，用 `python -m pip install --upgrade httpie`，不过这样也安装成功了
使用 `httpie`
```
http http://localhost:8080/test/
HTTP/1.1 200
Content-Length: 44
Content-Type: text/plain;charset=UTF-8
Date: Sat, 01 May 2021 09:57:25 GMT

service: demo.generator.ServiceImpl@5ada344b
```
跟 `curl` 对比
```
curl  http://localhost:8080/test/
service: demo.generator.ServiceImpl@7a2e4ad
```
`post` 请求，`1.txt` 为 入参的 `json` 格式文件
```
http http://localhost:8080/ < C:1.txt
HTTP/1.1 200
Content-Length: 6
Content-Type: application/json;charset=UTF-8
Date: Sat, 01 May 2021 22:03:05 GMT

kangan
```
#### HTTPie 使用详解
#### 概述
`httpie` 是一个命令形式的`http`客户端，它提供了简单的`http`命令，返回带高亮的结果信息，可以很方便的在`http`交互场景下进行测试、调试等。
#### 语法
```
http [flags] [METHOD] URL [ITEM [ITEM]]
METHOD没有指定时，默认为 get
URL协议没有指定时，默认为 http://
```
#### `GET` 
```
http get 请求url
请求 `url`带参数 :http 请求url param==value
```
#### `POST` 请求
```
1.post请求时表单用 = ，默认为post：http 请求url param=value
2.传递json: 可以直接传一个json类型文件，用 =@ 和 :=G，http 请求url param=@C:\1.txt
3.重定向传json：http 请求url < C:\1.txt
```
#### 更多参考 [https://github.com/httpie/httpie](https://github.com/httpie/httpie)
```
Custom HTTP method, HTTP headers and JSON data:

$ http PUT pie.dev/put X-API-Token:123 name=John
Submitting forms:

$ http -f POST pie.dev/post hello=World
See the request that is being sent using one of the output options:

$ http -v pie.dev/get
Build and print a request without sending it using offline mode:

$ http --offline pie.dev/post hello=offline
Use GitHub API to post a comment on an issue with authentication:

$ http -a USERNAME POST https://api.github.com/repos/httpie/httpie/issues/83/comments body='HTTPie is awesome! :heart:'
Upload a file using redirected input:

$ http pie.dev/post < files/data.json
Download a file and save it via redirected output:

$ http pie.dev/image/png > image.png
Download a file wget style:

$ http --download pie.dev/image/png
Use named sessions to make certain aspects of the communication persistent between requests to the same host:

$ http --session=logged-in -a username:password pie.dev/get API-Key:123
$ http --session=logged-in pie.dev/headers
Set a custom Host header to work around missing DNS records:

$ http localhost:8000 Host:example.com
```