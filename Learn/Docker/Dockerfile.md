## Dockerfile

> Dockerfile 是由一行行命令语句组成，并且支持以 `#` 开头的注释行。

#### 基本结构

* Dockerfile 一般分为四个部分：

  * **基础镜像信息**
  * **维护者信息**
  * **镜像操作指令**
  * **容器启动时执行指令**

  ```dockerfile
  # 其中，一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息。
  # 后面则是镜像操作指令，例如 RUN 指令，RUN 指令将对镜像执行跟随的命令。每运行一条 RUN 指令，镜像添加新的一层，并提交。
  # 最后是 CMD 指令，来指定运行容器时的操作命令。
  FORM ubuntu
  
  MAINTAINER docker_user docker_user@email.com
  
  RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
  RUN apt-get update && apt-get install -y nginx
  RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
  
  CMD /usr/sbin/nginx
  ```

#### 指令

> 指令的一半格式为 `INSTRUCTION arguments`，指令包括 `FORM`、`MAINTAINER`、`RUN` 等。

###### FORM

* **FORM**
  * 格式为 `FORM <image>` 或 `FORM <image>:<tag>` 。
  * 第一条指令必须是 `FORM` 指令。并且，如果在同一个 Dockerfile 中创建多个镜像时，可以使用多个 `FORM` 指令（每个镜像一次）。

###### MAINTAINER

* **MAINTAINER**
  * 格式为 `MAINTAINER <name>`，指令维护者信息。

###### RUN

* **RUN**
  * 格式为 `RUN <command>` 或 `RUN ["executable", "param1", "param2"]` 。
  * 前者将在 shell 终端中运行命令，即 `/bin/sh -c`；后者则使用 `exec` 执行。指定使用其它终端可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]`。
  * 每条 `RUN` 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 `\` 换行。

###### CMD

* **CMD**
  * 支持三种格式：
    * `CMD ["executable", "param1", "param2"]` 使用 `exec` 执行，推荐使用。
    * `CMD command param1 param2` 在 `/bin/sh` 中执行，提供给需要交互的应用。
    * `CMD ["param1", "param2"]` 提供给 `ENTRYPOINT` 的默认参数。
  * 指定启动容器时执行的命令，每个 Dockerfile **只能有一条 `CMD`命令**。如果指定了多余命令，只有最后一条会被执行。
  * 如果用户启动容器时指定了运行的命令，则会覆盖掉 `CMD` 指定的命令。

###### EXPOSE

* **EXPOSE**
  * 格式为 `EXPOSE <port> [<port>...]`。
  * 告诉 Docker 服务器容器暴露的端口号，供互联网使用。在启动容器时需要通过 `-P`，Docker 主机会自动分配一个端口转发到指定的端口。

###### ENV

* **ENV**

  * 格式为 `ENV <key> <value>`。指定一个环境变量，会被后续 `RUN` 指令使用，并在容器运行时保持。

    ```dockerfile
    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.3.4
    RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && ...
    ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
    ```

###### ADD

* **ADD**
  * 格式为 `ADD <src> <dest>` 。
  * 该命令将复制指定的 `<src>` 到容器中得 `<dest>` 。其中 `<src>` 可以是：
    * Dockerfile 所在目录的一个相对路径；
    * 一个 URL；
    * 一个 tar 文件（自动解压为目录）。

###### COPY

* **COPY**
  * 支持两种格式：
    * `ENTRYPOINT ["executable", "param1", "param2"]` 。
    * `ENTRYPOINT command param1 param2`（shell 中执行）。
  * 配置容器启动后执行的命令，并且不可被 `docker run` 提供的参数覆盖。
  * 每个 Dockerfile 中只能有一个 `ENTRYPOINT`，当指定多个时，只有最后一个有效。

###### VOLUME

* **VOLUME**
  * 格式为 `VOLUME ["/data"]` 。
  * 创建一个可以从本地主机或其它容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

###### USER

* **USER**

  * 格式为 `USER daemon` 。

  * 指定容器运行时的用户名或 UID，后续的 `RUN` 也会使用指定用户。

  * 当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，    

    例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres`。

    要临时获取管理员权限可以使用 `gosu`，而不推荐 `sudo`。

###### WORKDIR

* **WORKDIR**

  * 格式为 `WORKDIR /path/to/workdir` 。

  * 为后续的 `RUN`、`CMD`、`ENTRYPOINT`、`COPY` 和 `ADD` 指令配置工作目录。

  * 可以使用多个 `WORKDIR` 命令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。

    ```dockerfile
    # 最终路径为：/a/b/c
    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd
    ```

###### ONBUILD

* **ONBULID**

  * 格式为 `ONBUILD [INSTRUCTION]` 。

  * 配置当所创建的镜像作为其它新创建镜像的基础镜像。

  * 使用 `ONBUILD` 指令的镜像，推荐在标签中注明，如：`ruby:1.9-onbuild` 。

    ```dockerfile
    # 创建镜像 image-A
    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]
    
    # 基于 image-A 创建新镜像时，新的Dockerfile中使用 FORM image-A 指定基础镜像时，会自动ONBUILD指令内容，等价于在后边添加了两条指令。
    FROM image-A
    
    # 自动运行以下：
    ADD . /app/src
    RUN /usr/local/bin/python-build --dir /app/src
    ```

#### 创建镜像

> 编写完 Dockerfile 后，可以通过 `docker build` 命令来创建镜像。

> 基本的格式为 `docker build [options] [path]` 。
>
> 该命令将读取指定路径下（包括子目录）的 Dockerfile，并将该路径下所有内容发送给 Docker 服务器，由服务端来创建镜像。
>
> 因此一般建议放置 Dockerfile 的目录为空目录。也可以通过 `.dockerignore` 文件来让 Docker 忽略路径下的目录和文件。

```shell
# 要指定镜像的标签信息，可以通过 -t 选项
$ sudo docker build -t myrepo/myapp /tmp/test1/
```



