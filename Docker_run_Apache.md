### 下载镜像
```
> docker pull httpd
Using default tag: latest
latest: Pulling from library/httpd
75646c2fb410: Pull complete
a51c6e95ef2e: Pull complete
97251e2deed4: Pull complete
83942ffdf87a: Pull complete
fcf0f47f7ede: Pull complete
Digest: sha256:31ee85db3ebec898ae4e3e19ceb5c19ce622ea395d7e4844a13a8b1b141b62be
Status: Downloaded newer image for httpd:latest
docker.io/library/httpd:latest
```
### 查看镜像
```
> docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
httpd                              latest              d5995e280a0e        8 days ago          138MB
busybox                            latest              a9d583973f65        4 weeks ago         1.23MB
hello-world                        latest              d1165f221234        4 weeks ago         13.3kB
redis                              latest              bd571e6529f3        5 months ago        104MB
hub.c.163.com/lightingfire/nexus   2.13.0-01           95543f26ca31        4 years ago         455MB
```
### 运行容器
将C盘的 docker-data 文件 映射容器的  /usr/local/apache2/htdocs/  文件中
```
> docker run -it -d -p 80:80 -v C:/docker-data/:/usr/local/apache2/htdocs/ httpd
796299a44475b457c46e3a26b00aec389d2cdd669e0a1d525db79b70bd227903
```
在本地 C:\docker-data  新建  index.html ，打开 index.html 文件 编写  hello world 
查看容器 /usr/local/apache2/htdocs/  文件，先进入容器， ls  查看文件，可以看到 index.html  
```
> docker exec -it 7962 bash
root@796299a44475:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs  modules
root@796299a44475:/usr/local/apache2# cd htdocs/
root@796299a44475:/usr/local/apache2/htdocs# ls
index.html
```
浏览器输入 http://localhost:80 查看结果