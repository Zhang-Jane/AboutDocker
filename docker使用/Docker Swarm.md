## 1.Docker Swarm是什么

来源：https://www.kingname.info/2018/10/13/use-docker-swarm/

Docker Swarm是Docker自带的一个集群管理模块。他能够实现Docker集群的创建和管理。



## 2.Docker Swarm环境搭建

#### 初始化条件

- Master：45.77.138.242
- Slave-1：199.247.30.74
- Slave-2：95.179.143.21

#### 在Master上安装Docker

通过依次执行下面的命令，在Master服务器上安装Docker

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
apt-get update
apt-get install -y docker-ce
```

#### 创建Manager节点

一个Docker Swarm集群需要Manager节点。现在初始化Master服务器，作为集群的Manager节点。运行下面一条命令。

```bash
docker swarm init
```

运行完成以后，可以看到的返回结果下图所示。

![img](https://kingname-1257411235.cos.ap-chengdu.myqcloud.com/docker_swarm_2.png)

这个返回结果中，给出了一条命令：

```bash
docker swarm join --token SWMTKN-1-0hqsajb64iynkg8ocp8uruktii5esuo4qiaxmqw2pddnkls9av-dfj7nf1x3vr5qcj4cqiusu4pv 45.77.138.242:2377
```

这条命令需要在每一个从节点（Slave）中执行。现在先把这个命令记录下来。

初始化完成以后，得到一个只有1台服务器的Docker 集群。执行如下命令：

```
docker node ls
```

#### 创建子节点初始化脚本

对于Slave服务器来说，只需要做三件事情：

1. 安装Docker
2. 加入集群
3. 信任源

从此以后，剩下的事情全部交给Docker Swarm自己管理，你再也不用SSH登录这个服务器了。

为了简化操作，可以写一个shell脚本来批量运行。在Slave-1和Slave-2服务器下创建一个`init.sh`文件，其内容如下。

```
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
apt-get update
apt-get install -y docker-ce
echo '{ "insecure-registries":["45.77.138.242:8003"] }' >> /etc/docker/daemon.json
systemctl restart docker 
docker swarm join --token SWMTKN-1-0hqsajb64iynkg8ocp8uruktii5esuo4qiaxmqw2pddnkls9av-dfj7nf1x3vr5qcj4cqiusu4pv 45.77.138.242:2377
```

把这个文件设置为可自行文件，并运行

```bash
chmod +x init.sh
./init.sh
```

如下图所示。

![img](https://kingname-1257411235.cos.ap-chengdu.myqcloud.com/docker_swarm_7.png)

等待脚本运行完成以后，你就可以从Slave-1和Slave-2的SSH上面登出了。以后也不需要再进来了。

回到Master服务器，执行下面的命令，来确认现在集群已经有3个节点了：

```bash
docker node ls
```

看到现在集群中已经有3个节点了。如下图所示。

![img](https://kingname-1257411235.cos.ap-chengdu.myqcloud.com/docker_swarm_9.png)