## 1.github官方查找镜像

https://github.com/docker-library/python/tree/0b1fb9529c79ea85b8c80ff3dd85a32a935b0346/3.7/alpine3.10

docker-alpine 版本是3.10

## 2.在alpine环境下构建python3.7

坑爹

我们一般可能会在容器启动后进入容器，常用的是`docker attach 镜像id`，但是启动镜像的时候如果没有带 参数 `-it`的话，attach进去后可能是日志界面，并不能执行命令。所以我们会用`docker exec -it 镜像id /bin/bash/`

平常的容器一般都可以执行/bin/bash，可是alpine没有，改成 `docker exec -it 镜像id sh` 就好了。

1.创建alpine镜像

```
docker run -di --name=alpine alpine
```

2.创建对应镜像的容器

```
docker exec -it alpine sh
```

## 3.构建基础的python包

