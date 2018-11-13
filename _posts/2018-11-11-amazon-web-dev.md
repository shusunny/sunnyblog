---
layout: post
title: "在amazon AWS上部署程序"
author: "sun"
categories: Web-dev
tags: [Web-dev, AWS, Golang]
---

## 什么是AWS

亚马逊AWS（Amazon Web Services (AWS) ）是亚马逊提供的专业云计算服务，于2006年推出，以Web服务的形式向企业提供IT基础设施服务，通常称为云计算。其主要优势之一是能够以根据业务发展来扩展的较低可变成本来替代前期资本基础设施费用。

亚马逊网络服务所提供服务包括：亚马逊弹性云计算（Amazon EC2）、亚马逊简单储存服务（Amazon S3）、亚马逊简单数据库（Amazon SimpleDB）、亚马逊简单队列服务（Amazon Simple Queue Service）以及Amazon CloudFront等。

截至2016年，AWS拥有70多项服务，包括计算，存储，网络，数据库，分析，应用程序服务，部署，管理，移动，开发人员工具和物联网工具。相比于我们自己构建的物理服务器，用AWS构建的服务器更便于维护，更稳定，并且价格也比较合适个人用户和轻量级企业。

## EC2实例

亚马逊弹性云计算（Amazon Elastic Compute Cloud，简称Amazon EC2） ，是由亚马逊公司提供的Web服务，是一个让用户可以租用云端计算机运行所需应用的系统。EC2借由提供Web服务的方式让用户可以弹性地运行自己的Amazon机器映像档，用户将可以在这个虚拟机上运行任何自己想要的软件或应用程序。操作系统支持Windows以及各种Linux，适合不同习惯的各种用户。

用户可以随时创建、运行、终止自己的虚拟服务器，使用多少时间算多少钱，也因此这个系统是“弹性”使用的。EC2让用户可以控制运行虚拟服务器的主机地理位置，这可以改善让延迟还有备援性例如，为了让系统维护时间最短，用户可以在每个时区都运行自己的虚拟服务器。

## 如何创建EC2实例

网上有很多相关教程，这里不作累述了，只记录下简单步骤。可以参见amazon[创建 EC2 实例并安装 Web 服务器](https://docs.aws.amazon.com/zh_cn/AmazonRDS/latest/UserGuide/CHAP_Tutorials.WebServerDB.CreateWebServer.html)，或自行百度(google)

1. 创建 AWS 帐户
2. 登陆控制台(console) 
3. 创建EC2实例

  - services / EC2
  - launch instance
  - choose your instance
  - add storage / 30GB free
  - add tags / webserver
  - security / ssh / http
  - launch
  - create new key pair / download

## 部署程序

1. 找一个安全的文件夹妥善保存key文件
```
 mv [src] [dst] / sudo chmod 400 your.pem
```

2. 编译Go程序（这里我用的是linux系统）
```
GOOS=linux GOARCH=amd64 go build
```

3. 用scopy方式来上传程序 (可以在实例信息栏找到user和public-DNS信息)
```
scp -i /path/to/[your].pem ./main ec2-user@[public-DNS]:
```
 - say "yes" to The authenticity of host if necessary.

4. 用SSH方式登陆server 
```
ssh -i /path/to/[your].pem ec2-user@[public-DNS]
```

5. 运行web程序
```
sudo chmod 700 yourprogram
sudo ./yourprogram
```
- 可以在实例信息栏查看并转到 **[public-IP]** 看到我们部署的程序

6. 退出实例
  - ctrl + c
  - exit

## 持续运行程序

现在我们的程序有个问题，一旦我们退出实例或关闭终端，程序也会被一同关闭。为了解决这个问题，我们可以用以下方式

- screen
- init.d
- upstart
- system.d

这里我们用system.d的方式

首先我们需要在系统system文件夹中创建配置文件
```
cd /etc/systemd/system/
sudo nano <filename>.service
```

在nano编辑器中复制以下内容，注意需要更改入口程序(ExecStart)和工作目录(WorkingDirectory)

```
[Unit]
Description=Go Server

[Service]
ExecStart=/home/<username>/<exepath>
WorkingDirectory=/home/<username>/<exe-working-dir>
User=root
Group=root
Restart=always

[Install]
WantedBy=multi-user.target
```

现在我们需要用以下命令来运行程序

```
// Add the service to systemd.
sudo systemctl enable <filename>.service

// Activate the service.
sudo systemctl start <filename>.service

// Check if systemd started it.
sudo systemctl status <filename>.service

// Stop systemd if so desired.
sudo systemctl stop <filename>.service
```