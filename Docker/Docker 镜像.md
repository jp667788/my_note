
# 获取镜像

- ***dokcer pull*** 

``` shell
$ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

- Docker 镜像仓库地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub(`docker.io`)。
- 仓库名：如之前所说，这里的仓库名是两段式名称，即 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。


# 运行镜像

- ***docker run*** 

``` shell
$ docker run -it --rm ubuntu:18.04 bash
```

- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。

- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。

- `ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。

- `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。


# 列出镜像

- ***docker image ls***

``` shell
$ docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
redis                latest              5f515359c7f8        5 days ago          183 MB
nginx                latest              05a60462f8ba        5 days ago          181 MB
mongo                3.2                 fe9198c04d62        5 days ago          342 MB
<none>               <none>              00285df0df87        5 days ago          342 MB
ubuntu               18.04               329ed837d508        3 days ago          63.3MB
ubuntu               bionic              329ed837d508        3 days ago          63.3MB
```


- 列表包含了 `仓库名`、`标签`、`镜像 ID`、`创建时间` 以及 `所占用的空间`。
- **镜像 ID** 则是镜像的唯一标识，一个镜像可以对应多个 **标签**。


# 虚悬镜像

镜像既没有仓库名，也没有标签，均为 `<none>`。：

``` shell
<none>            <none>           00285df0df87        5 days ago  342 MB
```

>这个镜像原本是有镜像名和标签的，原来为 `mongo:3.2`，随着官方镜像维护，发布了新版本后，重新 `docker pull mongo:3.2` 时，`mongo:3.2` 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 `<none>`。除了 `docker pull` 可能导致这种情况，`docker build` 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `<none>` 的镜像。这类无标签镜像也被称为 **虚悬镜像(dangling image)**

- 列出虚悬镜像
``` shell
$ docker image ls -f dangling=true
```

- 删除虚悬镜像
``` shell
$ docker image prune
```


# 中间层镜像

为了加速镜像构建、重复利用资源，Docker 会利用 **中间层镜像**。默认的 `docker image ls` 列表中只会显示顶层镜像。如果希望显示包括中间层镜像在内的所有镜像的话，需要加 `-a` 参数。

``` shell
$ docker image ls -a
```


这样会看到很多无标签的镜像，与之前的虚悬镜像不同，这些无标签的镜像很多都是中间层镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。


# 列出部分镜像

- 根据仓库名列出镜像
``` shell
$ docker image ls ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               329ed837d508        3 days ago          63.3MB
ubuntu              bionic              329ed837d508        3 days ago          63.3MB
```

- 指定仓库名和标签
``` shell
$ docker image ls ubuntu:18.04
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               329ed837d508        3 days ago          63.3MB
```

- 过滤器参数 `--filter`，或者简写 `-f`
比如，我们希望看到在 `mongo:3.2` 之后建立的镜像，可以用下面的命令：

``` shell
$ docker image ls -f since=mongo:3.2
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              5f515359c7f8        5 days ago          183 MB
nginx               latest              05a60462f8ba        5 days ago          181 MB
```

如果镜像构建时，定义了 `LABEL`，还可以通过 `LABEL` 来过滤：

``` shell
$ docker image ls -f label=com.example.version=0.1
...
```


# 以特定格式显示

- 列出镜像 id：
	- ***docker image ls -q***
``` shell
$ docker image ls -q
5f515359c7f8
05a60462f8ba
fe9198c04d62
00285df0df87
329ed837d508
329ed837d508
```

- 列出镜像结果，并且只包含镜像ID和仓库名：
	- ***$ docker image ls --format "{{.ID}}: {{.Repository}}"***
``` shell
$ docker image ls --format "{{.ID}}: {{.Repository}}"
5f515359c7f8: redis
05a60462f8ba: nginx
fe9198c04d62: mongo
00285df0df87: <none>
329ed837d508: ubuntu
329ed837d508: ubuntu
```


# 删除本地镜像

- ***docker image rm***
``` shell
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```

## 用ID、镜像名、摘要删除镜像

- 使用短 ID 删除
	- docker image rm 501
- 使用镜像名删除
	- docker image rm centos
- 使用摘要删除
	- docker image rm node@sha256:b4f0e0bdeb5780
``` shell
$ docker image ls --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

$ docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
```


## 用 docker image ls 命令来配合

比如，我们需要删除所有仓库名为 `redis` 的镜像：
``` shell
$ docker image rm $(docker image ls -q redis)
```

或者删除所有在 `mongo:3.2` 之前的镜像：
``` shell
$ docker image rm $(docker image ls -q -f before=mongo:3.2)
```


# docker commit 

当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化

``` shell
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```


> 使用 `docker commit` 命令虽然可以比较直观的帮助理解镜像分层存储的概念，但是实际环境中并不会这样使用


# 使用 Dockerfile 构建镜像

镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 **Dockerfile**。

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建


以定制 `nginx` 镜像为例，这次我们使用 Dockerfile 来定制。

在一个空白目录中，建立一个文本文件，并命名为 `Dockerfile`:
``` shell
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
```

其内容为：
``` shell
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

## FROM 命令

定制镜像是以一个镜像为基础，在其上进行定制。
`FROM` 就是指定 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

> 在 [Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=official) 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像。scratch 是一个空白镜像。

## RUN  命令

`RUN` 指令是用来执行命令行命令的，是最常用的指令之一。其格式有两种：

- ***shell*** 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。
``` shell 
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

- ***exec*** 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。如果使用多行 RUN 命令执行多个命令，实质上就创建了7层镜像


# 构建镜像

- ***docker build*** 
``` shell
docker build [选项] <上下文路径/URL/->

$ docker build -t nginx:v3 .
```


## 镜像构建上下文

构建命令中的 `.` 为上下文路径。

Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 [Docker Remote API](https://docs.docker.com/develop/sdk/)，而如 `docker` 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。

当我们进行镜像构建的时候，并非所有定制都会通过 `RUN` 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 `COPY` 指令、`ADD` 指令等。而 `docker build` 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

当构建的时候，用户会指定构建镜像上下文的路径，`docker build` 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。


``` shell
COPY ./package.json /app/
```

这并不是要复制执行 `docker build` 命令所在的目录下的 `package.json`，也不是复制 `Dockerfile` 所在目录下的 `package.json`，而是复制 **上下文（context）** 目录下的 `package.json`。

因此，`COPY` 这类指令中的源文件的路径都是 _相对路径_ 。

## 其他 *docker build* 用法

### 直接用 Git repo 进行构建

``` shell
$ docker build -t hello-world https://github.com/docker-library/hello-world.git#master:amd64/hello-world
```

### 用给定的 tar 压缩包构建

``` shell
$ docker build http://server/context.tar.gz
```

### 从标准输入中读取 Dockerfile 进行构建

``` shell
$ docker build - < Dockerfile
```

### 从标准输入中读取上下文压缩包进行构建

``` shell
$ docker build - < context.tar.gz
```
