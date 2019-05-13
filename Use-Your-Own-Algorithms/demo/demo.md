# 使用自定义算法demo
## 1.制作自己的docker镜像
　　使用自定义算法，需要自己制作docker镜像，docker制作和使用，可以参考[docker官方教程](https://docs.docker.com/develop/)，本例主要演示docker镜像如何配合ModelMaker平台进行使用，基于alpine:3.7镜像进行制作。code目录下包含了本例中的Dockerfile和脚本demo.sh。

Dockerfile的内容如下（演示如何基于alpine:3.7镜像，安装程序和拷贝自己的代码到镜像中）：  
```
FROM alpine:3.7
MAINTAINER demo <juest a demo>

RUN apk add --update \
    bash 

COPY  demo.sh /home/
```
脚本demo.sh内容如下（演示如何获取环境变量，查看数据集内容及输出模型）：  
```
#!/bin/bash
echo "===========================打印环境变量=================================="
env

echo "===================列出数据集目录下的文件情况============================"
ls -l /opt/ai/input/data/

echo "==========在训练输出目录创建一个假的模型文件model.txt===================="
touch /opt/ai/output/model.txt
ls -l /opt/ai/output/
echo "========================================================================="
```
在Dockerfile的目录下，使用以下命令可以将上面的Dockerfile编译成镜像名为demo，版本号为1.0的镜像：
```
docker build -t demo:v1.0 .
```
## 2.上传自己的docker镜像到本地仓库
步骤如下：  
1）新建本地仓库（可选，如果容器服务中已有与AI训练同个分区的仓库可用，可跳过此步）  
　　点开网宿云->容器服务->选择与AI训练相同的分区->交付中心->本地仓库，新建仓库  
　　第一次使用前，需要点击重置密码，设置仓库密码  
2）点击仓库名，进行仓库，可以查看此仓库下的镜像，点击操作指引，依据指引上传镜像：  
　　登陆网宿云docker registry （docker login命令）  
　　生成完成后，标记您的映像，以便将映像推送到此仓库(docker tag命令)  
　　运行以下命令将此镜像推送到您的本地仓库(docker push命令)  
## 3.使用自定义算法镜像进行训练  
在AI平台点击 训练任务 -> 创建训练任务，进行训练任务的创建。创建训练任务时，需要关注以下信息：  
_ _ _
任务信息：  
　　　　名称：　填写此次训练的名称，如自定算法  
　　　　算法来源选择： 自定义  
　　　　算法镜像： 选择步骤2上传的代码镜像  
　　　　代码目录：支持s3和git选择需要下载的代码，本例放空  
_ _ _
运行设置：  
　　　　环境变量：依据需求填放环境变量，本例放空  
　　　　Command：启动命令，本例使用bash脚本，故填入：/bin/bash  
　　　　args：启动参数，体例需要使用bash启动/home/demo.sh，故填入：/home/demo.sh  
_ _ _
输入：  
　　　　数据集： 数据集的s3目录  
_ _ _
输出：  
　　　　训练输出位置： 选择模型文件保存的位置  
_ _ _
运行配置：  
　　　　资源配置： 选择训练所使用的硬件资源 本例中使用最小配置 4核8G即可  
　　　　附加卷： 选择1G  
　　　　最长训练时间： 放空即可  
_ _ _
配置完成后，点击开始训练，即可开始训练。训练过程中可通过运行监控及训练日志，查看训练情况，训练完成后，任务状态会更新为训练完成，可以查看到训练输出位置中指定的s3路径下，有生成了一个model.txt文件。  
## 4.推理部署
推理部署镜像的制作和上传方法同训练一样，只是推理代码中需要监听10000端口，并且对推理请求进行处理，在此就不再举例。



