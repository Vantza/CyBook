# Redis cluster 4.0/ 5.0

[redis-cluster 4.0](https://www.cnblogs.com/Lenbrother/p/12222715.html)
[redis-cluster 5.0](https://blog.csdn.net/adolph586/article/details/85340764#01__CentOS_75_2)



#### 拉取镜像
```
docker pull redis:4.0
```

#### 创建网卡
```
docker network create redis-net
```

### 创建redis容器
#### 创建redis配置文件
创建模板配置文件, 内容如下:
```
port ${PORT}
protected-mode no
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 10.211.55.13 # 本机ip
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
appendonly yes
```
#### 在目录下生成conf与data目标, 并生成配置信息
```
$ for port in `seq 7001 7006`; do \
  mkdir -p ./${port}/conf \
  && PORT=${port} envsubst < ./redis-cluster.tmpl > ./${port}/conf/redis.conf \
  && mkdir -p ./${port}/data; \
done
```
共生成6个文件夹, 7001~7006, 每个文件夹下包含data与conf文件夹, 同时conf里面有redis.conf配置文件

#### 创建6个redis容器
路径需要根据实际情况修改, 以下是我虚拟机中的路径
```
for port in `seq 7001 7006`; do \
  docker run -d -ti \
  -v /home/ycao/cy_space/cy_docker/redis_cluster/standalone/${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  -v /home/ycao/cy_space/cy_docker/redis_cluster/standalone/${port}/data:/data \
  --restart always --name redis-${port} --net host \
  --sysctl net.core.somaxconn=1024 redis:4.0 redis-server /usr/local/etc/redis/redis.conf; \
done
```
这里使用 host 网络方式启动容器。
至此，通过命令docker ps可查看刚刚生成的6个容器信息
docker ps 查看容器

#### 至此为止, redis 4.0 与 5.0 并无太大不同


### 集群5.0
#### 任意进入一个已运行的容器
```
$ docker exec -it redis-7001 bash
```
#### 在容器中运行执行集群命令
```
/usr/local/bin/redis-cli --cluster create \
192.168.1.157:7001 \
192.168.1.157:7002 \
192.168.1.157:7003 \
192.168.1.157:7004 \
192.168.1.157:7005 \
192.168.1.157:7006 \
--cluster-replicas 1
```
输入命令后，中间需要输入 yes 确认。


### 集群4.0
#### 打开redis-cli
```
docker exec -it redis-7001 redis-cli -p 7001
```
#### 在redis-cli中将各个容器关联
```
cluster meet 127.0.0.1 7001
cluster meet 127.0.0.1 7002
cluster meet 127.0.0.1 7003
cluster meet 127.0.0.1 7004
cluster meet 127.0.0.1 7005
cluster meet 127.0.0.1 7006
```
查看关联信息:
```
cluster nodes
```

#### 主从配置
使用7001-7003为master 7004-7006为slave
```
docker exec -it redis-7004 redis-cli -p 7004 cluster replicate {7001的容器id(cluster nodes 中的id)}
docker exec -it redis-7004 redis-cli -p 7005 cluster replicate {7002的容器id(cluster nodes 中的id)}
docker exec -it redis-7004 redis-cli -p 7006 cluster replicate {7003的容器id(cluster nodes 中的id)}
```
可执行cluster nodes 查看分配情况

#### 主节点分配redis槽点
```
docker exec -it redis-7001 redis-cli -p 7001 cluster addslots {0..5461}
docker exec -it redis-7002 redis-cli -p 7002 cluster addslots {5462..10922}
docker exec -it redis-7003 redis-cli -p 7003 cluster addslots {10923..16383}
```

#### 注意
注意1：不管你有几个主节点，一定要完全分配16383个槽点，不能多也不能少。加节点的话，就必须重复切分部分槽点到新节点上。
注意2：一定要在外面执行这个cluster addslots {a..b}命令，在客户端执行的话只能分配单独的节点，不能分配连续的节点。会报错： ERR Invalid or out of range slot