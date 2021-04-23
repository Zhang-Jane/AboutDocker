## docker是基于C/S架构的，分server和client端

### 开放远端连接（centos）

1. 修改配置文件
    执行命令： `vim /lib/systemd/system/docker.service`

   ```
   修改如下
   ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
   ```

   

   ![img](https:////upload-images.jianshu.io/upload_images/16394388-6ddbb4f196ed4204.png?imageMogr2/auto-orient/strip|imageView2/2/w/779/format/webp)

   

2. 将管理地址写入  /etc/profile
    执行命令：`echo 'export DOCKER_HOST=tcp://0.0.0.0:2375' >> /etc/profile`
    执行命令：`source /etc/profile`

3. 重启服务
    执行命令： `systemctl daemon-reload && systemctl restart docker`

4. 查看端口的是否开放

   ```bash
   #查看端口
   firewall-cmd --list-ports
   
   #添加端口
   firewall-cmd --permanent --zone=public --add-port=2375/tcp
   
   #重启防火墙
   service firewalld restart
   ```

   

****



### 另外一种方式

修改/etc/docker/daemon.json文件，如果没有在该目录创建即可，然后添加hosts

{

"hosts": [

"unix:///var/run/docker.sock",

"tcp://0.0.0.0:2375"

]

}

hosts 分tcp,uninx,fd三种模式，第一中时tcp指定网络连接方式，0.0.0.0:2375是指所有网络都可以连接，不安全，因此一般会加上stl证书形式，这里我用的局域网，所有没有加证书，第二种uninx时指本地可以自由连接docker，第三种，理解不是很清楚，不发表见解，现在重启docker服务，重启前可以执行重新加载配置文件sudo systemctl daemon-reload

重启docker：

```
systemctl daemon-reload 
systemctl restart docker.service
```

