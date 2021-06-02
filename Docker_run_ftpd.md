### 查看  docker search vsftpd 
```
NAME                                     DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
fauria/vsftpd                            vsftpd Docker image based on Centos 7. Suppo…   153                                     [OK]
panubo/vsftpd                            vsftpd - Secure, fast FTP server                36                                      [OK]
vimagick/vsftpd                                                                          13                                      [OK]
odiobill/vsftpd                          Very light vsftpd installation based on Debi…   7                                       [OK]
million12/vsftpd                         VSFTPD Server in a Docker                       7                                       [OK]
emilybache/vsftpd-server                                                                 6                                       [OK]
avenus/vsftpd-alpine                     Docker image of vsftpd server based on Alpin…   5                                       [OK]
wildscamp/vsftpd                         An FTP server designed to simplify local dev…   4                                       [OK]
loicmathieu/vsftpd                       vsftpd container                                2                                       [OK]
akue/vsftpd                              vsftpd Docker image based on Centos 7. Suppo…   1
hiproz/vsftpd                            an vsftpd that support virtual user which ha…   1
benssson/vsftpd                          copy of wildscamp/vsftpd but with pasv_addr_…   1                                       [OK]
instantlinux/vsftpd                      A clean, easy-to-use, tiny yet full-featured…   1                                       [OK]
mikenye/vsftpd-anon-uploads              A generic, ready-to-go anonymous ftp server …   1                                       [OK]
undying/vsftpd                           Vsftpd Docker Container                         0                                       [OK]
ledermann/vsftpd                         Clone of helderco/docker-vsftpd, just to pro…   0
dmanas/vsftpd-mysql                                                                      0
dolphyvn/vsftpd_priv                                                                     0
markhobson/vsftpd                                                                        0
valus/vsftpd                             vsftpd on CentOS 7 for internal usage.          0
shourai/vsftpd-alpine                    vsftpd based on alpine                          0
ernestas/vsftpd-server                   simple vsftpd server                            0                                       [OK]
openmicroscopy/vsftpd-anonymous-upload   Vsftpd Docker image for anonymous FTP upload…   0                                       [OK]
zloystrelok/vsftpd                       fixed fork vsftpd                               0
vistrcm/vsftpd                           This Docker container implements a vsftpd se…   0                                       [OK]

```
### 拉取镜像  docker pull fauria/vsftpd 
```
Using default tag: latest
latest: Pulling from fauria/vsftpd
75f829a71a1c: Downloading [=>                                                 ]   2.15MB/75.86MB
a1a6b490d7c7: Downloading [=====>                                             ]  719.8kB/6.415MB
ad2cabfec967: Downloading [======>                                            ]  986.3kB/7.923MB
c7a98e8d62f5: Waiting
10d192add873: Waiting
fc18a09c86d0: Waiting
5397e9c5e314: Waiting
e89f582c70f5: Waiting
8b8bdebbfc97: Waiting
026ae919720d: Waiting
```
### 获取本机地址  ipconfig 
```
以太网适配器 vEthernet (Default Switch):

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::f55c:61e5:557f:503d%39
   IPv4 地址 . . . . . . . . . . . . : 192.168.231.193
   子网掩码  . . . . . . . . . . . . : 255.255.255.240
   默认网关. . . . . . . . . . . . . :
```
### 运行容器
参数说明：
 C:/docker-data/:/home/vsftpd  :映射  docker  容器  ftp  文件根目录（冒号前面是宿主机的目录
 -p ：映射 docker  端口（冒号前面是宿主机的端口）
 -e FTP_USER=test -e FTP_PASS=test  ：设置默认的用户名密码
 PASV_ADDRESS ：宿主机  ip ，当需要使用被动模式时必须设置。
 PASV_MIN_PORT~ PASV_MAX_PORT ：给客服端提供下载服务随机端口号范围，默认  21100-21110 ，与前面的  docker  端口映射设置成一样。
```
docker run -d -v C:/docker-data/:/home/vsftpd -p 20:20 -p 21:21 -p 21100-21110:21100-21110 -e FTP_USER=test -e FTP_PASS=test -e PASV_ADDRESS=192. 168.231.193 -e PASV_MIN_PORT=21100 -e PASV_MAX_PORT=21110 --name vsftpd --restart=always fauria/vsftpd
```
### 进入容器  docker exec -i -t vsftpd bash 
进入  home/vsftpd  文件，查看创建的用户  test ，进入 test  目录， 创建  1.txt 2.txt ，或者在宿主机的  C:/docker-data/test  文件里面手动创建，创建之后自动更新  home/vsftpd/test 里面的文件。
```
[root@bfdc203461de /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@bfdc203461de /]# cd home/
[root@bfdc203461de home]# ls
vsftpd
[root@bfdc203461de home]# cd vsftpd/
[root@bfdc203461de vsftpd]# ls
index.html  test
[root@bfdc203461de vsftpd]# cd test/
[root@bfdc203461de test]# ls
1.txt  2.txt
[root@bfdc203461de test]# exit
exit
```
使用 IE 浏览器 输入  ftp://192.168.231.193/ 可以查看到我们新建的文件
```
FTP 根位于 192.168.231.193
若要在文件资源管理器中查看此 FTP 站点，请单击“视图”，然后单击“在文件资源管理器中打开 FTP 站点”。 
04/29/2021 01:10下午              0 1.txt
04/29/2021 01:12下午              0 2.txt

```