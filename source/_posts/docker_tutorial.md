title: docker tutorial
tags: docker
---

### 安装docker engine
有两种方式来安装docker，一种是通过linux的包管理工具（apt或者yum），另外一种是通过docker提供的shell脚本来安装，这里通过第二种方式来安装。
   
1. 确保系统安装了curl，可以通过下面的命令确认
   
		$ which curl   
如果没安装的话，可以通过下面的命令来安装

		$ sudo apt-get update
		$ sudo apt-get install curl
2. 获取最新的docker包

		$ curl -fsSL https://get.docker.com/ | sh
3. 确认docker安装成功

		ubuntu@ubuntu:~$ docker run hello-world
		Hello from Docker.
		This message shows that your installation appears to be working correctly.
		
		To generate this message, Docker took the following steps:
		 1. The Docker client contacted the Docker daemon.
		 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
		    (Assuming it was not already locally available.)
		 3. The Docker daemon created a new container from that image which runs the
		    executable that produces the output you are currently reading.
		 4. The Docker daemon streamed that output to the Docker client, which sent it
		    to your terminal.
		
		To try something more ambitious, you can run an Ubuntu container with:
		 $ docker run -it ubuntu bash
		
		For more examples and ideas, visit:
		 http://docs.docker.com/userguide/
### images & containers
Docker引擎是docker的核心，`docker run hello-world`分为三个部分![](/images/container_explainer.png)

container是一个基础的linux，image是需要加载进container运行的软件。
### 运行whalesay image
每个人都可以创建自己的docker image，你可以通过浏览docker官方的docker hub来寻找这些image。

1. 查找whalesay image   
docker hub[(https://hub.docker.com)](https://hub.docker.com)包含了个人或者大公司创建的image。
2. 运行whalesay image

		docker run docker/whalesay cowsay boo
### 创建自己的image
1. 编写Dockerfile   
Dockerfile描述了如何构建一个image， Dockerfile描述了如何构建一个image。

	- 建立一个目录，存放构建image需要的所有内容

			mkdir mydockerbuild


	- 建立Dockerfile文件   
编译Dockerfile，添加下面内容

			FROM docker/whalesay:latest
			RUN apt-get -y update && apt-get install -y fortunes
			CMD /usr/games/fortune -a | cowsay
2. 从Dockerfile构建image

		docker build -t docker-whale .
### 给image打tag，push到dockerhub，从dockerhub上pull
1. 获取image id

		docker build -t docker-whale .
2. 给image打tag
![](/images/image_tagger.png)
3. `docker login` 登录后通过`docker push`上传到docker hub




