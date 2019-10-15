# ubuntu 18.04 LTS environment settings
## base env
1. vim
1. ssh server
1. python3
1. java8
1. docker
2. git

### **1. vim** （server版本自带）
> sudo apt install vim

---

### **2. ssh server**
##### 2.1 检查是否存在sshd服务，若存在则已安装，不存在即进行以下步骤。
> ps -e | grep ssh 

##### 2.2 安装 openssh-server 服务
> sudo apt-get install openssh-server

##### 2.3 启用RSA与公钥认证
> sudo vim /etc/ssh/sshd_config <br/>
> RSAAuthentication yes PubkeyAuthentication yes

##### 2.4 将其他机器公钥放入authorized_keys文件中

> vim ~/.ssh/authorized_keys
##### 2.5 生成本机ssh key，用于获取项目
> ssh-keygen -t rsa -C "your_email@example.com"

---

### **3. python3** (server版本自带)
> sudo apt install python3
>> 由于2020年1月1日起，2.x版本就不再维护了，故以后只使用3及以上版本

##### 3.1 安装pip工具
> sudo apt install python3-pip
---

### **4. java8**
##### 4.1 安装openJDK8
> sudo apt-get install openjdk-8-jdk

---

### **5. docker**
##### 5.1 更换国内软件源 （中科大）
> sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak <br/>
> sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list <br/>
> sudo apt update
##### 5.2 安装需要的包
> sudo apt install apt-transport-https ca-certificates software-properties-common curl
##### 5.3 添加 GPG 密钥，并添加 Docker-ce 软件源，这里是中国科技大学的 Docker-ce 
> curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add - <br/>
> sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
##### 5.4 添加成功后更新软件包缓存
> sudo apt update
##### 5.5 安装 Docker-ce
> sudo apt install docker-ce
##### 5.6 设置开机自启动并启动 Docker-ce（安装成功后默认已设置并启动，可忽略）
> sudo systemctl enable docker <br/>
> sudo systemctl start docker
##### 5.7 测试运行
> sudo docker run hello-world
##### 5.8 添加当前用户到 docker 用户组，可以不用 sudo 运行 docker（可选）
> sudo groupadd docker <br/>
> sudo usermod -aG docker $USER
---
### **6. git** (server版本自带)
##### 6.1 安装git
> sudo apt install git