### 搜索镜像 docker seach 
```
> docker search busybox
NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
busybox                   Busybox base image.                             2168                [OK]

```
注：BusyBox 将许多具有共性的小版本的UNIX工具结合到一个单一的可执行文件。这样的集合可以替代大部分常用工具比如的GNU fileutils ， shellutils等工具，BusyBox提供了一个比较完善的环境，可以适用于任何小的嵌入式系统

* **拉取镜像** docker pull
* **查看镜像** docker images
* **删除运行或停止的镜像** docker rmi
* **运行容器** docker run 
* **查看运行中的容器** docker ps 或者 docker container ls
* **查看所有容器状态** docker ps -a 或者 docker container ls -a
```
> docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS                    NAMES
dc8b9ba2efb6        redis               "docker-entrypoint.s…"   7 minutes ago       Created                                                tender_chatterjee
cff3358881a2        redis               "docker-entrypoint.s…"   20 minutes ago      Exited (0) 8 minutes ago                               admiring_vaughan
cc638be418cb        busybox             "sh"                     22 minutes ago      Exited (127) 20 minutes ago                            trusting_yalow
03cd134e1313        redis               "docker-entrypoint.s…"   29 minutes ago      Up 29 minutes                 0.0.0.0:6379->6379/tcp   competent_galois
950da22b6340        95543f26ca31        "/bin/sh -c '${JAVA_…"   2 days ago          Exited (143) 2 days ago                                nexus
1c648c6719db        redis               "docker-entrypoint.s…"   5 months ago        Exited (0) 5 months ago                                redis
```
* **停止容器** docker stop
* **快速停止容器** docker kill
* **启动容器** docker start
* **重启容器** docker restart
* **暂停容器，容器不占CPU**  docker pause
* **暂停恢复** docker unpause
* **删除容器** docker rm 容器短ID1，容器短ID2，可以删除多个
* **批量删除所有已经退出的容器** docker rm -v $(docker ps -ap -f status=exited)

### 运行容器参数  docker run -d -p 80:8080

* -d 后台方式启动容器
* -p 80:8080 端口映射
* --name 容器命名
* --restart 因错误停止或者正常退出将自动重启，docker stop 或者 docker kill 的退出不会自动重启
* --restart=always 无论何种原因退出立即重启
* --restart=on-failure:3 重启3次

### 进入/退出 容器

* attach 直接进入容器启动命令的终端，不会启动新的进程
* exec 在容器中打开新的终端，并且启动新的进程
* exit 退出
* run -it 容器启动后直接进入
```
> docker run -d -it -p 6379:6379  redis
03cd134e1313edad3304154282262daa6839bc7bc6f1bed51f3404b09eca3580
docker exec -it 03cd bash
root@03cd134e1313:/#
> exit
```
```
> docker run -it redis
1:C 07 Apr 2021 13:49:07.859 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 07 Apr 2021 13:49:07.859 # Redis version=6.0.8, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 07 Apr 2021 13:49:07.859 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.0.8 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'
```
### 参考
[docker 官方镜像](https://hub.docker.com/)
[网易云镜像](https://c.163yun.com/hub#/home)
