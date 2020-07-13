# Redis cluster by docker

[redis-cluster(包含自己创建dockerfile启动redis cluster)](https://www.jianshu.com/p/b7dea62bcd8b)

---

## Docker方式部署redis-cluster步骤
1. redis容器初始化
2. redis容器集群配置


#### redis容器初始化
容器初始化，使用docker-compose方式，先创建一个docker-compose.yml文件，内容如下：
```
version: '3'

services:
 redis1:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8001/data:/data
  environment:
   - REDIS_PORT=8001

 redis2:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8002/data:/data
  environment:
   - REDIS_PORT=8002

 redis3:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8003/data:/data
  environment:
   - REDIS_PORT=8003
 redis4:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8004/data:/data
  environment:
   - REDIS_PORT=8004

 redis5:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8005/data:/data
  environment:
   - REDIS_PORT=8005

 redis6:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8006/data:/data
  environment:
   - REDIS_PORT=8006
```

这里引用了别人的一个镜像publicisworldwide/redis-cluster，方便快捷。
这里使用host(主机)网络模式，把redis数据挂载到本机目录/data/redis/800*下。

(未验证)若不想使用host模式，也可以把network_mode去掉，但就要加ports映射。
redis-cluster的节点端口共分为2种，一种是节点提供服务的端口，如6379；一种是节点间通信的端口，固定格式为：10000+6379。
```
version: '3'

services:
 redis1:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8001/data:/data
  environment:
   - REDIS_PORT=8001
  ports:
    - '8001:8001'       #服务端口
    - '18001:18001'   #集群端口

 redis2:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8002/data:/data
  environment:
   - REDIS_PORT=8002
  ports:
    - '8002:8002'
    - '18002:18002'

 redis3:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8003/data:/data
  environment:
   - REDIS_PORT=8003
  ports:
    - '8003:8003'
    - '18003:18003'

 redis4:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8004/data:/data
  environment:
   - REDIS_PORT=8004
  ports:
    - '8004:8004'
    - '18004:18004'

 redis5:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8005/data:/data
  environment:
   - REDIS_PORT=8005
  ports:
    - '8005:8005'
    - '18005:18005'

 redis6:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8006/data:/data
  environment:
   - REDIS_PORT=8006
  ports:
    - '8006:8006'
    - '18006:18006'
```

创建文件后，直接启动服务

```
窗口模式
docker-compose up
后台进程
docker-compose up -d
```

查看启动进程
```
ycao@ubuntu_vm:~/cy_space/cy_docker/redis_cluster$ docker-compose ps
        Name                       Command               State   Ports
----------------------------------------------------------------------
rediscluster_redis1_1   /usr/local/bin/entrypoint. ...   Up
rediscluster_redis2_1   /usr/local/bin/entrypoint. ...   Up
rediscluster_redis3_1   /usr/local/bin/entrypoint. ...   Up
rediscluster_redis4_1   /usr/local/bin/entrypoint. ...   Up
rediscluster_redis5_1   /usr/local/bin/entrypoint. ...   Up
rediscluster_redis6_1   /usr/local/bin/entrypoint. ...   Up
```

状态为Up，说明服务均已启动，镜像无问题。

---


#### redis容器集群配置
上面只是启动了6个redis容器，并没有设置集群，通过下面的命令可以设置集群。
```
docker run --rm -it inem0o/redis-trib create --replicas 1 10.211.55.13:8001 10.211.55.13:8002 10.211.55.13:8003 10.211.55.13:8004 10.211.55.13:8005 10.211.55.13:8006
```

这里同样使用了另一个镜像inem0o/redis-trib，执行时会自动下载。

可以看到设置了3个Master，3个Slave.

开放本地redis接口
```
sudo iptables -I INPUT -p tcp --dport 8004 -j ACCEPT
```