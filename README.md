
# Ros-Noetic-Docker-Bridge

本项目旨在实现 Docker 与 ROS 系统在 Ubuntu 环境中的互通。核心目的是让 Docker 能够向宿主机发布消息，并使主机能够接收来自 Docker 的消息。得益于 Docker 的高度封装性，任何 ROS 项目都可以保存于 Docker 容器中，并且可以无缝迁移到任何支持 Docker 的平台上直接运行。

## 使用 FishROS 下载 Docker 和 Images

### 安装docker

本项目需要对 Docker 有一定了解。在终端中执行以下命令：

```shell
wget http://fishros.com/install -O fishros && . fishros
```

执行后会显示如下界面：

<img src="./assets/image-20240306174537980.png" alt="image-20240306174537980" style="zoom:67%;" />

接着输入 **8**，一键安装 Docker。FishROS 会根据当前系统及架构安装合适的 Docker 应用。按照提示完成安装后，用以下命令检查 Docker 是否安装成功：

```shell
docker --version
```

如果得到版本信息，表示安装成功。

![image-20240306174644591](./assets/image-20240306174644591.png)



### 拉取镜像

鉴于 Docker Hub 中 ROS 的镜像众多，且某些镜像可能存在小问题，建议继续使用 FishROS 来下载所需的镜像。
```
. fishros
```

执行后将重新显示之前的界面。此时选择 **11** 以一键安装 ROS 的 Docker 版本，然后跟随指示操作。**在此示例中，选择的是 Noetic 版本。**注意，FishROS 会自动创建一个 Docker 实例，该实例与系统共享所有用户文件，这可能不是预期的行为。因此，建议先删除该 Docker 实例。

```shell
docker ps -a # 查看所有镜像
docker rm <CONTAINER ID>  # <CONTAINER ID>换成对应的容器id
```

接下来，查看当前的镜像列表：

```shell
docker images  
```

![image-20240306175357205](./assets/image-20240306175357205.png)

此时应能看到名为 `ros` 的镜像，即刚才拉取的镜像。接着，创建一个新的实例：

```shell
docker run -it -d --network=host --name=<CONTAINER NAME> <IMAGE ID>
# 示例
# docker run -it -d --network=host --name=tmp c9bea440a091
```

然后，查看已创建的实例：

```shell
docker ps
```

![image-20240306175658740](./assets/image-20240306175658740.png)

此时，可以进入到容器中：

```shell
docker exec -it 246a4bf148ed /bin/bash
# 246a4bf148ed是容器的id，由于是随机的，可能会不一样
```

![image-20240306180005964](./assets/image-20240306180005964.png)

**注意，每次 Docker 重启后都需要重新加载 ROS 的基本路径：**

```shell
. ros_entrypoint.sh
```




## 测试ROS与Docker通信

首先，在宿主机上打开第一个终端，输入以下命令启动 ROS 核心：

```shell
roscore
```

接着，在新的终端中监听话题 `/test_topic`，输入以下命令：

```
rostopic echo /test_topic
```

然后，在另一个新的终端中进入 Docker 容器：

```shell
docker exec -it 246a4bf148ed /bin/bash
```

在 Docker 容器中，发送消息给 ROS，输入以下命令：

```shell
rostopic pub -r 1 /test_topic std_msgs/String "data: Test"
```

这样，在 Docker 内部运行的程序就可以成功接收宿主机发送的 ROS 消息了。

![image-20240306180618281](./assets/image-20240306180618281.png)



**注意：MacOS下的操作系统宿主机和docker是有物理隔离的，不能直接使用上述方法，Windows尚未测试，推荐使用Ubuntu**

