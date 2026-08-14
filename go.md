# GoMusic项目

## [go mod使用 - Go语言中文网 - Golang中文社区](https://studygolang.com/articles/28712)

### 运行项目

1. 打开终端，`cd` 到您的项目根目录 `GoMusic/`。

2. 执行构建和启动命令：

    ```Bash
    docker-compose up --build -d
    ```

### 验证和访问

1. **查看容器状态**:

    ```Bash
    docker-compose ps
    ```

    您应该能看到 `gomusic-mysql`, `gomusic-redis`, `gomusic-backend`, `gomusic-frontend` 四个容器正在运行 (`Up` 状态)。

2. **访问应用**: 打开您的浏览器，访问 **`http://localhost:8080`**。

## Docker

wsl更新太慢了用github镜像

[windows11 安装WSL2详细过程 - 肖祥 - 博客园](https://www.cnblogs.com/xiao987334176/p/18864140)

> [!NOTE]
>
> 我想知道该项目被docker容器化后，具体会存储在电脑什么位置

好的，这是一个非常棒的问题！了解 Docker 如何管理文件，能帮助您更好地理解容器化，也方便您未来进行备份、迁移或清理。

当您使用 Docker 容器化 `GoMusic` 项目后，相关的数据会存储在两个主要的地方：**您项目文件夹中的持久化数据**和 **Docker 自己管理的内部数据**。

### 1. 您能直接看到和管理的数据 (持久化数据)

这部分数据是最重要、最需要您关注的。根据我们编写的 `docker-compose.yml` 文件，您项目中的数据库和 Redis 数据被**显式地映射**到了您电脑上的文件夹里。

在您的项目根目录 (`D:\WEB\GoMusic\`) 下，`docker-compose up` 命令会自动创建两个新的文件夹：

- `D:\WEB\GoMusic\mysql-data\`
    - **作用**：这个文件夹存放着 **MySQL 数据库的所有数据**。包括您创建的表、插入的歌曲信息等，都会以文件的形式保存在这里。
    - **特点**：这是“持久化”数据。即使您运行 `docker-compose down` 删除了 MySQL 容器，这个文件夹和里面的数据**依然会保留**。下次您再次运行 `docker-compose up`，新的 MySQL 容器会重新加载这里的数据，您的所有记录都不会丢失。
- `D:\WEB\GoMusic\redis-data\`
    - **作用**：同理，这个文件夹存放着 **Redis 的所有持久化数据**。
    - **特点**：和 MySQL 数据一样，这也是持久化的。

> **小结**：这两个文件夹是您应用的核心数据。如果您需要备份数据库，直接备份 `mysql-data` 文件夹即可。

---

### 2. Docker 内部管理的数据 (镜像、容器等)

这部分数据由 Docker Desktop 在后台统一管理，普通用户一般不需要直接操作这些文件。它们存储在一个比较深的位置。对于标准的 Windows + Docker Desktop (使用 WSL 2 后端) 安装，这些文件位于：

- **Docker 镜像 (Images)**：
    - 您 `build` 出来的 `gomusic-backend` 和 `gomusic-frontend` 镜像，以及您 `pull` 下来的 `mysql`、`redis`、`golang`、`node` 等基础镜像，都存储在这里。
- **Docker 容器 (Containers)**：
    - 当容器运行时，它本身的文件系统、日志等非持久化的数据也在这里。当您删除容器 (`docker-compose down`)，这部分数据就会被清理掉。
- **Docker 卷 (Volumes)**：
    - 除了我们上面提到的文件夹映射方式，Docker 还有一种自己管理的“命名卷”，也会存在这里。

**它们在电脑上的具体位置是在一个虚拟硬盘文件 (`.vhdx`) 中**。您可以在文件资源管理器中，通过以下路径找到它：

```txt
\\wsl.localhost\docker-desktop-data\data\docker
```

(您可以直接复制这个路径到文件资源管理器的地址栏并回车)

**说明：**

- `wsl.localhost` 是访问在 WSL 2 (Windows Subsystem for Linux 2) 中运行的 Linux 系统的网络路径。
- `docker-desktop-data` 是 Docker Desktop 为其数据创建的一个专用的 Linux 发行版。
- 里面的 `ext4.vhdx` 文件就是那个虚拟硬盘，所有 Docker 的内部数据（镜像、容器等）都封装在这个文件里。

> **小结**：您不需要手动修改这个虚拟硬盘里的任何文件。所有对镜像、容器的操作都应该通过 Docker 命令 (`docker image ls`, `docker container rm` 等) 或 Docker Desktop 的图形化界面来完成。

### 总结

| 存储内容                        | 存储位置                                                               | 由谁管理？                                  | 我需要关心吗？                                                  |
| ------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------- | --------------------------------------------------------------- |
| **MySQL 和 Redis 的数据**       | `D:\WEB\GoMusic\` 目录下的 `mysql-data` 和 `redis-data` 文件夹         | **您自己** (通过 `docker-compose.yml` 配置) | **非常需要！** 这是您应用的核心数据，需要备份。                 |
| **Docker 镜像、运行中的容器等** | `\\wsl.localhost\docker-desktop-data\data\docker` 路径下的虚拟硬盘文件 | **Docker 自己**                             | **基本不需要。** 把它看作一个黑盒，通过 Docker 命令来管理即可。 |

当然！一旦您的项目成功容器化，后续的启动和停止就变得非常简单。您只需要记住几个核心指令即可。

请务un必在您的项目根目录 (`D:\WEB\GoMusic\`) 下打开终端来执行这些命令。

---

### 场景1：日常启动项目（最常用）

如果您已经停止了容器（比如关机重启后），或者使用 `docker-compose down` 关闭了项目，想要重新启动所有服务（前端、后端、数据库、Redis），请使用这个命令：

```Bash
docker-compose up -d
```

- `up`: 启动 `docker-compose.yml` 文件中定义的所有服务。
- `-d`: `-d` 代表 `detached`，表示在后台运行容器。这样您的终端就不会被日志占用了。

这个命令会**重用**您上次构建好的镜像，所以启动速度会非常快。

---

### 场景2：修改了代码后，需要重新运行

如果您修改了 Go 后端代码或 Vue 前端代码，您需要重新构建镜像以包含您的改动。这时，请使用和第一次一样的命令：

```Bash
docker-compose up --build -d
```

- `--build`: 这个参数会强制 Docker Compose 重新构建您的 `backend` 和 `frontend` 服务的镜像，然后再启动容器。只有修改了代码的服务的镜像会被重新构建，没有修改的会直接重用缓存，所以速度也很快。

---

### 如何停止项目？

当您想停止所有 `GoMusic` 相关的容器时，使用以下命令：

```Bash
docker-compose down
```

- `down`: 这个命令会优雅地停止并**移除**所有相关的容器和网络。但请放心，您在 `mysql-data` 和 `redis-data` 文件夹里的数据**不会**被删除。

---

### 总结2

| 您的需求                            | 使用的指令                     |
| ----------------------------------- | ------------------------------ |
| **启动项目** (电脑重启后、日常使用) | `docker-compose up -d`         |
| **修改了代码后，更新项目**          | `docker-compose up --build -d` |
| **停止项目**                        | `docker-compose down`          |
