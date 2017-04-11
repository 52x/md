title: docker architecture
tags: docker
---

### docker是什麽
Docker是一个开放的平台，用来开发，传输和运行你的应用。   
Docker通过将内核的容器功能和一个工作流程结合起来，提供相应的工具来管理和部署你的应用。   
Docker的核心是将任何的应用都隔离的运行在一个容器里，相比于虚拟机，docker要轻量的多。   

### Docker能用来干什么
- 改善你的开发周期


### Docker的主要组件
- Docker: 开源的容器平台
- Docker hub: docker提供的Software-as-a-Service平台用来分享和管理docker容器

### Docker架构
Docker使用的client-server架构。 [](/images/docker_architecture.png)

- Docker daemon   
运行在主机上，用户通过docker client来与docker daemon交互。
- Docker client   
Docker client是以docker命令提供给用户的。
- Docker内部
	- Docker images   
		Docker image是一个只读的镜像，通过docker image来创建docker容器。
	- Docker registries
		用来存放images。docker hub是一个公开的docker registries。
	- Docker containers
		从image创建，包含运行应用的所有依赖。

### Docker是如何工作的
- docker image   
Docker image包含很多层，利用UFS文件系统来将这些层生成一个image，每当你改变docker image的时候，一个新的层就创建了。   
Docker包含很多基础的镜像，docker image可以在这些镜像的基础上构建，docker可以根据一系列指令来构建你的镜像，每个指令都创建一个新的层，指令包含 :
	- 运行一个命令
	- 增加文件
	- 创建一个环境变量 
	- 当容器运行时启动什么程序
将这些指令存储到Dockerfile，docker会根据里面的指令创建镜像。

- container   
docker image描述了container包含些什么，当contianer启动时运行什么程序。当docker启动容器时，会在image之上增加一个新的可读写的层。

		1. Pulls the ubuntu image: Docker checks for the presence of the ubuntu image and, if it doesn’t exist locally on the host, then Docker downloads it from Docker Hub. If the image already exists, then Docker uses it for the new container.
		2. Creates a new container: Once Docker has the image, it uses it to create a container.
		3. Allocates a filesystem and mounts a read-write layer: The container is created in the file system and a 4. read-write layer is added to the image.
		5. Allocates a network / bridge interface: Creates a network interface that allows the Docker container to talk to the local host.
		6. Sets up an IP address: Finds and attaches an available IP address from a pool.
		7. Executes a process that you specify: Runs your application, and;
		8. Captures and provides application output: Connects and logs standard input, outputs and errors for you to see how your application is running.