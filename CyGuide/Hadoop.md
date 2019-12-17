# Hadoop in ubuntu settings
    基本环境：
    1. ubuntu18.04
    2. jdk8 (需配置$JAVA_HOME), 如不配置$JAVA_HOME需修改hadoop启动文件
    3. 参考文档 http://dblab.xmu.edu.cn/blog/install-hadoop/
## 安装Hadoop2
Hadoop 2 可以通过 http://mirror.bit.edu.cn/apache/hadoop/common/ 或者 http://mirrors.cnnic.cn/apache/hadoop/common/ 下载，一般选择下载最新的稳定版本。
（本次实践安装的是2.7.7版本。）

解压下载下来的压缩包
> sudo tar -zxf ~/cy_space/cy_software/hadoop-2.7.7.tar.gz -C ~/cy_space/cy_software/hadoop_dir    # 解压到hadoop_dir中 <br/>
> cd ~/cy_space/cy_software/hadoop_dir <br>
> sudo chown -R ycao ./hadoop-2.7.7       # 修改文件权限

使用命令检查hadoop是否可用
> cd ~/cy_space/cy_software/hadoop_dir/hadoop-2.7.7 <br>
> ./bin/hadoop version

## Hadoop 单机模式（简单描述，具体见参考文档）
> cd ~/cy_space/cy_software/hadoop_dir/hadoop-2.7.7 <br>
> mkdir ./input <br>
> cp ./etc/hadoop/*.xml ./input   # 将配置文件作为输入文件 <br>
> ./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep ./input ./output 'dfs[a-z.]+' <br>
> cat ./output/*          # 查看运行结果

## Hadoop 伪分布式配置
修改./etc/hadoop/core-site.xml文件 与 hdfs-site.xml文件
```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/home/ycao/cy_space/cy_software/hadoop_dir/hadoop-2.7.7/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/home/ycao/cy_space/cy_software/hadoop_dir/hadoop-2.7.7/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/home/ycao/cy_space/cy_software/hadoop_dir/hadoop-2.7.7/tmp/dfs/data</value>
    </property>
</configuration>
```
    遇到问题，当name dir 与 data dir设置在/usr/目录下时，会出现启动hadoop失败的问题，暂未知详细原因，暂时猜测为权限问题。

配置完成后，需要格式化：
> ./bin/hdfs namenode -format
    
    此时出现的问题为，1. permission denied。 2. JAVA_HOME 获取不了的问题。此时需要将~/cy_space/cy_software/hadoop_dir/hadoop-2.7.7/etc/hadoop/hadoop-env.sh 中修改JAVA_HOME为指定目录。

启动hadoop:
> ./sbin/start-dfs.sh  #start-dfs.sh  是个完整的可执行文件，中间没有空格
> jps   # 查看启动状态

访问 http://localhost:50070 查看dashbord， 此时hadoop已启动。

### 运行Hadoop伪分布式实例（具体见参考文档）
1. 创建HDFS上的目录
   > ./bin/hdfs dfs -mkdir -p /user/hadoop
2. 创建input目录，将./etc/hadoop下的xml文件复制到文件系统中
   > ./bin/hdfs dfs -mkdir /user/hadoop/input
   > ./bin/hdfs dfs -put ./etc/hadoop/*.xml /user/hadoop/input
3. 复制完成后，可以通过命令查看文件列表：
   > ./bin/hdfs dfs -ls /user/hadoop/input
4. 执行grep操作
   > ./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep /user/hadoop/input /user/hadoop/output 'dfs[a-z.]+'
5. 查看运行结果：
   > ./bin/hdfs dfs -cat /user/hadoop/output/*

## 关闭Hadoop
> ./sbin/stop-dfs.sh


