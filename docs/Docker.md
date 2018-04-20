# Docker容器化

文档维护 | 辰枫
---|---
更新日期 | 2017-11-27
文档版本 | v1.0


## Why Docker?

### 传统虚拟机和Docker

| 特性    | 容器        | 虚拟机    |
| ----- | --------- | ------ |
| 启动    | 秒级        | 分钟级    |
| 硬盘使用  | 一般为 MB    | 一般为 GB |
| 性能    | 接近原生      | 弱于     |
| 系统支持量 | 单机支持上千个容器 | 一般几十个  |

无论是性能、效率、还是资源占用上，Docker比传统虚拟机都有非常明显的优势。这还只是其中很小一部分。对于生产环境中的虚拟机，我们对它的要求可不仅限于性能、效率和资源占用。我们还要考虑制作、分发、部署、管理是否方便快捷、是否可以自动化。应用环境的管理成本越低，就可以投入更多的资源到更有价值是事情上，比如需求分析、产品设计、用户运营。

### 更快速的交付和部署

> 对开发和运维（devop）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。
>
> 开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。 Docker 可以快速创建容器，快速迭代应用程序，并让整个过程全程可见，使团队中的其他成员更容易理解应用程序是如何创建和工作的。 Docker 容器很轻很快！容器的启动时间是秒级的，大量地节约开发、测试、部署的时间。

### 更高效的虚拟化

> Docker 容器的运行不需要额外的 hypervisor 支持，它是内核级的虚拟化，因此可以实现更高的性能和效率。

### 更轻松的迁移和扩展

> Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。
>

### 更简单的管理

> 使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。Docker 可以使用仓库来统一管理镜像，就像Github管理代码一样管理镜像。

## Docker的基本概念

### 镜像（Image）

> Docker 镜像（Image）就是一个只读的模板。
>
> 例如：一个镜像可以包含一个完整的 ubuntu 操作系统环境，里面仅安装了 Apache 或用户需要的其它应用程序。
>
> 镜像可以用来创建 Docker 容器。
>
> Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

就好像我们运行一个程序，当程序运行起来了就变成了一个进程。但是程序本身是一个只读模板。我们可以从别人那里拷贝一份程序，或者自己写一个程序，这个程序就相当于镜像，而运行起来的进程就相当于容器。

### Docker 容器

> Docker 利用容器（Container）来运行应用。
>
> 容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。
>
> 可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

### Docker 仓库

> 仓库（Repository）是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。
>
> 当用户创建了自己的镜像之后就可以使用 `push` 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 `pull` 下来就可以了。
>

## Docker化实践

环境：CentOS 7.3 64位

项目：https://gitlab.dianchu.cc/DevOpsGroup/GoWebStartKit

### 安装Docker

1.更新与添加yum仓库
```
yum -y update
```
```
cat >/etc/yum.repos.d/docker.repo <<-EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```
2.安装docker

```
yum install -y docker-engine
yum install -y docker-selinux
```
3.启动Docker
```
systemctl start docker.service
```

### 配置镜像加速器

阿里云容器服务阿里云容器服务：https://yq.aliyun.com/articles/29941

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器：
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["<your accelerate address>"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### Dockerfile打包镜像

#### 创建Dockerfile
```
FROM hub.dianchu.cc/library/gotool:latest  # 基于gotool
WORKDIR $GOPATH/src/maester-go/  # 设置工作目录
EXPOSE 9090 # 声明开放端口 只是声明并没有改变什么
COPY ./maester-go/ ./ # 复制Dockerfile同级目录下的项目文件夹 到容器的工作目录中
RUN govendor build # build Go项目
CMD ["./GoWebStartKit"] # 容器启动执行的命令
```

gotool镜像的Dockerfile：https://gitlab.dianchu.cc/docker-library/gotool/blob/master/Dockerfile

#### 使用Dockerfile build 镜像
```
docker build -t goweb:latest .
```
docker build -t 名字:版本号 .（点代表当前目录下的Dockerfile）

运行结果：
```
docker build -t goweb:latest .
Sending build context to Docker daemon 8.655 MB
Step 1 : FROM hub.dianchu.cc/library/gotool:latest
 ---> df2f964fcd66
Step 2 : WORKDIR $GOPATH/src/maester-go/
 ---> Running in c3807e9c4fef
 ---> 8aae1e51f4d6
Removing intermediate container c3807e9c4fef
Step 3 : EXPOSE 9090
 ---> Running in cb9448b3ac21
 ---> c973dc4cf54f
Removing intermediate container cb9448b3ac21
Step 4 : COPY ./maester-go/ ./
 ---> 76ca5bf259a4
Removing intermediate container 064ab6f906a1
Step 5 : RUN govendor build
 ---> Running in 32808d0f2996
 ---> ca8c50f3c15a
Removing intermediate container 32808d0f2996
Step 6 : CMD ./GoWebStartKit
 ---> Running in a97421cb483f
 ---> 17015b6ff70e
Removing intermediate container a97421cb483f
Successfully built 17015b6ff70e

```
#### 使用Docker images 查看构建好的镜像
```
Docker images
```

执行成功后：
```
[root@localhost test]# docker images
REPOSITORY                                                   TAG                 IMAGE ID            CREATED             SIZE
goweb   ←刚刚构建的镜像                                         latest              17015b6ff70e        10 minutes ago      540 MB
hub.dianchu.cc/library/gotool                                latest              df2f964fcd66        40 hours ago        510.4 MB
hub.dianchu.cc/library/fluentd-pilot-test                    0.2                 943d603a0052        10 days ago         89.63 MB
hub.dianchu.cc/library/fluentd-pilot                         latest              943d603a0052        10 days ago         89.63 MB
registry.cn-shenzhen.aliyuncs.com/xflxfl/fluentd-pilot       0.2                 943d603a0052        10 days ago         89.63 MB
hub.dianchu.cc/library/fluentd-pilot                         <none>              b1c2a06567f6        10 days ago         89.63 MB
hub.dianchu.cc/library/pilot                                 1.5.3               eb5134b879f2        11 days ago         91.35 MB
.....
```

### 容器操作

#### 运行
----------

项目需要开发的端口是9090,因此我们的docker run 命令为
```
docker run -p 9090:9090 -d  goweb:latest
```
```
docker run -p 外部端口号:内部端口号 -d (使容器在后台运行)
```
运行成功后：
```
[root@localhost ~]# docker run -p 9090:9090 -d --name goweb goweb:latest
6664b5b4dd6846077f0dd42a152b1768c70adbce3c7d735e69489bb00c9a3841
```

#### 管理运行中的容器
```
docker ps
```
```
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
6664b5b4dd68        goweb:latest        "dockerd-entrypoint.s"   32 seconds ago      Up 31 seconds       2375/tcp, 0.0.0.0:9090->9090/tcp   goweb
```
通过CONTAINER ID 可以对容器进行删除或关闭：
```
docker rm -f 666
```
or
```
docker stop 666
```

**在运行的容器中执行命令**
```
docker exec -it 666 /bin/sh （或者是/bin/bash）
```
    -d :分离模式: 在后台运行
    -i :即使没有附加也保持STDIN 打开
    -t :分配一个伪终端

```
[root@localhost ~]# docker exec -it 666 /bin/sh
/go/src/GoWebStartKit # ps
PID   USER     TIME   COMMAND
    1 root       0:00 ./GoWebStartKit
   10 root       0:00 /bin/sh
   14 root       0:00 ps
/go/src/GoWebStartKit # exit  ←退出
[root@localhost ~]#

```
### 上传镜像
hub.dianchu.cc 是内网的镜像仓库

账号：admin

密码：Harbor12345

#### 为要上传的镜像打上标签
```
docker tag goweb:latest hub.dianchu.cc/library/gotestweb:latest
```
```
docker tag 要上传的镜像 仓库地址/项目地址/镜像名:版本号
```

#### 登录内网仓库
```
docker login hub.dianchu.cc
```
```
[root@localhost docker]# docker login hub.dianchu.cc
Username (admin): admin
Password:
Login Succeeded
```
#### 上传镜像
```
docker push hub.dianchu.cc/library/gotestweb:latest
```

### 删除镜像、删除所有已关闭的容器

```
docker rmi -f goweb:latest
```

可以删除所有已关闭的容器，并清理掉它们的 Managed Volume ：
```
docker rm -v $(docker ps -aq)
```

### 日志收集

方法一: https://gitlab.dianchu.cc/go_chaos/system_log/log_output
方法二：https://gitlab.dianchu.cc/docker-library/fluentd-pilot-json

当前项目的方案是把日志标准输出到控制台，由fluentd-pilot收集：

```
func simpleLog(start time.Time, level, ip, msg string, params interface{}) {
	data := gin.H{
		"level": level, "Date": start, "IP": ip,
		"Cost":    fmt.Sprintf("%v", time.Now().Sub(start)),
		"Params":  fmt.Sprintf("%s", params),
		"message": msg,
	}
	fmt.Fprintln(os.Stdout, string(json.Marshal(data)))
}
```
收到的日志：
```
{
  "_index": "usercenter-go-ol-2017.11.27",
  "_type": "fluentd",
  "_id": "AV_7QFtK7pOjcnAEHYST",
  "_version": 1,
  "_score": null,
  "_source": {
    "log": {
      "Date": "2017-11-27 10:13:00",
      "IP": "101.201.34.209",
      "Cost": "2.6212ms",
      "Params": "map[actionid:[2106] uid:[ul955o0k81z5] sign:[] token:[eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOiJ1bDk1NW8wazgxejUiLCJqdGkiOiJZYUhuK2YwUEYrVlBkZEZndERaajdRPT0iLCJleHAiOjE1MTE4MzUxNzcsImlhdCI6MTUxMTc0ODc3NywibmJmIjoxNTExNzQ4Nzc3fQ.kac5XukiFaO0tSezfgZg23ZRb4tr0LJtJqRoyzJeKEYy9qIPdcPcJMSstx6rLITgpiiLLxHykqmTbRH4-NFtJH00SMq44wb5vIUB--mCUeM5fe0UaZsV1YW4ZYpzYJyz3u108NACHlwKutQMteBDxqgUdyLzXJoKmARWyUrfGDY9I1G5_nLz4OivgKrb4MbmNbEze7zD_IL24U5LM2YRlwQtuwXdPF6w7haf2vsGevDIfJOUH1hDumNtWKsgRStiAeg65TfugncHN8xfFKJXOEJyzbBT0nQKqRSI1SJaWhkRNhzvFQsH-67quNculbiRydmqnOEdKifKQz_JGcAViA]]",
      "Resp": "{\"Stat\":0,\"ActionId\":\"2106\",\"Msg\":\"Token 验证通过\",\"Info\":\"Token 验证通过\",\"StatusCode\":0}"
    },
    "stream": "stdout",
    "@timestamp": "2017-11-27T10:13:00.768+0800",
    "host": "e2a835728d73",
    "@target": "usercenter-go-ol",
    "docker_container": "r-baseService-usercenter-go-3-95f1c1f8"
  },
  "fields": {
    "@timestamp": [
      1511748780768
    ]
  },
  "sort": [
    1511748780768
  ]
}
```

参考地址：

[cherlex.github.io](https://cherlex.github.io/2016/09/21/%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8Docker/)

[yeasy.gitbooks.io](https://yeasy.gitbooks.io/docker_practice/content/)