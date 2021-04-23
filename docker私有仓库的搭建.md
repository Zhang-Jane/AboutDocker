## 搭建无认证私有仓库

- 第一步:在需要搭建仓库的服务器.上安装docker。

- 第二步:在服务器上,从docker hub下载registry仓库
    docker pull registry

- 第三步:在服务器上,启动仓库

    ```bash
    docker run -d -ti --restart always\ # 自动重启
    --name my-registry\ # 仓库的名称
    -p 8000:5000\ # 对外开放端口8000
    -v /my-registry/registry:/var/lib/registry\ # 挂载数据卷
    registry
    ```

    

    注意: registry内部对外开放端口是5000。默认情况下,会镜像存放于容器内的
    /var/lib/registry(官网Dockerfile中查看)目录下,这样如果容器被删除,则存放于容器中的镜像也会丢
    失。
    本地利用curl服务器IP:8000/v2/_ catalog查看当前仓库中的存放的镜像列表。( 注意打开8000端口访问)

### 私有仓库--_上传、下 载镜像
- 第一步:利用docker tag重命名需要上传的镜像
    docker tag IMAGE服务器IP:端口/IMAGE_ NAME
- 第二步:利用docker push,上传刚刚重命名的镜像
    docker push服务器IP:端口/centos
- 注意:
    必须重命名为服务器IP:端口/IMAGE_ NAME
    如果push出现了类似https的错误那么需要往配置文件/etc/docker/daemon.json里添
    加:”insecure-registries" :[“服务器IP:端口”]
    然后重启docker.

## 搭建带认证的私有仓库
在服务器上:
第一步:删除先前创建的无认证的仓库容器
docker rm -f my-registryr
第二步:创建存放认证用户名和密码的文件:
mkdir /my-registry/auth -p
第三步:创建密码验证文件。注意将将USERNAME和PASSWORD替换为设置的用户名和密码

```bash
docker run --entrypoint htpasswd registry -Bbn USERNAME PASSWORD >
/my-registry/auth/htpasswd
```



第四步:重新启动仓库镜像

```bash
docker run -d -p 8000:5000 --restart= always --name docker-registry \
-v /my-registry/registry:/var/lib/registry \
-v /my-registry/auth:/auth \
-e "REGISTRY_ _AUTH=htpasswd" \
-e "REGISTRY_ _AUTH_ HTPASSWD_ _REALM= Registry Realm" \
-e "REGISTRY_ _AUTH_ HTPASSWD_ _PATH=/auth/htpasswd" \
registry
```

### 带认证的私有仓库-上传、下 载镜像
在本地机器上:
第一步:首先登录到服务器
docker login -u username -p password IP:8000
第二步:然后执行pull或者push命令
第三步:操作完毕后,可以退出登录
docker logout IP:8000
这是如果想查看仓库中已有的镜像,那么需要进行http验证才可以。可以直接借助浏览器访问
IP:8000/v2/_ catalog就可以访问了
IP指服务器IP