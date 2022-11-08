# AWS

SAP(Solutions Architect-Professional) 认证 --- 题库 --- 11/15 后更新题库

重点

1. 网络 network - VPC
2. IAM 身份验证管理

容器 - docker - Kubernetes- eks

​									k8s

考试练习：aws_tiku

## IAM

### 组件

+ 用户

    1. Root User

    2. IAM User

+ 组

+ 角色

+ 权限

### IAM Policy

+ [Resource](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/s3-arn-format.html) – 存储桶、对象、访问点和
+ [Action](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/using-with-s3-actions.html) – 对于每个资源，Amazon S3 支持一组操作。Allow or Deny
+ [Effect](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_effect.html) – 当用户请求特定操作（可以是 *allow* 或 *deny*）时的效果。
+ [Principal](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/s3-bucket-user-policy-specifying-principal-intro.html) – 允许访问语句中的操作和资源的账户或用户。
+ [Condition](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/amazon-s3-policy-keys.html) – 策略生效时的条件。

**最小权限原则**

### IAM Password Policy

![image-20221107202014929](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221107202014929.png)

### Multi Factor Authentication - MFA

xusjf user ---> password + MFA Token ---> successful login

#### MFA Devices

1. Virtual MFA device
    + Google Authenticator（phone only）
    + Authy（multi-device）
2. U2F Security key（physical device）
    + Yubico（multiple root and IAM Users）

## AWS Access

1. Management Console
2. Command Line Interface - CLI
3. Software Developer Kit - SDK

### Access key

> AKIA2FYMBB3PJDPRIG2D
>
> +VQImUHCsFUCFflOz5zTidJYQbLEy9YvoCbdVKue

```bash
aws configure
```

```bash
aws iam list-users
```

### IAM Roles for Services

1. EC2 Instance Roles
2. Lambda Function Roles
3. Roles and CloudFormation

IAM Best Practies

1. 

## 启动一个 EC2 实例

### 创建 SSH Key Pair

### 创建 EC2 安全组

+ key-pair

+ 安全组

![4.png](https://s2.loli.net/2022/09/29/wr4zcxtVqWBE81h.png)

```bash
cd d:/sql
ssh -i "xushengjin.pem" ec2-user@ec2-3-83-213-41.compute-1.amazonaws.com
ssh -i "xushengjin.pem" ec2-user@3.83.213.41
## 利用弹性 ip 更换为以下地址
ssh -i "xushengjin.pem" ec2-user@54.209.165.254

```

切换到 root 用户

```bash
sudo -s
sudo su
```

## 备份和恢复

EBS 是 EC2 的块存储设备、快照 S3

### 创建 EBS 卷、快照

```bash
df -h
fdisk -l
```

## Linux User Data 的使用

在 Launch 时才会生效

![14.png](https://s2.loli.net/2022/09/29/c4TywenOIf6gABJ.png)

预置 LAMP、wordpress

```bash
#!/bin/bash
# 安装 WordPress 依赖
yum install -y httpd
yum install -y mysql
amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2

# 修改 Apache 客户端权限，以便于 WordPress 创建文件
chown -R apache /var/www
chgrp -R apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec sudo chmod 2775 {} \;
find /var/www -type f -exec sudo chmod 0644 {} \;

# 下载 WordPress 并安装
wget https://cn.wordpress.org/latest-zh_CN.tar.gz
tar -xzf latest-zh_CN.tar.gz
cp -r wordpress/* /var/www/html/

# 修改端口从 80->8443 (中国区特殊处理)
sed -i 's/Listen 80/Listen 8443/g'  /etc/httpd/conf/httpd.conf

# 重启服务并设定开机自启动
service httpd restart
systemctl enable httpd

```

预置 nginx

```bash
#!/bin/bash
sudo su
yum update -y
amazon-linux-extras install -y nginx1
systemctl start nginx
systemctl enable nginx
```

```bash
lsof -i tcp:80		# 查看占用 80 端口的进程
systemctl start nginx
systemctl enable nginx
lsof -i tcp:80
```

返回结果

```bash
[root@ip-172-31-6-60 ec2-user]# lsof -i tcp:80
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   6455  root    6u  IPv4  27714      0t0  TCP *:http (LISTEN)
nginx   6455  root    7u  IPv6  27715      0t0  TCP *:http (LISTEN)
nginx   6456 nginx    6u  IPv4  27714      0t0  TCP *:http (LISTEN)
nginx   6456 nginx    7u  IPv6  27715      0t0  TCP *:http (LISTEN)

```

## 对象存储服务 S3

Simple Storage Service

```
s3://test-xushengjin-bucket/95981713_p0.jpg
https://test-xushengjin-bucket.s3.amazonaws.com/95981713_p0.jpg
```

## S3 存储性能调优

### S3 存储权限判断流程

1. Bucket ACL
2. Object ACL
3. Bucket Policy

#### 策略最小权限原则

1. 如果某条规则显式拒绝，最终解决就是拒绝
2. 如果所有规则测试完了，有规则显示允许，则最终结果就是允许
3. 没有规则显示允许或拒绝，那么最终结果就是拒绝

+ IAM 权限
+ Bucket 权限 --- ACL --- Policy
+ Object 权限  --- ACL --- Policy
+ **存储桶权限控制** --- 觉着重要 



![5.png](https://s2.loli.net/2022/09/29/dDPIXVSzHNg1qT3.png)

### S3 防盗链

#### 识别来源

1. HTTP Referer 识别来访路径

2. S3 Bucket Policy 判断 HTTP Referer

> 以下示例
>
> 只授权 https://xushengjin.cn/* 网站访问 S3 桶 my-xushengjin-bucket-2 中的内容
>
> "Action": "s3:GetObject" 对存储桶的访问对象进行 Allow
>
> "Resource": "arn:aws:s3:::my-xushengjin-bucket-2/*"  桶资源
>
> https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/access-policy-language-overview.html

所有 S3 存储桶和对象：

```
arn:aws:s3:::*
```




```json
{
	"Version": "2022-9-27",
	"Id": "PreventHotLinking",
	"Statement": [
		{
			"Sid": "Http Referer Policy example",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::my-xushengjin-bucket-2/*",
			"Condition": {
				"StringLike": {
					"aws:Referer": "https://xushengjin.cn/*"    
				}
			}
		}
	]
}

```

+ S3 桶公有性访问设置如下：

![8.png](https://s2.loli.net/2022/09/29/c1OiLxDAYdKns4a.png)

+ 未经授权访问如下:

![6.png](https://s2.loli.net/2022/09/29/rvxjKJR7X9PS1bg.png)

![9.png](https://s2.loli.net/2022/09/29/QiW5PxrZySbFuV3.png)

+ xushengjin.cn/* 授权访问如下：

![7.png](https://s2.loli.net/2022/09/29/KG4P9Moip8DIWgw.png)

## S3 创建简单的静态网站

> http://xsj-static-site.s3-website-us-east-1.amazonaws.com/ReportStudent/

> 问题：重定向没有设置好

## 利用 CLI 访问 S3

1. 安装 AWS CLI-2

    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```

2. 查看 aws cli 版本

    ```bash
    [ec2-user@ip-172-31-88-45 ~]$ sudo ./aws/install
    You can now run: /usr/local/bin/aws --version
    [ec2-user@ip-172-31-88-45 ~]$ aws --version
    aws-cli/2.7.35 Python/3.9.11 Linux/5.10.135-122.509.amzn2.x86_64 exe/x86_64.amzn.2 prompt/off
    ```

3. 导入密钥

    > AKIA2FYMBB3PJDPRIG2D
    >
    > +VQImUHCsFUCFflOz5zTidJYQbLEy9YvoCbdVKue

    ```
    [ec2-user@ip-172-31-88-45 ~]$ aws configure
    AWS Access Key ID [****************no]: AKIA2FYMBB3PJDPRIG2D
    AWS Secret Access Key [None]: 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
    Default region name [None]: us-west-2
    Default output format [None]: json
    
    ```

4. 查看身份

    ```bash
    aws sts get-caller-identity
    ```

5. 一些命令

    | 命令    | 说明                         |
    | ------- | ---------------------------- |
    | ls      | aws s3 ls 列出当前所有 s3 桶 |
    | cp      |                              |
    | rm      |                              |
    | mb      |                              |
    | mv      |                              |
    | sync    |                              |
    | rb      |                              |
    | website |                              |

    + 创建 S3 桶

        ```bash
        aws s3 mb s3://s3-china-demo
        ```

    + 查看 S3 桶

        ```
        [ec2-user@ip-172-31-88-45 ~]$ aws s3 ls
        2022-09-27 07:29:56 my-xushengjin-bucket-2
        2022-09-27 08:36:12 s3-china-demo
        2022-09-26 08:10:40 test-xushengjin-bucket
        2022-08-07 18:55:40 vap-global-cur-699568033502
        2022-09-27 07:45:58 xsj-static-site
        2022-09-19 06:17:04 zhnc-ekk-full-log-test
        ```

    + 查看某个 S3 桶的文件内容

        ```
        [ec2-user@ip-172-31-88-45 ~]$ aws s3 ls s3://s3-china-demo
        2022-09-27 08:46:31         12 aaa1.txt
        2022-09-27 08:46:31          0 aaa10.txt
        2022-09-27 08:46:31          0 aaa2.txt
        2022-09-27 08:46:31          0 aaa3.txt
        2022-09-27 08:46:31          0 aaa4.txt
        2022-09-27 08:46:31          0 aaa5.txt
        2022-09-27 08:46:31          0 aaa6.txt
        2022-09-27 08:46:31          0 aaa7.txt
        2022-09-27 08:46:31          0 aaa8.txt
        2022-09-27 08:46:31          0 aaa9.txt
        ```

    + 复制

        ```bash
        aws s3 cp ./aaa1.txt s3://s3-china-demo
        ```

    + 多个文件复制

        ```bash
        aws s3 sync ./temp s3://s3-china-demo
        ```

    
> 问题：S3.test-xushengjin-bucket 为空，使用 rb 删除不了 S3 桶
    
![10.png](https://s2.loli.net/2022/09/29/gI5kYAQ6KwRiSZp.png)
    
```
    remove_bucket failed: s3://test-xushengjin-bucket An error occurred (BucketNotEmpty) when calling the DeleteBucket operation: The bucket you tried to delete is not empty. You must delete all versions in the bucket.
    你尝试删除的 bucket 不为空，您必须删除存储桶中的所有版本。
    ```
    
解决方法：
    
![11.png](https://s2.loli.net/2022/09/29/P5E96p4BnGyFKC8.png)
    
    ![12.png](https://s2.loli.net/2022/09/29/bF8QDlas7kiPEjc.png)
    
    ![13.png](https://s2.loli.net/2022/09/29/ng48ytDRmsKjCzO.png)



> rb 命令才得以正常删除

```bash
[ec2-user@ip-172-31-88-45 ~]$ aws s3 rb s3://test-xushengjin-bucket
remove_bucket: test-xushengjin-bucket
```

## 查看 AWS 费用账单、设置告警 --- 觉着现阶段不重要

## CloudWatch

## 负载均衡器 ELB

ELB：Elastic Load Balancer

Application Load Balancer 根据请求属性跨多个目标（例如 Amazon EC2 实例、微服务和容器）分配传入的 HTTP 和 HTTPS 流量。

![15.png](https://s2.loli.net/2022/09/29/YAzJMDmdjaqP76N.png)

> 意思就是 ELB、EC2-1、EC2-2 都必须在一个 VPC 下

```json
VPC: {
	EC2-1,
    EC-2,
    ELB
}
```

![16.png](https://s2.loli.net/2022/09/29/3bf9gGX1eT8L4ov.png)

![17.png](https://s2.loli.net/2022/09/29/Ik3rNeG9q1snEAO.png)

## RDS.MySQL

### 多可用区部署

+ 同步副本

> 选择“在不同区域中创建副本”，以便 Amazon RDS 在其他可用区中维护不同于数据库实例的同步备用副本。当主服务器出现计划或计划外停机时，Amazon RDS 将自动故障转移到备用服务器。
>
> 在不同的可用区 (AZ) 中创建一个备用实例以提供数据冗余，消除 I/O 冻结，并在系统备份期间将延迟峰值降至最小。

#### 账号密码

```
test.c6sngntbni6t.us-east-1.rds.amazonaws.com	3306	admin	xsj1234..      # 费用太贵 - 2022/10/12 已被销毁
```

**修改为多可用区 **

![18.png](https://s2.loli.net/2022/09/29/KL4YpadcNZvijfy.png)

> 将 xushengjin.cn 的数据库切成 aws.MariaDB 测试下查询速度

+ **将 aliyu.mysql.db.table ---> aws.mysql.db.table** 中

![19.png](https://s2.loli.net/2022/09/29/p8byc1KQxz5VFfu.png)

![20.png](https://s2.loli.net/2022/09/29/bChI83rpwVKSdRH.png)

+ 修改 .env 中数据库配置

    ```
    #DB_CONNECTION=mysql
    #DB_HOST=127.0.0.1
    #DB_PORT=3306
    #DB_DATABASE=larabbs
    #DB_USERNAME=admin_xsj
    #DB_PASSWORD=3.1415926
    
    DB_CONNECTION=mysql
    DB_HOST=test.c6sngntbni6t.us-east-1.rds.amazonaws.com
    DB_PORT=3306
    DB_DATABASE=test_pro
    DB_USERNAME=root
    DB_PASSWORD=xsj1234..
    ```

+ 测试性能

    **加载主页**

![21.png](https://s2.loli.net/2022/09/29/UlgHOu1kBCX3MJy.png)

+ 测试发帖

![aws.rds.mariadb.gif](https://s2.loli.net/2022/09/29/DBbYVsSUzRFAdNt.gif)

![22.png](https://s2.loli.net/2022/09/29/OYJTXzxGPcDMSoe.png)

## RDS 横向扩展

> 从 db.t3.micro 2 vCPUs 1 GiB RAM 网络：2,085Mbps ---> db.t3.xlarge 4 vCPUs 16 GiB RAM 网络：2,780Mbps

![24.png](https://s2.loli.net/2022/09/29/wHX8yg5sBIvuZSW.png)

##  VPC --- 重点

```
WordPress
2 个 Public Subnet
2 个 Private Subnet
```

![image-20221104112900084](C:\Users\XuShengjin\AppData\Roaming\Typora\typora-user-images\image-20221104112900084.png)

 1.	创建 VPC: vpc-034308cfebe5710f0  
2.	启用 DNS 主机名
3.	启用 DNS 解析
4.	正在验证 VPC 创建: vpc-034308cfebe5710f0  
5.	创建 S3 端点: vpce-03d81a5842ab8ae79  
6.	创建子网: subnet-09de66310d5dc6fc4  
7.	创建子网: subnet-07b0a8874164c2a6e  
8.	创建子网: subnet-00ebe6e393180c89d  
9.	创建子网: subnet-02055836e682fad35  
10.	创建互联网网关: igw-0bc21a60e10e7de42  
11.	将互联网网关附上该 VPC
12.	创建路由表: rtb-0ab6076b1b6d02e39  
13.	创建路由
14.	关联路由表
15.	关联路由表
16.	创建路由表: rtb-0f9021db22a99a733  
17.	关联路由表
18.	创建路由表: rtb-0c1078d2dd2e25c5d  
19.	关联路由表
20.	验证路由表创建
21.	将 S3 终端节点与私有子网路由表关联: vpce-03d81a5842ab8ae79

------

Virtual Private Cloud 虚拟私有云

公有子网 CIDR: 10.0.0.0/24 251 ip 可用

私有子网 CIDR: 10.0.1.0/24   251 ip 可用

### 路由表

#### 为什么不建议使用主路由表？

创建子网时，每个子网默认都与主路由表关联

只有在创建完成之后，才能修改分配给子网的路由表

建议在主路由表中只留下本地路由，杜绝子网被赋予不该有的路由的可能性。

在 AWS 中，典型的一个应用场景是

负载均衡器所在的 Subnet，关联到 `Public Route Table`，其默认路由指向 `IGW`，因为负载均衡器通常都`需要固定的公网IP地址` ;

而对于后端服务器，可以关联到 `Private Route Table`，由于不需要对外暴露端口，`Private Route Table` 可以将默认路由指向一个 NAT 网关，通过 NAT 网关访问互联网。

公有子网

| **目标**    | **目标**              | 说明                                              |
| :---------- | :-------------------- | ------------------------------------------------- |
| 10.0.0.0/16 | local                 | 自动创建的，VPC 内部的所有 EC2 实例之间的互访     |
| 0.0.0.0/0   | igw-021e29352101b530f | 也是自动建立的，默认路由，目标为 igw 进行公网访问 |

私有子网

| **目标**    | **目标** | 说明                                          |
| :---------- | :------- | --------------------------------------------- |
| 10.0.0.0/16 | local    | 自动创建的，VPC 内部的所有 EC2 实例之间的互访 |

### 子网关联

> 自动设置好了

公有路由表中关联公有子网

私有路由表中关联私有子网

#### 主路由表

将私有子网所在的路由表设置为 `主路由表`

> 主路由表控制的是未与任何其他路由表显式关联的所有子网的路由。是否确定要将此路由表设置为主路由表？

### 互联网网关

> 出口，可用来访问 internet，自动设置好了

创建一个 `IGW`，然后附加到 VPC

一个 VPC 只能与一个 IGW 互联

#### 与 VPC 分离

```
是否确定要将互联网网关 igw-021e29352101b530f (Demo-igw) 与 VPC vpc-0f4c8be6608511d77 分离？
如果将互联网网关分离，VPC 中的资源则无法与互联网通信。
```

#### 公网路由表配置默认路由指向 igw

| **目标**  | **目标**              | 说明                                              |
| :-------- | :-------------------- | ------------------------------------------------- |
| 0.0.0.0/0 | igw-021e29352101b530f | 也是自动建立的，默认路由，目标为 igw 进行公网访问 |

![image-20221025150923665.png](https://s2.loli.net/2022/11/01/yiMZ1exBuD4hvnp.png)

### 基础架构图

![image-20221025151527973.png](https://s2.loli.net/2022/11/01/vNFIPfUKTOZh5aM.png)

## 创建 VPC 对等连接

> 每个 VPC 之间都是互相隔离的，有时候需要让多个 VPC 之间通信
>
> 比如两个 VPC 之间，每个 VPC 里面各有 1 个私有子网，怎么实现相互通信？

+ 流量一直处于 AWS 内部网络，不会经过 Internet

+ VPC 不能有重叠的 CIDR 块

+ VPC 对等连接不支持传递（两个 VPC 之间一对一关系）

+ 可以在自己账户下的 VPC 于其它 AWS 账户的 VPC 之间创建对等连接

+ 需要 `手动更新`每个 VPC 子网的路由表

    + 路由表中要有指向其它 VPC 的路由条目

+ 可以引用另一端的对等 VPC 中的安全组，作为安全组规则中的入向或出向规则的源或目标（同一 AWS 区域）

+ 最长掩码匹配

    ![image-20221107164023123](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221107164023123.png)

![image-20221107164139344](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221107164139344.png)

配置步骤

1. 发起方创建 VPC 对等连接请求
2. 接受方接受 VPC 对等连接请求
3. 双方配置路由表，添加指向对方 VPC 的路由
4. 更改 VPC DNS 解析配置

> 1. 创建 demo-a、demo-b VPC

| CIDR 块       | VPC 名称 | 公有子网      |
| ------------- | -------- | ------------- |
| 172.29.0.0/16 | demo-a   | 172.29.0.0/24 |
| 172.30.0.0/16 | demo-b   | 172.30.0.0/24 |

> 2. 创建对等连接

![image-20221025154913835.png](https://s2.loli.net/2022/11/01/M6FNxyQhzZjDTIC.png)

> 3. 接收方接收连接

![image-20221025155042421.png](https://s2.loli.net/2022/11/01/UnFeVKJMbdLBpPN.png)

![image-20221025155143638.png](https://s2.loli.net/2022/11/01/qr6vn5CD3TQVliF.png)

![image-20221025155136595.png](https://s2.loli.net/2022/11/01/BjR2T1OPm3vqDaC.png)

> 4. 双方配置路由表，添加指向对方 VPC 的路由

| **目标**      | **目标**                              | **状态** |
| :------------ | :------------------------------------ | :------- |
| 172.29.0.0/16 | local                                 | 活动     |
| 172.30.0.0/16 | pcx-0c01145dba0f843c0(对等连接 - A2B) | 活动     |

demo-b

| **目标**      | **目标**                              | **状态** |
| :------------ | :------------------------------------ | :------- |
| 172.29.0.0/16 | pcx-0c01145dba0f843c0(对等连接 - A2B) | 活动     |
| 172.30.0.0/16 | local                                 | 活动     |

> 5. 更改 VPC DNS 解析配置

![image-20221025160442316.png](https://s2.loli.net/2022/11/01/qbra7su3FeOUTpS.png)

## 私有子网访问公网

1. 创建 NAT 网关
    + NAT 网关放在 Public 子网中
    + 分配一个弹性 ip
2. 编辑私有子网路由表指向 nat 网关的默认路由

| **目标**    | **目标**                                                     |
| :---------- | :----------------------------------------------------------- |
| 0.0.0.0/0   | [nat-0ddaf13e0696c8d6a](https://ap-northeast-1.console.aws.amazon.com/vpc/home?region=ap-northeast-1#NatGateways:natGatewayId=nat-0ddaf13e0696c8d6a) |
| 10.0.0.0/16 | local                                                        |

## EC2 使用私网 VPC 访问 S3

### 创建 S3 终端节点

1. 创建 IAM 角色

2. 创建 VPC，一个公有子网、一个私有子网

3. 创建 EC2，一个放在公有子网下，一个放在私有子网下（附加 IMA 角色）

4. 测试私有子网下的 EC2 访问 S3

5. 创建终端节点

    ```json
    name: S3EndPoint
    service name: com.amazonaws.ap-northeast-1.s3 gateway
    vpc: xushengjin-vpc-demo1
    routetable: Private1RouteTable
    policy: full_control
    ```

6. 测试私有子网下的 EC2 访问 S3

    ```bash
    aws s3 ls
    ```

## VPC 终端节点访问 SQS

## VPC 中的安全组和和 VPC 中的 Network ACL 的区别

+ 作用范围不同：
    1. 安全组运行在 EC2 级别，指定允许传入或传出 EC2 实例的流量
    2. Network ACL 运行在子网级别，评估进出某个子网的流量

+ 筛选流量范围不同
    1. 安全组可以针对  EC2 进行
    2. Network ACL 不能筛选同一子网中实例之间的流量

+ 筛选规则不同
    1. Network ACL 执行无状态筛选
    2. 安全组则执行有状态筛选

有状态筛选和无状态筛选有什么区别？

安全组放行一条规则，但 Network ACL 需要放行两条规则。

有状态筛选**可跟踪请求的来源**，并可自动允许将请求的回复返回到来源计算机。

例如，允许入站流量进入 Web 服务器上的 TCP 端口 80 的有状态筛选器将允许返回流量（通常为编号较高的端口，如目标 TCP 端口 63、912）通过客户端与 Web 服务器之间的有状态筛选器。筛选设备维护一个状态表，跟踪来源和目标端口编号与 IP 地址。**筛选设备上仅需要一条规则：允许流量进入 Web 服务器的 TCP 端口 80。**

无状态筛选则相反，仅检查来源或目标 IP 地址和目标端口，而忽略流量是新请求还是对请求的回复。上例中的筛选设备上需要实施两条规则：**一条规则用于允许流量在 TCP 端口 80 上传入 Web 服务器，另一条规则用于允许流量传出 Web 服务器（TCP 端口范围 49、152 到 65、535）。**



这两个都是访问控制，差别在于，SG 有会话特性，也就是支持自反，对于一个被放行的会话，不会去阻挡反向连接；Network ACL 看上去就是工作在普通三层设备上的 ACL，没有自反。**没有自反的 ACL 基本等于废物**，所以在我们的实践中，Network ACL 的入站和出站都是允许 `0.0.0.0/0`，安全控制完全依赖 SG 进行。

### 创建流日志 FlowLogs

> 示例文档：https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/flow-logs-cwl.html

1. 创建 IAM 角色

    + 编辑内联策略

        ```json
        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams"
              ],
              "Resource": "*"
            }
          ]
        }   
        ```

    + 信任策略

        ```json
        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Principal": {
                "Service": "vpc-flow-logs.amazonaws.com"
              },
              "Action": "sts:AssumeRole"
            }
          ]
        } 
        ```

2. 创建 CloudWatch 日志组

3. VPC 中创建流日志

4. 查看日志记录情况

> id 为 699568033502 的账户，eni 接口为 eni-04666156f3dff211e，源地址为 180.169.220.200 允许访问私有 ip 地址 10.0.1.38 的 22 端口

```bash
1667788443000 2 699568033502 eni-04666156f3dff211e 180.169.220.200 10.0.1.38 9141 22 6 34 5109 1667788443 1667788492 ACCEPT OK
```

> id 为 699568033502 的账户，eni 接口为 eni-04666156f3dff211e，源地址为 167.248.133.163 拒绝访问 7129 端口

```bash
1667788443000	2 699568033502 eni-04666156f3dff211e 167.248.133.163 10.0.1.38 7129 30017 6 1 44 1667788443 1667788492 REJECT OK

```



## AWS 中国区 ICP 备案

1. 新增备案：公司主体下没有任何备案记录，没有备案过任何网站或域名。 
2. 新增网站：公司主体下有其他网站做过备案，但需要再添加新网站。 
3. 新增接入：网站已在其他接入商完成备案，现需要增加新接入商。

### 需要的材料

+ **公司营业执照**

## 域名解析 --- Route53

## AWS CloudFormation 管理工具

+ 模板 

    json、yaml

    EC2、块设备

+ 堆栈

    相关资源的集合

+ 更改集

    需要更改的堆栈资源的集合

> LAMP Stack
>
> AWS CloudFormation 示例模板 LAMP_Single_Instance：使用单个 EC2 实例和本地 MySQL 数据库创建 LAMP 堆栈进行存储。此模板演示如何使用 AWS CloudFormation 引导脚本安装在实例启动时部署 Apache web 服务器、PHP 和 MySQL 所需的包和文件 ** 警告 ** 此模板创建 Amazon EC2 实例。如果您使用此模板创建堆栈，则将为您使用的 AWS 资源计费。
>
> S3 URL: https://s3-external-1.amazonaws.com/cloudformation-templates-us-east-1/LAMP_Single_Instance.template

```bash
MyDatabase
qweasdzxc123
root
```

### 模板分为七大块

| 部分                     | 说明                         |
| ------------------------ | ---------------------------- |
| AWSTemplateFormatVersion | 版本，可选                   |
| Description              | 描述，必须紧跟 Version 部分  |
| Parameters               | 创建堆栈时讲自定义值传入模板 |
| Mappings                 |                              |
| Resources                | 必填                         |
| Outputs                  |                              |
| Metadata                 | 任意 json、yaml 对象         |


![image.png](https://s2.loli.net/2022/09/29/eVxJUpAHtYZDMaE.png)

## Lambda

无服务器 Serverless

### 触发 Lambda 事件

+ 在 S3 中上传、删除图片
+ DynamoDB 中插入数据
+ Alexa 语音调用智能设备

**顾客请求**

| serialNumber | timeStamp | clickType |
| ------------ | --------- | --------- |
| Gxxx         | 2022-9-30 | SINGLE    |

| serialNumber | location | phoneNumber    |
| ------------ | -------- | -------------- |
| Gxxx         | Laptop   | +8618196761224 |

## Elastic BeanStalk

> 文档：https://docs.aws.amazon.com/zh_cn/elasticbeanstalk/latest/dg/GettingStarted.html
>
> Docker 官方镜像源：https://hub.docker.com/

快速部署和管理应用程序，而不必考虑运行这些应用程序的基础设施

Elastic BeanStalk 本身不付费，但会为创建的资源付费

1. 部署 httpd 服务

    + success 版本
    + error 版本
    + 压缩 success.zip、error.zip

    目录架构

    ```bash
    tree /F
    C:.
    │  error.zip
    │  success.zip
    │
    ├─error
    │      Dockerfile
    │      index.html
    │
    └─success
            Dockerfile
            index.html
    ```

    success/Dockerfile

    ```dockerfile
    FROM httpd:2.4
    COPY ./index.html /usr/local/apache2/htdocs/
    EXPOSE 80
    ```

    success/index.html

    ```html
    <!DOCTYPE html>
    <html>
    <head>
    	<meta charset="utf-8">
    	<meta name="viewport" content="width=device-width, initial-scale=1">
    	<title>success</title>
    </head>
    <body>
    	<h1>this is the success envent</h1>
    </body>
    </html>
    ```

    error/Dockerfile

    ```dockerfile
    FROM httpd:2.4
    COPY ./index.html /usr/local/apache2/htdocs/
    EXPOSE 80
    CMD ["nginx"]	# 故意报错
    ```

    success/index.html

    ```html
    <!DOCTYPE html>
    <html>
    <head>
    	<meta charset="utf-8">
    	<meta name="viewport" content="width=device-width, initial-scale=1">
    	<title>error</title>
    </head>
    <body>
    	<h1>this is the error envent</h1>
    </body>
    </html>
    ```

    

2. 创建 Elastic Beanstalk

    + 名称：xushengjin

    + Docker 环境
    + 上传 `success.zip` 源代码
    + 等待创建完成

3. 查看事件过程 --- 倒叙查看

    2.	环境运行状况已从待处理转变为正常。初始化在 19 秒前完成，耗时 3 分钟。
    3.	成功启动环境：Xushengjin-env
    4.	应用程序可在 Xushengjin-env.eba-2zp7t6i5.ap-northeast-1.elasticbeanstalk.com 获得。
    5.	实例部署成功完成。
    6.	向您的环境添加了实例 [i-08845dc21f9b007d5]。
    7.	创建的负载均衡器侦听器名为：arn:aws:elasticloadbalancing:ap-northeast-1:699568033502:listener/app/awseb-AWSEB-1RXOY0BHYIQHK/e74f6ff72452a12a/e73eab71029b5837
    8.	创建的负载均衡器名为：arn:aws:elasticloadbalancing:ap-northeast-1:699568033502:loadbalancer/app/awseb-AWSEB-1RXOY0BHYIQHK/e74f6ff72452a12a
    9.	创建了名为：awseb-e-nhg7p87jpx-stack-AWSEBCloudwatchAlarmHigh-1KVH3199LAGW2 的 CloudWatch 警报
    10.	创建了名为：awseb-e-nhg7p87jpx-stack-AWSEBCloudwatchAlarmLow-YZ5DH22XFV95 的 CloudWatch 警报
    11.	创建的 Auto Scaling 组策略名为：arn:aws:autoscaling:ap-northeast-1:699568033502:scalingPolicy:002b1f88-64fb-4803-a01b-8afc1c4b864e:autoScalingGroupName/awseb-e-nhg7p87jpx-stack-AWSEBAutoScalingGroup-562OA52TFZKaw2: -e-nhg7p87jpx-stack-AWSEBAutoScalingScaleUpPolicy-Bg8mnsmN1MsG
    12.	创建的 Auto Scaling 组策略名为：arn:aws:autoscaling:ap-northeast-1:699568033502:scalingPolicy:c60630b2-1f83-4086-a6a0-a9874fbbca1b:autoScalingGroupName/awseb-e-nhg7p87jpx-stack-AWSEBAutoScalingGroup-562OA52TFZK2:policyName/scyaweb: -e-nhg7p87jpx-stack-AWSEBAutoScalingScaleDownPolicy-rUy4k815yJtu
    13.	等待 EC2 实例启动。这可能需要几分钟的时间。
    14.	创建名为：awseb-e-nhg7p87jpx-stack-AWSEBAutoScalingGroup-562OA52TFZK2 的 Auto Scaling 组
    15.	环境健康已过渡到待处理。正在进行初始化（运行 10 秒）。没有实例。
    16.	创建的安全组名为：awseb-e-nhg7p87jpx-stack-AWSEBSecurityGroup-149CMPTU784QY
    17.	创建名为：sg-0e77302bfd3d634c6 的安全组
    18.	创建的目标组名为：arn:aws:elasticloadbalancing:ap-northeast-1:699568033502:targetgroup/awseb-AWSEB-8UYT53L05JZK/a5cd7c83ceab8f77
    19.	使用 elasticbeanstalk-ap-northeast-1-699568033502 作为环境数据的 Amazon S3 存储桶。
    20.	createEnvironment 正在启动。

4. 访问 URL

    会发现 Elastic BeanStalk 会自动创建一台 EC2 运行

    http://xushengjin-env.eba-2zp7t6i5.ap-northeast-1.elasticbeanstalk.com/

    ![67.png](https://s2.loli.net/2022/11/05/wUxvaYLRD5TdX3A.png)

5. 接着重新部署 `error.zip`，并访问 URL

    + 日志报错
    + 访问 URL 提示 502
    + ec2 实例被终止

    ![68.png](https://s2.loli.net/2022/11/05/iHTm5jtyqUelnMX.png)

6. 重新上传 `success.zip`
7. 更改配置目录下的 `滚动更新和部署` 模式
    + 将部署策略：一次部署全部，修改为额外批量滚动
8. 上传 `error.zip`，访问 URL
    + 查看日志发现，先是报错，然后又重新回滚到上一个版本
    + 访问 URL 正常

## CloudFront - CDN

内容分发网络

### 应用场景

+ 静态/动态网站加速 
+ 静态文件分发 
+ 文件下载 (游戏、视频、软件更新等) 
+ 视频流（直播/点播 )

### CloudFront 地域限制

![cloudfront_limit.gif](https://s2.loli.net/2022/10/09/684nvfzbHdKJtqc.gif)

### 让 CloudFront 的缓存失效原因：

1. 版本控制 --- version 或 md5 做版本控制

2. `Cloudfront Invalidation` 功能

 **第一种**

![image-20201208224400712](https://i.loli.net/2020/12/08/HYMNagK49srV2OJ.png)

**第二种**

![cloudfront_Invalidation.gif](https://s2.loli.net/2022/10/09/xjwBdSTyIJ7bgq8.gif)



## ECR

**ECR** 完全托管的 Docker 容器资料库

AWS 文档：https://docs.aws.amazon.com/zh_cn/AmazonECR/latest/userguide/what-is-ecr.html

示例文档：https://docs.aws.amazon.com/zh_cn/AmazonECR/latest/userguide/getting-started-cli.html



+ 网页访问

![33.jpg](https://s2.loli.net/2022/10/11/8cmZFdHEVsBC6x7.jpg)

+ curl 测试

![34.jpg](https://s2.loli.net/2022/10/11/M8wxof4zibsPUGt.jpg)



1. ##### 检索身份验证令牌并向注册表验证 Docker 客户端身份。

    使用 AWS CLI:

    ```bash
    aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 699568033502.dkr.ecr.ap-northeast-1.amazonaws.com
    ```

2. ##### 使用 Dockerfile 并生成镜像

    touch dockerfile

    ```dockerfile
    FROM public.ecr.aws/docker/library/ubuntu:18.04
    
    # Install dependencies
    RUN apt-get update && \
     apt-get -y install apache2
    
    # Install apache and write hello world message
    RUN echo 'Hello World!' > /var/www/html/index.html
    
    # Configure apache
    RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh && \
     echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh && \
     echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \ 
     echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \ 
     chmod 755 /root/run_apache.sh
    
    EXPOSE 80
    
    CMD /root/run_apache.sh
    ```

    构建镜像

    ```bash
    docker build -t hello-world .
    ```

3. ##### 生成完成后，标记您的映像，以便将映像推送到此存储库:

    ```bash
    docker tag hello-world:latest 699568033502.dkr.ecr.ap-northeast-1.amazonaws.com/ecr-demo2:latest	# 这里最后应该为仓库名
    ```

4. ##### 运行以下命令将此映像推送到您新创建的 AWS 存储库:

    ```bash
    docker push 699568033502.dkr.ecr.ap-northeast-1.amazonaws.com/ecr-demo2:latest # 这里最后应该为仓库名
    ```

    ![35.png](https://s2.loli.net/2022/10/11/vNLo1cneIwjKB23.png)

    ![36.png](https://s2.loli.net/2022/10/11/1lxWgRTZXaYU7AG.png)

5. 控制台查看镜像推送情况

    ![37.png](https://s2.loli.net/2022/10/11/eTtIPJLw4anVyQ7.png)

### 拉取镜像

```bash
docker pull 699568033502.dkr.ecr.ap-northeast-1.amazonaws.com/ecr-demo2:latest
```



## ECS 创建容器服务

lastic Container Service (ECS)

+ task - 最小的单位

+ task Definition - 任务定义

+ Cluser - 底层硬件的资源组合

    

## AutoScaling

+ 组 - 弹性的 EC2 实例
+ 启动配置
+ 扩展计划



## 扩展计划

增减服务器的方法

1. 始终保持当前实例等级 
2. 手动扩展
3. 按计划扩展
4. 根据需求进行扩展 --- 更高级 --- 策略



1. 设置策略，当 CPU 占有率到达 75% 后新增 EC2 实例

    ![29.png](https://s2.loli.net/2022/10/11/2CApcar8HJye5qM.png)

2. 设置自动扩容最大、最小数量

    ![image.png](https://s2.loli.net/2022/10/11/PJwxDenGQZo1hM7.png)

3. 在 EC2 中让 CPU 占有率过高，一行命令让 CPU 占用率达到 100％

    相关博文解释 https://www.cnblogs.com/my-first-blog-lgz/p/13884537.html

    ![31.png](https://s2.loli.net/2022/10/11/mF3cDKnzyvRo9Z7.png)

    ```bash
    for i in `seq 1 $(cat /proc/cpuinfo |grep "physical id" |wc -l)`; do dd if=/dev/zero of=/dev/null & done
    
    
    [root@ip-172-31-82-132 ec2-user]# top
    top - 09:29:03 up  1:06,  1 user,  load average: 1.00, 0.98, 0.70
    Tasks: 101 total,   2 running,  57 sleeping,   0 stopped,   0 zombie
    %Cpu(s): 99 us, 45.5 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem :   988932 total,   387712 free,    80916 used,   520304 buff/cache
    KiB Swap:        0 total,        0 free,        0 used.   769156 avail Mem
    
    ```

4. 等待 Auto Scaling 自动扩容

    ![27.png](https://s2.loli.net/2022/10/11/2bHVGsdtqDcflh4.png)

    邮件告警 - 扩容信息

    ![_LYANHX26JIC~4SXF2`K8_N.png](https://s2.loli.net/2022/10/11/dVJPq2bNXMKihUB.png)

5. 服务器停止 dd 进程

    ```
    top
    kill 3509
    ```

6. 等待 Auto Scaling 自动缩减

    ![32.png](https://s2.loli.net/2022/10/11/M7WfcAjZHJbUedy.png)



## EC2 RI 预留实例

## 多账户策略

提供了最大数量的资源和足够程度的安全隔离。

+ 身份账户体系结构

+ 日志账户体系结构
+ 发布账户体系结构
+ 账单结构

#### 一个 AWS 账号实现隔离

东京区域部署开发环境、首尔区域部署生产环境

企业不同的环境部署在 AWS 不同区域中实现资源隔离

这种做法隔离程度不够，也不是最佳实现， IAM 用户以及相应策略分配给了开发人员，策略没有进行合理的安全配置，开发人员是可以访问到首尔区域的生成环境的，若发生误操作的情况，会影响生产环境的业务稳定性。

企业配置多个 AWS 账户是最佳实践

+ 开发人员：单独一个 AWS 账户

+ 生产环境：单独一个 AWS 账户部署资源

可以避免开发账户意外访问生产环境资源

如果有需求访问生产环境资源，可以需要开对应的策略权限

## EC2

### EC2 型号

计算、内存和联网资源三方面的平衡，可用于各种不同的工作负载

https://aws.amazon.com/cn/ec2/instance-types/

| 型号       | 系列   | 说明 |
| ---------- | ------ | ---- |
| 计算优化型 | C 系列 |      |

### EC2 启动类型

+ 按需实例（On Demand），适用于短期且不能中断的任务

+ Spot 实例，价格低于按需实例，适用于可中断的任务

    适用于数据分析、后台作业、批处理

+ 预留实例，承诺 1 年或 3 年，来获得较高的折扣（最短一年）

    + 标准预留实例，可修改实例大小等
    + 可转换预留实例，可修改实例类型、实例系列
    + 计划预留实例，一年为期限的指定开始时间和持续时间的

+ 专用实例



### 状态检查

1. 系统状态检查

    AWS  参与修复

    + AWS 电源

    + 网络或软件系统

2. 实例状态检查

    此检查用于验证您实例的操作系统是否在接收流量。

    客户自行修复

#### 状态告警 demo

1. 创建一个名为 `reboot-test` 的警报

    #####  操作已启用的操作

    | 类型 | 描述                                                         | 配置 |
    | :--- | :----------------------------------------------------------- | :--- |
    | 通知 | 当处于 告警中 时，发送消息至主题“Default_CloudWatch_Alarms_Topic” | -    |
    | EC2  | 当处于 告警中 时，重启 ID 为“i-08247b7ccf0f3f54b”的实例      | -    |

2. 将状态设置为警报

    + --alarm-name "reboot-test" 为警报名称
    + --state-value ALARM 设置为 `ALARM` 告警装填
    + --region us-east-1 时区为 us-east-1
    + \ 分隔符

    ```bash
    aws cloudwatch set-alarm-state --alarm-name "reboot-test" \
    --state-value ALARM \
    --state-reason "test" \
    --region us-east-1
    ```

3. 查看 `cloudwatch` 日志

    ![39.png](https://s2.loli.net/2022/10/14/gSD7MOZQRsjN38G.png)

    ![40.png](https://s2.loli.net/2022/10/14/OhT5oDrqiwMaEyY.png)

### 置放群组

> 在您启动新的 EC2 实例时，EC2 服务会尝试以某种方式放置实例，以便将所有实例分布在基础硬件上以最大限度减少相关的故障。您可以使用*置放群组*影响如何放置一组*相互依赖的*实例，从而满足您的工作负载需求。

放置策略

- *集群* – 将一个可用区中靠近的实例打包在一起。通过使用该策略，工作负载可以实现所需的低延迟网络性能，以满足 HPC 应用程序通常使用的紧密耦合的节点到节点通信的要求。

    > + 一个集群置放群组不能跨过多个可用区
    > + 集群置放群组中的实例，对于实例类型是有要求的
    > + 集群置放群组中的两个实例之间的最大网络吞吐量流量速度受两个实例中的
    >     较慢实例限制
    > + 使用单个启动请求在置放群组中启动所需的实例数。对置放群组中的所有实
    >     例使用相同的实例类型
    > + 如果您停止置放群组中的某个实例，然后重启该实例，则其仍将在该置放群
    >     组中运行。
    > + 如果您在已有正在运行的实例的置放群组中启动实例时接收到容量错误信息，
    >     请在该置放群组中停止并启动所有实例，然后尝试再次启动。

- *分区* – 将实例分布在不同的逻辑分区上，以便一个分区中的实例组不会与不同分区中的实例组使用相同的基础硬件。该策略通常为大型分布式和重复的工作负载所使用，例如，Hadoop、Cassandra 和 Kafka。

    对于每个可用区，一个分区置放群组最多可具有 `7` 个分区

- *分布* – 将一小组实例严格放置在不同的基础硬件上以减少相关的故障。

    将每个实例放置在不同的机架上
    关键业务的多个实例使用分布置放群组
    可以跨越同一区域中的多个可用区
    每个群组在每个可用区中最多有 7 个正在运行的实例

![image-20221014134441218](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221014134441218.png)

### EC2 Spot 实例

#### Spot 队列



### 设计弹性伸缩 - Auto Scaling

+ 垂直扩展

    m5.large ---> m5.24xlarge

+ 水平扩展

    m5.large + m5.large + m5.large

    自动扩展 - `Auto Scaling`

> 需求：某公司开发的办公系统，工作日负载价高、周末负载较低

![image-20221014144038545.png](https://s2.loli.net/2022/11/01/iN8yAtsw71lSBvX.png)

1. 创建两个 `CloudWacth` 警报

    + 增加 EC2 的指标，当 CPU 使用率，持续一分钟平均使用率超过 75%，就会增加一台 EC2 实例
    + 缩减 EC2 的指标，当 CPU 使用率，持续一分钟平均使用率小于 20%，会就缩减一台 EC2 实例

    ![image-20221014142837827.png](https://s2.loli.net/2022/11/01/mJtayopbRETPshS.png)

    ![image-20221014143722975.png](https://s2.loli.net/2022/11/01/QhENoWm4rDC2Rls.png)

####  Auto Scaling 关键组件

+ 配置模板
    + AMI - CentOS、RedHat
    + 实例类型
    + 安全组
    + 角色等
+ EC2 Auto SCaling 组
    + 所使用的配置模板
    + 最大数量、最小数量、所需数量
    + 可用区、子网
    + 负载均衡器
    + Cloud Watch
+ 扩展选项

![42.png](https://s2.loli.net/2022/11/01/KBfXrMte4Aq21yp.png)

## ELB（Elastic Load Balancing）负载均衡器

比较：https://aws.amazon.com/cn/elasticloadbalancing/features/#compare

需要 ElB 支持静态 ip 来实现某些功能 --- 网络负载均衡器

> demo：
>
> + EC2-A ---> Hello World! --- Server 1
> + EC2-B ---> Hello World! --- Server 2
>
> + Classic Load Balancer ---> http://xsj-elb-test-763812980.us-east-1.elb.amazonaws.com/
>
> 注意：安全组 80 端口开放

### 安装 Nginx 服务器

1. 使用 amazon-linux-extras 查看是否有 nginx

    ```bash
    sudo amazon-linux-extras install | grep nginx
    
    38  nginx1=latest            enabled      [ =stable ]
    ```

2. 安装 nginx

    ```bash
    sudo amazon-linux-extras install nginx1
    ```

3. 启动 nginx 进程

    ```bash
    systemctl start nginx
    systemctl status nginx
    ```

4. 修改 index.html 文件

    ```bash
    vim /usr/share/nginx/html/index.html
    H1 标题改掉
    ```

5. 访问 ip

6. 演示内容

    使用 `ECR` 那一章推送的 `hello-world` 镜像库搭建

    ```bash
    docker pull 	# 拉去镜像
    docker exec -it container-id # 进入容器内部
    cat > /var/www/html/index.html << EOF	# docker 内部没有 vim，使用 cat 写入文件内容
    Hello World --- Server 2
    EOF
    ```


![loadBalancer.gif](https://s2.loli.net/2022/10/17/8WAhk76aONU5gy2.gif)

### 应用程序负载均衡器 --- 基于路径的路由功能

> 基于路径的路由功能 --- 基于 HTTP 标头的 URL 路径 | 路由 | 客户端请求
>
> xushengjin.com/images/ ---> Server 1
>
> xushengjin.com/about/ ---> Server 2

1. 负载均衡器

2. 目标群组
	+ elb-test 	---> Server 1 
	+ \*images*  ---> Server 1 ---> 重定向目标群组 images
	+ \*about*   ---> Server 2 ---> 重定向目标群组 about

> about.txt

```json
{
	"Version": "2012-10-17",
	"Id": "PreventHotLinking",
	"Statement": [
		{
			"Sid": "Http Referer Policy example",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::my-xushengjin-bucket-2/*",
			"Condition": {
				"StringLike": {
					"aws:Referer": [
						"https://xushengjin.cn/*",
						"http://xsj-static-site.s3-website-us-east-1.amazonaws.com/*",
						"http://ec2-54-80-191-166.compute-1.amazonaws.com/",
						"http://ec2-44-211-143-103.compute-1.amazonaws.com/"
					]
				}
			}
		},
		{
			"Sid": "2",
			"Effect": "Allow",
			"Principal": {
				"AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E2Y0SFNZJ8I5YD"
			},
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::my-xushengjin-bucket-2/*"
		}
	]
}
```

![image-20221018110530291.png](https://s2.loli.net/2022/11/01/jLpKe8ic6qST1QB.png)

![image-20221018110547327.png](https://s2.loli.net/2022/11/01/aqPQMf7eVY9R15s.png)

> 实验结果
>
> 1. Server 1 为 girl.jpg
> 2. Server 2 为 about.txt
>
> 3. 访问 application-elb.dns 默认为 Sever 1 内容
> 4. application-elb.dns.images.girl.jpg 重定向到 Server 1.girl.jpg
> 5. application-elb.dns.about.about.txt 重定向到 Server 2.about.about.txt

![application-elb.gif](https://s2.loli.net/2022/11/01/6OfJtrwCEsNnkbK.gif)

### 侦听器

1. 侦听器是负载均衡器用于**检查连接请求**的的进程
2. 侦听器使用配置的**协议和端口**检查来自客户端的连接请求，HTTP 协议、80 端口
3. 侦听器定义的**规则决定**负载均衡器如何将请求**路由到**其已注册目标

![image-20221019140837700.png](https://s2.loli.net/2022/11/01/nzPHq4e1VJSCMhN.png)

### 目标组

1. 目标组使用指定的**协议和端口号**将请求路由到一个或多个**注册目标**，比如 EC2 实例
2. 再创建每个侦听器规则时，可以指定**目标组和条件**。满足规则条件时，流量会转发到**相应的目标组**。

负载均衡器、侦听器、目标组、注册目标之间的关系

![基本应用程序负载均衡器的组件](https://docs.aws.amazon.com/zh_cn/elasticloadbalancing/latest/application/images/component_architecture.png)

![image-20221019135945356.png](https://s2.loli.net/2022/11/01/ZhHVkaIYdwLue6J.png)

### 网络负载均衡器 NLB

> 工作在 OSI 四层、传输层（TCP\UDP\TLS）

路由算法：

可以配置目标组级别使用的路由算法。默认路由算法为**轮询路由算法**；或者，可以指定**最少未完成请求路由算法**。

1. 对于TCP流量，基于协议、源 IP 地址、源端口、目标 IP 地址、目标端口和 TCP 序列号，
    使用**流哈希算法**选择目标。
2. 来自客户端的 TCP 连接具有不同的源端口和序列号，这样可以路由到不同的目标。
3. 每个单独的 TCP 连接在连接的有效期内只会路由到单个目标。



## Route 53

> 域名系统 (DNS) Web 服务
>
> 三个主要功能：域注册、DNS 路由、运行状况检查 



| 记录       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| A 记录     | 流量路由到资源的 IPv4 地址                                   |
| AAAA 记录  | 流量路由到资源的 IPv6 地址                                   |
| CNAME 记录 | 流量路由到另外一个域名                                       |
| ALIAS 记录 | 流量路由到 AWS 资源、负载均衡器、CloudFront、S3 bucket、支持顶级域名上创建别名记录 |



###  TTL 生命周期

+ TTL 是指缓存 DNS 记录的时间（秒）
+ 较大的 TTL: DNS 更少请求、更低费用但记录更新生效慢
+ 较小的 TTL：DNS 更多请求和流量，记录更新生效快，适合做迁移

### 简单路由策略

+ 将流量路由到单个资源，比如 WEB 服务器
+ 简单路由策略无法附加健康状态检查
+ 如果您在同一记录中指定多个值，则 Route53
    将所有值以随机顺序返回客户端（例如，Wb浏览器）--- 就是可以设置多个 ip，但随机返回，也是一种负载均衡

### 加权路由策略

+ 允许将多个资源关联至单个域名，并可配置向每个资源路由多少流量

+ 适合负载均衡、测试软件新版本

+ 两个 AWS 区域分配流量，做一些负载均衡的实现

+ 可以关联健康状态检查

+ 配置的权重相加不需要必须等于 100

    ![image-20221020101335908.png](https://s2.loli.net/2022/11/01/InJS31NXVrj4alx.png)

### 故障转移路由策略

> 在 Route53 创建相应记录并指定 forover 作为路由策略

+ 主资源健康状态检查健康，就会访问资源

+ 主资源健康状态检查有问题时就会自动切换到备用资源

![back-route-policy.gif](https://s2.loli.net/2022/11/01/nGdRgS7PTpLFmN4.gif)

### 基于延迟的路由策略

> 应用程序部署在 AWS 多个区域中	

+ 从用户到延迟最低的AWS区域处理用户的请求
+ 评估延迟是指从用户端到 AWS 区域的延迟
+ 添加健康状况检查，提高自动故障迁移能力
+ 延迟是优选的唯一参数

![image-20221020102941206.png](https://s2.loli.net/2022/11/01/dGhVcuSEmUNgMrt.png)

  ### 地理位置路由策略

+ 根据用户的**地理位置**选择提供流量的资源
+ 比如可以配置将来自欧洲的所有用户，路由到位于法兰克福区域的 ELB
+ **创建默认记录**，处理来自未映射到任何位置的以及未创建地理位置记录的的查询
+ 内容本地化或者因法律法规要求，网站或业务只服务于某些国家等需求时非常适用

### 复杂、嵌套记录路由策略

> 延迟路由 + 加权路由

![image-20221020103819153.png](https://s2.loli.net/2022/11/01/76Z5i4vhJRaXjDs.png)

具体配置步骤

1. 先创建加权路由，分配对应权重指向对应的 EC2.ip

2. 创建延迟 alias 记录，alias 配置为前面加权策略的域名

    www.example.com ---> us-east-1-www.example.com

    www.example.com ---> ap-southeast-1-www.example.com

### 多值应答路由策略

> 很多资源提供同一服务

+ 允许将 Route 53 配置返回多个值（WebService.ip）来响应 DNS 查询
+ 多值应答路由允许附加并检查每个资源的运行状态 --- 以便 Route 53 只返回正常的资源的值
+ 每一次多值应答最多返回 `8` 个健康的记录
+ 不能代替负载均衡器



### 私有 DNS （Private DNS）

+ Route 53 作为内部私有 DNS

    set VPC.enableDnsHostnames = True 

    set VPC.enableDnsSupport = True 

+ DNSSEC（保护域不受劫持或者中间人攻击）

    支持将 DNSSEC 用于域注册

    不支持将 DNSSEC 用于 DNS 服务

    想使用 DNSSEC 保护您的域不受劫持，则必须使用其他 DNS 服务提供商或
    在 EC2 上自行配置您自己的 DNS 服务器，比如通过 bind,dnsmasq 等三方应用

+ 第三方域注册商
    可以在第三方注册机构注册域名，然后使用 Route53 提供 DNS 服务
    需要在三方注册机构更新域名的 NS 记录指向 Route53

### Route 53 健康状态检查（Health Checks）

> 监控资源的运行状况，在状态更改时获取通知，配置 DNS 实现自动故障转移

1. 监控终端节点的运行状况检查，如 ELB、WebServer 等其它资源

2. 监控 `CloudWatch` 警报的运行状况检查

    ELB 的健康主机数量、RDS 警报

3. Route 53 的运行状况检查于 `CloudWatch` 指标集成

4. 支持配置运行状况检查为使用字符串匹配的 HTTP 和 HTTPS 运行状况检
    查；该字符串必须完全在响应正文的**前 5120 个字节**

5. 终端节点必须使用 `2xx` 和 `3xx` 的 HTTP 状态代码响应才能通过运行状况检查

6. 使用运行状况检查监控其它运行状况检查的状态

    指定**子运行**状况检查至少有多少项运行状况检查正常时，才认定**父**
    **运行状况**检查状态通过。

![image-20221020111105054.png](https://s2.loli.net/2022/11/01/kiFnRrxh9TJyu4v.png)

7. 运行状况检查是通过 `Internet` 向您的应用、服务器或其他
    资源发起请求

8. 私有子网以及不可路由的资源终端节点是没有办法创建运
    行状况检查的

    > 方案：
    >
    > + 分配公网 ip 地址并使其公网可访问
    >
    > + 运行状况检查监控其依赖的 EC2 实例，比如网站前端，数
    >     据库等假设公网可访问的资源
    > + 基于 CloudWatch 指标创建 CloudWatch 警报，在通过创建
    >     运行状况检查关联这个 CloudWatch 警报

![image-20221020112501507.png](https://s2.loli.net/2022/11/01/HiALg1VQMwKWR8X.png)

#### 通过运行状况检查实现 RDS 跨区域故障转移

![image-20221020112716491.png](https://s2.loli.net/2022/11/01/kNEuDsXAaqFnfeK.png)

#### 将私有托管区域与多个 AWS 账号下的 VPC 关联

> 文档教程: https://aws.amazon.com/cn/premiumsupport/knowledge-center/route53-private-hosted-zone/

+ 设置共享服务的 AWS 账号，在账号下配置一个内部私有的共享 DNS，作为在 AWS 上资源内部 DNS 解析
+ 其它的 AWS 账号 `VPC` 的资源需要访问共享服务账号解析 DNS 请求
    1. 所有 VPC 到共享服务 VPC 都分别创建 **VPC 对等连接**
    2. AWS 管理控制台无法实现，必须使用 **`CLI` 或开发工具包**等编程方式，将这些 VPC 与共享服务中的私有托管区域关联
+ **每个账号的 VPC 都要和共享账号中的私有托管区域做一次关联**

![image-20221020113259013.png](https://s2.loli.net/2022/11/01/5aspvIoyXiZbnuP.png)

## AWS Lambda

> 与其它服务使用：API 网关、Kinesis、DynamoDB、S3、IOT、CloudWatch Events、CloudWatch Logs、SNS、Cognito、SQS

### 使用 Lambda 结合 S3 自动创建缩略图

教程: https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/with-s3-tutorial.html

![image-20221020113820712.png](https://s2.loli.net/2022/11/01/xJRy56PUBTkELIS.png)

### 使用 Lambda 结合 CloudWatch Events 执行 Cron Job

定期执行任务：

+ 以往都是在 EC2 上使用 cron 定期调用，这样就需要提供 EC2 实例，尽管这台 EC2 实例大部分时间不工作，还要为其付费

+ 现在通过定义一个 `CloudWatch Events` 定义一个时间间隔（每分钟、每小时）触发配置的 `Lambda` 函数，执行一个最长 **15 分钟**的任务 --- 无服务器架构 ServerLess 执行 Cron job 的最佳实践

### Lambda 支持的运行环境

+ Node.js、Python、Ruby、Java、Go、.NET 等
+ 通过自定义运行环境来使用其它的编程语言
+ 出现 Docker 相关的场景，优先考虑使用 ECS、Fargate，或者批处理 Batch.

### Lambda 的使用限制

+ 内存：128 MB - 10 GB

+ CPU 与配置的内存成正比的方式线性分配 CPU 处理能力 --- 无法手动指定 CPU 计算能力

> Serverless 库
>
> web application hosting on AWS

+ 函数超时时间为 15 分钟
+ /tmp 目录：最大这次hi 512 MB 存储空间
+ 最大并发数量：1000，可以提交申请增加

### Lambda 的延迟因素

+ 函数运行的并发限制：1000

+ 冷启动时间

    ![image-20221024105427340.png](https://s2.loli.net/2022/11/01/FiM3aesyX1N7Ejp.png)

    第一次 调用Lambda 函数时，要从 S3 下载 code 和依赖项，创建容器并在执行代码前初始化运行环境 --- 这就是冷启动时间

    如果在函数执行期间有新的请求调用 Lambda 函数，函数并发调用，会在分配新的实例处理该事件，新的实例在执行代码前照样需要走冷启动流程，势必会增加响应时间

    1. 预配置并发，预热 Lambda 执行环境
    2. 预配置并发， 可以使用环境来立即执行代码

+ 监控

    Lambda 结合多个 AWS 服务，也需要知道下游 aws，例如 DynamoDB、S3，对于每个 Lambda 情况的话就需要 `X-Ray`，通过可视化的方式跟踪和调试 Lambda 函数和其它服务之间的问题。

### Lambda 安全相关

+ 使用 IAM 管理对 Lambda Api 和资源的访问
+ **执行角色**：想函数授予访问 AWS 服务和资源的权限
+ 基于资源的策略 --- 类似于 S3 的桶策略
    1. 允许其它的 AWS 账户调用或管理 Lambda 函数
    2. 允许 AWS 服务调用或管理 Lambda 函数

### Lambda 网络

#### Lambda 默认部署

> 默认部署在 AWS 的安全的 VPC 中，自己的 VPC 之外的，可以访问 Internet 访问外部 API 以及 DynamoDB 的公有 xxx 地址，如果在这种情况下有个私有 VPC，其中有个只能通过私有地址访问的 RDS，这样 Lambda 是无法访问到这个  RDS。

![image-20221024110943555.png](https://s2.loli.net/2022/11/01/b94HVrhJFyjY2DW.png)

> 解决方案：将 Lambda 函数连接至 VPC

 Lambda 如何访问私网连接的 RDS？

1. 函数在运行时就可以访问 VPC 中的私有资源，Lambda 在函数的 VPC 配置中对 VPC 的每个子网和安全组组合创建一个 DNI，通过在安全组配置相应的安全规则就可以通过私有子网访问到 RDS

![image-20221024111604393.png](https://s2.loli.net/2022/11/01/mtRg2Q9YKoH1Ebf.png)

2. 现在 Lambda 在自己的私有 VPC 内，之前的外部 API 就无法访问了，如果需要访问外部的 API，就需要通过公有子网的 NAT 网关或 NAT 实例，通过 Internet 网关访问外部 API

    ![image-20221024111845099.png](https://s2.loli.net/2022/11/01/jO4W5JDsu61hSlf.png)

3. 函数如何访问 DynamoDB

    + 函数可以通过 IGW 访问 DynamoDB，但是这种方式不是通过私网访问的，还得产生费用
    + 创建 DynamoDB 的网关终端节点，私有子网的 Lambda 函数通过终端节点访问 DynamoDB，需要配置好路由表

    ![image-20221024112116352.png](https://s2.loli.net/2022/11/01/HeVFomZx79vEqzs.png)

### Lambda 日志、监控以及追踪

+ CloudWatch
    1. Lambda 的执行日志都会发送并存储至 Cloudwatch logs
    2. 通过 Cloudwatch 指标，查看 Lambda 的一些指标，如：成功调用、错误率、延迟、超时等等
    3. Lambda 函数的日志写入 Cloudwatch，需要分配函数执行角色，并添加相应的如写入 CloudWatch 权限
+ X-Ray
    1. 可以使用 X-Ray 来追踪 Lambda 函数
    2. 在 Lambda 的配置中开启，AWS 就会为您启动一个 X-Ray 守护进程
    3. 可以使用 AWS SDK 添加 X-Ray 指针
    4. Lambda 函数的执行角色要分配相应的与 X-Ray 访问的权限

### 调用 Lambda 函数

#### 同步调用：使用 API、CLI 以及 API 网关等函数调用

+ 运行并等待响应，函数完成时将结果同步返回
+ 负责确定处理错误的策略，可以重试、将事件发送到队列，或者忽略该错误

### 异步调用

+ 当事件源时 **S3、SNS** 等服务调用函数时，通过异步调用的方式

+ 在异步调用函数时，您不必等待函数代码的响应

+ Lambda 会针对函数**错误重试两次**

+ 确保这个过程是幂等的

+ 死信队列：可以是 SQS 队列或者是 SNS 主题，保存失败的时间供进一步处理

    当所有的重试都失败之后不是丢弃这些时间，而是发送到 SQS 死信队列供后续处理

### 事件映射源

### 配置目标

+ 异步调用：可为成功和失败的事件定义目标
    1. SQS
    2. SNS
    3. Lambda
    4. EventBridge bus
+ AWS 建议在异步调用 Lambda 函数时，就是用**目标配置**来代替**死信队列**
+ 事件源映射：只能配置失败事件的目标，用于接收失败时的事件，详细信息发送到以下
    1. SQS
    2. SNS
+ 处理 SQS 队列中的内容，也可以从 SQS 创建死信队列

### Labmda 函数版本（Versions）

+ 函数版本
    1. 代码
    2. 依赖性
    3. 运行环境
    4. 设置
    5. 环境变量等（发布后不可改变）

+ 可以使用 `版本` 来管理函数的部署
+ $LATEST 是最新的 `未发布版本`
+ 需要发布来使用 Lambda 函数，就需要创建一个函数版本
+ 发布版本后，代码和设置锁定不可改变
+ 修改函数后通过将其发布为新的版本投入使用
+ 每一个版本都有各自的 ARN

### Lambda 函数别名（aliases）

+ Lambda 别名类似于指向特定函数版本的指针
+ 可以定义别名然后分别将其指向不同的 Lambda 版本，如：开发、测试、生产
+ 别名是可改变的，可更改指向的
+ 使用别名支持蓝绿部署，可以为 Lambda 函数分配权重

![image-20221024151025181.png](https://s2.loli.net/2022/11/01/IdimKXDBZ7V6A5s.png)

#### Lambda 函数别名与 API 网关场景

> 不需要调整 API GW 的指向，一直指向的是别名，只需要调整**别名指向的版本**即可
>
> PROD 是生产环境对应版本 v1
>
> TEST 是测试环境对应版本 v2
>
> 当 TEST 测试版本需要上线时，可以为其分配 5% 的流量到 v2 

![image-20221024151601483.png](https://s2.loli.net/2022/11/01/SwqxpgrvXo38Gfd.png)

### Lambda 与 CodeDeploy

+ CodeDeploy 可以实现自动化的 Lambda 别名的流量转移功能
+ SAM（无服务器应用程序模型）上面的这个功能是直接内置的
+  慢慢切换到新版本 v2，要多久，每次多少百分比流量，需要定义部署策略
+ 线性：定义每隔 N 分钟，增加 10% 的流量，直到 100%
    1. Liner10PercentEvery1Minutes
    2. Liner10PercentEvery2Minutes

+ 金丝雀：现切换一定百分比的访问量，如果期间没有错误，然后切换全部

    1. Canary10Percent5Mintes
    2. Canary10Percent10Mintes
    3. ......

+ All-at-once：立即

+ 配置 `Hooks` 钩子，在流量转换开始到新版本之前和流量转换完成后**运行健康检查**

    比如我们配置了转移前的 `Hooks`，在部署 v2 版本时，测试此版本函数是否正常，当流量转移到新版本 v2 ，还会运行一个流量转移后的 `Hooks`，确保一切正常，当测试发现问题时，会自动对版本进行**回滚**



## S3

### S3 的访问策略

1. 基于资源
    + ACL
    + 存储桶策略
2. 基于用户
    + IAM，其它 AWS 账户则不行

> AWS 将 pdf 白皮书放到了 S3 存储桶中，用于任何人的访问
>
> AWS 月成本计算器

```bash
curl -I https://d1.awsstatic.com/whitepapers/aws-web-hosting-best-practices.pdf
curl -I https://calculator.amazonaws.cn/#/addService?nc2=h_ql_pr_calc	
```

```bash

HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Length: 759597
Connection: keep-alive
Date: Tue, 01 Nov 2022 02:10:05 GMT
x-amz-replication-status: COMPLETED
Last-Modified: Wed, 06 Oct 2021 18:27:47 GMT
ETag: "d811c4119544a9a875e0b20cc77e5784"
x-amz-meta-version: 2021-10-06T18:27:00.324Z
x-amz-version-id: BkBjQlpsfjhKEz.9.4HBDwu4dc25PYeF
Accept-Ranges: bytes
Server: AmazonS3			### 看这一行
X-Cache: Miss from cloudfront
Via: 1.1 f10b600ea97ac09e072e022f40ed7078.cloudfront.net (CloudFront)
X-Amz-Cf-Pop: NRT57-P1
X-Amz-Cf-Id: ChrhyOo3Ec6HRn4G28k1sy7M3ICznD7V36AHXstyLRDmStLwHHg22g==

```

#### 基于 ip 地址段的存储桶策略 --- 将访问权限限定为特定 IP 地址

只希望单位的网络中访问存储桶中的内容，除此之外不允许访问，这就需要开放指定的 ip 段

```json
{
    "Version": "2012-10-17",
    "Id": "S3PolicyId1",
    "Statement": [
        {
            "Sid": "IPAllow",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::qianyukang",
                "arn:aws:s3:::qianyukang/*"
            ],
            "Condition": {
                "NotIpAddress": {
                    "aws:SourceIp": [
                        "180.169.220.200/32",
                        "66.249.65.172/32"
                    ]
                }
            }
        }
    ]
}

```

#### 跨账户 S3 存储桶访问

1. 账号以及 AK、SK

| Account name | Account id                                                   |
| ------------ | ------------------------------------------------------------ |
| xusj         | 699568033502    AKIA2FYMBB3PJDPRIG2D        +VQImUHCsFUCFflOz5zTidJYQbLEy9YvoCbdVKue |
| qinxue       | 128292574314    AKIAR3XWVKRVPE2Y3EVX           iKbAaaIWvlexchWT8LN9VcuyPW0lC3dAOANqVrM3 |

2. 配置存储桶策略：

    `s3://dingworld` 存储中的创建者为 xusj

    ```bash
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "Cross",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::128292574314:root"
                },
                "Action": "s3:*",
                "Resource": [
                	"arn:aws:s3:::dingworld",
                    "arn:aws:s3:::dingworld/*"
                ]
            }
        ]
    }
    ```

3. 配置访问密钥

    > CLI 文档：https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-configure-files.html

    ​	在 xusj 账户中的 EC2 中

    ```bash
    aws configure --profile xusj
    aws configure --peofile qinxue
    ```

4. 查看 `~/.aws/credentilas`，`~/.aws/config`

> 存放密钥信息

```bash
[ec2-user@ip-172-31-89-208]$ cat ~/.aws/credentials
[xusj]
aws_access_key_id = AKIA2FYMBB3PJDPRIG2D
aws_secret_access_key = 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
[qinxue]
aws_access_key_id = AKIAR3XWVKRVPE2Y3EVX
aws_secret_access_key = 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
```

> 存放时区信息

```
[ec2-user@ip-172-31-89-208]$ cat ~/.aws/config
[profile xusj]
region = us-east-1
output = json
[profile qinxue]
region = ap-northeast-1
output = json
```

5. 分别使用 xusj、qinxue 访问 `dingworld` 存储桶

    发现都可以访问 `dingworld` 内容
    
    ```bash
    [ec2-user@ip-172-31-89-208 ~]$ aws s3 ls s3://dingworld --profile xusj
    2022-11-01 03:16:20         38 utils.txt
    
    
    [ec2-user@ip-172-31-89-208 ~]$ aws s3 ls s3://dingworld --profile qinxue
    2022-11-01 03:16:20         38 utils.txt
    
    ```
    
6. 使用 qinxue 将 `qinxue.txt` 上传到 `s3://dingworld`

    ```bash
    [ec2-user@ip-172-31-89-208 ~]$ ls
    aws  awscliv2.zip  qinxue.txt
    [ec2-user@ip-172-31-89-208 ~]$
    [ec2-user@ip-172-31-89-208 ~]$ cat qinxue.txt
    name: qinxue
    gender: woman
    age: 28
    [ec2-user@ip-172-31-89-208 ~]$ aws s3 cp qinxue.txt s3://dingworld --profile qinxue
    upload: ./qinxue.txt to s3://dingworld/qinxue.txt		### 显示上传成功
    ```

    ![44.png](https://s2.loli.net/2022/11/01/HroLYlvKgTiZsB9.png)



7. 使用 xusj 将 utils.txt、qinxue.txt 下载到本地

    发现 xusj 可以将自己上传的 utils.txt 下载下来，但 qinxue 上传的 qinxue.txt 无法下载 --- **原因？**

    ```bash
    [ec2-user@ip-172-31-89-208 ~]$ aws s3 cp s3://dingworld/utils.txt . --profile xusj
    download: s3://dingworld/utils.txt to ./utils.txt
    
    [ec2-user@ip-172-31-89-208 ~]$ aws s3 cp s3://dingworld/qinxue.txt . --profile xusj
    fatal error: An error occurred (403) when calling the HeadObject operation: Forbidden
    
    ```



### S3 ACL 基础知识

1. 每个存储桶和对象都有一个子资源而附加的 ACL
2. 当 S3 存储桶中的对象收到访问请求时，AWS S3 会检查与存储桶对象关联的 ACL，已决定是允许还是拒绝访问请求
3. 当我们创建一个存储桶或者一个存储桶对象时，AWS S3 默认会给资源分配一个**资源拥有者完全控制权限**

#### **get-object-acl**

功能：列出存储桶中对象的 acl 内容

1. xusj 查看 utils.txt 的 acl 内容，拥有 `FULL_CONTROL` 权限

    ```bash
    aws s3api get-object-acl --bucket dingworld --key utils.txt --profile xusj
    ```

    ![46.png](https://s2.loli.net/2022/11/01/fKo9xTn7VS4DjBZ.png)

2. xusj 查看 qinxue.txt 的 acl 内容，显示无权限

    qinxue 查看 qinxue.txt 的 acl 内容，拥有 `FULL_CONTROL` 权限

    ```bash
    aws s3api get-object-acl --bucket dingworld --key qinxue.txt --profile xusj
    
    An error occurred (AccessDenied) when calling the GetObjectAcl operation: Access Denied
    
    aws s3api get-object-acl --bucket dingworld --key qinxue.txt --profile qinxue
    ```

    ![47.png](https://s2.loli.net/2022/11/01/zmAOqusaBP8o1bd.png)

#### 标准 ACL

​	Amazon S3 支持一系列预定义的授权，称为*标准 ACL*。每个标准 ACL 都有一组预定义的被授权者和许可。下表列出了一系列标准 ACL 和相关联的预定义授权。

| 标准 ACL                    | 适用于       | 添加到 ACL 的权限                                            |
| :-------------------------- | :----------- | :----------------------------------------------------------- |
| `private`                   | 存储桶和对象 | 所有者将获得 `FULL_CONTROL`。其他人没有访问权限 (默认)。     |
| `public-read`               | 存储桶和对象 | 所有者将获得 `FULL_CONTROL`。`AllUsers` 组 (参阅 [谁是被授权者？](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/acl-overview.html#specifying-grantee)) 将获得 `READ` 访问权限。 |
| `public-read-write`         | 存储桶和对象 | 所有者将获得 `FULL_CONTROL`。`AllUsers` 组将获得 `READ` 和 `WRITE` 访问权限。通常不建议在存储桶上授予该权限。 |
| `bucket-owner-read`         | 对象         | 对象所有者将获得 `FULL_CONTROL`。存储桶拥有者将获得 `READ` 访问权限。如果您在创建存储段时指定此标准的 ACL，Amazon S3 将忽略它。 |
| `bucket-owner-full-control` | 对象         | 对象所有者和存储桶拥有者均可获得对对象的 `FULL_CONTROL`。如果您在创建存储段时指定此标准的 ACL，Amazon S3 将忽略它。 |

3. 使用 qinxue 上传一个新文件 acl.txt 并指定为 `bucket-owner-full-control`

    ```bash
    aws s3 cp acl.txt s3://dingworld/ --acl bucket-owner-full-control --profile qinxue
    ```

    返回结果如下：

    ```bash
    upload: ./acl.txt to s3://dingworld/acl.txt
    ```

4. 使用 qinxue 查看 acl.txt 的 acl 内容

    ```bash
    aws s3api get-object-acl --bucket dingworld --key acl.txt --profile qinxue
    ```

    返回结果如下：

    **明明是 qinxue 上传的为啥，对象所有权为 xusj？和实验结果不符**

    ```bash
    {
        "Owner": {
            "DisplayName": "dc_awsvap_global11",		###
            "ID": "d8963dc1ca6406eddbc7d1840d55aa4df0e1fdf081b0a6a475ac0a7c8efd6f5a"
        },
        "Grants": [
            {
                "Grantee": {
                    "DisplayName": "dc_awsvap_global11",	###
                    "ID": "d8963dc1ca6406eddbc7d1840d55aa4df0e1fdf081b0a6a475ac0a7c8                                                           efd6f5a",
                    "Type": "CanonicalUser"
                },
                "Permission": "FULL_CONTROL"
            }
        ]
    }
    
    ```

    找寻一番，原来是创建存储桶时指定的对象所有权问题

    ![49.png](https://s2.loli.net/2022/11/01/ZKsF7GJLfR2CDjl.png)

5. 对象所有权修改为：对象编写者优先即可，qinxue 重新上传 acl-2.txt 文件

    ```bash
    aws s3 cp acl-2.txt s3://dingworld/ --acl bucket-owner-full-control --profile qinxue
    upload: ./acl-2.txt to s3://dingworld/acl-2.txt
    
    aws s3api get-object-acl --bucket dingworld --key acl-2.txt --profile qinxue
    ```

    返回结果如下：

    ```bash
    {
        "Owner": {
            "DisplayName": "2513319345",	### 对象拥有者为 qinxue
            "ID": "8d22d91847b9930f8813023c0dbde416447dc545ae82af5fd1ec4db81716026a"
        },
        "Grants": [
            {
                "Grantee": {
                    "DisplayName": "2513319345",	### qinxue
                    "ID": "8d22d91847b9930f8813023c0dbde416447dc545ae82af5fd1ec4db81716026a",
                    "Type": "CanonicalUser"
                },
                "Permission": "FULL_CONTROL"
            },
            {
                "Grantee": {
                    "DisplayName": "dc_awsvap_global11",	### xusj
                    "ID": "d8963dc1ca6406eddbc7d1840d55aa4df0e1fdf081b0a6a475ac0a7c8efd6f5a",
                    "Type": "CanonicalUser"
                },
                "Permission": "FULL_CONTROL"
            }
        ]
    }
    
    ```



### S3 存储桶的跨区域复制（CRR）

假设再东京区域有一个 s3 存储桶开启了复制功能并配置了复制目标为首尔的存储桶，上传到东京区域的对象就可以自动复制到首尔的存储桶中

![50.png](https://s2.loli.net/2022/11/01/hwi2SEToDNuVtmM.png)

+ 跨 Amazon S3 存储桶**自动**以**异步方式**复制对象

+ 可在**不同 AWS 区域**或在**相同区域中**自动复制存储桶中的对象

+ 复制目标存储桶可以是和源存储桶**相同 AWS 账户**或**不同账户**拥有

+ 跨区域复制一般用于满足合规性要求、最大限度减少延迟、灾难恢复等

    1. 合规性要求：备份到 S3 的数据需要存储在不同区域，且对区域之间的距离有要求，这样就要将备份数据分别存储在不同的 AWS 区域的存储桶，而 S3 默认是跨多个可用区存储数据，这样就满足不了合规性的要求
    2. 最大限度减少延迟：如果需要 S3 服务的客户，处于两个地理位置，那么使用跨区域复制功能可以在**地理位置与客户较近的 AWS 区域**维护对象副本，从而最大限度缩短访问对象时的延迟，比如访问 S3 的客户在东京和首尔区域，我们的存储桶在东京区域，那么可以使用跨区域复制的功能配置复制目的区域为首尔，这样东京和首尔两地的客户都可以就近访问存储桶中的对象
    3. 灾难恢复：如果配置了跨区域复制功能，从东京复制到收到区域的话，当东京区域故障，东京的 S3 存储桶的对象可能就无法访问，在这个时候，您的存储桶中的对象还是可以在首尔区域访问的到，从而为对象提供了更高的持久性

+ 源存储桶和目标存储桶**必须已开启版本控制功能**

+ 复制功能不会复制在向存储桶添加复制配置**之前**存在的对象

    A 桶中有 girl-1.jpg,girl-2.jpg

    B 桶此时开启复制功能，不会同步 girl-1.jpg,girl-2.jpg

    当 A 桶中新增 `girl-3.jpg` 时，这时 B 桶才会同步 `girl-3.jpg`

S3 ---> 管理 ---> 复制规则

![image-20221101200835581.png](https://s2.loli.net/2022/11/01/3b6aiGJoPMu9kVX.png)

测试：

![sync-cp.gif](https://s2.loli.net/2022/11/01/E7oWmPIsvcQFq2U.gif)

### S3 预签名 URL

#### CLI 方式

将 mys3buckct-111 中的 utils.txt 设置为公开 `2` 分钟时间

1. 打开 `CloudShell Console`
2. 使用下面命令
    + --expires-in （整数）预签名 URL 过期前的秒数。默认值为 3600 秒。最大值为 604800 秒。

```bash
aws s3 presign s3://mys3buckct-111/utils.txt \
    --expires-in 120
```

## FSx

### Windwos 挂载 FSx

## EFS（Elastic File System）

### Linux 挂载 EFS

1. 创建安全组

    | name | port |
    | ---- | ---- |
    | ssh  | 22   |
    | efs  | 2049 |

2. 创建 EFS

3. 挂载 efs

    ```bash
    #!/bin/bash
    sudo su
    yum update -y
    yum install -y amazon-efs-utils
    mkdir efs
    mount -t efs -o tls fs-0e43a0c90d4ee7573:/ efs		## 挂载命令
    df -h
    cd efs
    mkdir aws
    cd aws
    echo "hello world" >> aws.txt
    ```

    返回结果

    ```bash
    [root@ip-172-31-14-65 ~]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    devtmpfs        474M     0  474M   0% /dev
    tmpfs           483M     0  483M   0% /dev/shm
    tmpfs           483M  472K  482M   1% /run
    tmpfs           483M     0  483M   0% /sys/fs/cgroup
    /dev/xvda1      8.0G  1.8G  6.2G  23% /
    127.0.0.1:/     8.0E     0  8.0E   0% /efs		### 挂载后的
    tmpfs            97M     0   97M   0% /run/user/1000
    
    ```

## RDS

### RDS 只读副本 - 轻松实现弹性扩展

不在使用单一的数据库同时处理**读取**和**写入**请求

可以创建数据库的只读副本

1. 数据中心服务器或使用 EC2 安装的 `mysql`  等数据库
    + 自行配置创建数据库只读副本
2. RDS 托管的数据库
    + 配置只读副本会更简单
    + 建议与原实例相同的类型和大小

![image.png](https://s2.loli.net/2022/11/02/1NYincDuZIeOo3M.png)

+ 只读副本更适用于**读取密集型数据库**工作负载的需求
+ 创建只读副本，**必须在源数据库实例上启用自动备份**，并将备份保留期设置为一个非零值
    + 选中数据库 ---> 修改 ---> 备份
+ 一个数据库实例可以最多创建 `5` 个只读副本，Aurora 最多 `15` 个
+ 可以将只读副本提升为独立的数据库实例使用
+ 只读副本适用于：
    1. MariaDB
    2. MySQL
    3. Oracle
    4. PostgreSQL
    5. Aurora

| 数据库名         | 终端节点                                             | 用户名及密码       |
| ---------------- | ---------------------------------------------------- | ------------------ |
| xsj-jp  主数据库 | xsj-jp.cnvjot9ubay1.ap-northeast-1.rds.amazonaws.com | admin    xsj1234.. |
| xsj-us 只读副本  | xsj-us.c6sngntbni6t.us-east-1.rds.amazonaws.com      | admin    xsj1234.. |

1. 创建数据库 `xsj-jp`

    注意开放公有访问权限

2. 创建只读副本 `xsj-us`

    操作 ---> 创建只读副本

    注意只读副本也要开启公有权限

3. 测试连通性以及连接 `xsj-jp`

    ```bash
    nc -zv xsj-jp.cnvjot9ubay1.ap-northeast-1.rds.amazonaws.com 3306
    ```

    ![51.png](https://s2.loli.net/2022/11/02/T8ol9vyd4AtHB2k.png)

    ```bash
    mysql -h xsj-jp.cnvjot9ubay1.ap-northeast-1.rds.amazonaws.com -P 3306 -u admin -p
    ```

    ![52.png](https://s2.loli.net/2022/11/02/2BKGObLtmrwHjP8.png)

4. 导入一个新的数据库 `larabbs` ，测试复制功能

    ![53.png](https://s2.loli.net/2022/11/02/HQinhDPpTogClbW.png)

5. 测试连通性以及连接 `xsj-us`

    ```bash
    nc -zv xsj-us.c6sngntbni6t.us-east-1.rds.amazonaws.com 3306
    ```

    ![51.png](https://s2.loli.net/2022/11/02/T8ol9vyd4AtHB2k.png)

    ```bash
    mysql -h xsj-us.c6sngntbni6t.us-east-1.rds.amazonaws.com -P 3306 -u admin -p
    ```

    ![52.png](https://s2.loli.net/2022/11/02/2BKGObLtmrwHjP8.png)

5. 查看 `larabbs` 数据库是否同步过来

    ![54.png](https://s2.loli.net/2022/11/02/iSHypC2ZV3q48WU.png)

6. `xusj-us` 只读副本，测试是否有写入权限

    发现是 `--read-only option` 只读模式

    ![55.png](https://s2.loli.net/2022/11/02/2VnKEuHMobtsklA.png)

8. 查看只读副本 `xsj-us` 的**副本滞后**指标

    显示只读副本当前时间戳与主数据库原始时间戳的差异，差异过大表示主数据库数据未能及时复制到只读副本中，只读副本数据较旧时需要及时排查问题

    实际使用环境中，需要对 `副本滞后` 指标做监控

#### 提升只读副本 --- 提升为主数据库

选中数据库 ---> 操作 ---> 提升

1. 条件：
    + 建议停止主数据库的所有事务并等待只读副本滞后为零
    + 提升过程是不可撤销的

2. 在 `xusj-us` 中创建数据库 `rdp_xushengjin` 测试是否有写入权限

    经过测试可以创建 `rdp_xushengjin` 数据库，有写入权限

![56.png](https://s2.loli.net/2022/11/02/FoLOsVRIDkP32jC.png)

### DynamoDB

+ NoSQL，完全托管的，无服务器的，最高支持每秒 1 百万个请求

+ 不需要提前预置存储空间，项目最大上限为 400 KB

    如果需要存储更大的对象到 DynamoDB，可以把对象存储到 S3 中，再将该对象的引用存储在 DynamoDB 中

+ 读写容量模式

    1. 按需：为每一次读写请求付费，开发环境中使用或表或应用程序负载无法预测时
    2. 预置（指定 WCU，WRU，AutoScaling）

+ 支持 CRUD，增加、检索、更新、删除

+ 支持最终一致性读取以及强一致性读取

+ 支持跨多个表的事务，因此是支持 **ACID** 的

+ 支持按需备份/还原，以及时间点恢复

+ DynamoDB 集成 IAM，使用 IAM 来控制访问

#### 组件

1. 表（必须要有主键）
2. 项目（行或记录）
3. 属性（列或字段）

#### DynamoDB 架构解决方案 --- 索引 S3 的对象

![57.png](https://s2.loli.net/2022/11/02/8gyfmZGRTFKrHC1.png)





### DAX

1. DAX 是 DynamoDB 的缓存服务
2. 它是 DynamoDB 的无缝缓存，不需要改写您的应用程序
3. 所有的写入操作都将通过 DAX 到 DynamoDB
4. 缓存的读取和查询，DAX 可提供微秒的延迟
5. 解决 HOT KEY 问题，通过缓存热点项目，使对其的访问不会访问 DynamoDB
6. 默认 DAX 的数据会被缓存 `5` 分钟
7. 集群中最多可配置 10 个节点
8. 可配置为多可用区 （AWS 建议在生产环境中至少配置 3 个节点）
9. KMS 静态加密、部署在 VPC、以及使用 IAM、以及 Cloudtrail 等安全服务控制
    访问和审计

![58.png](https://s2.loli.net/2022/11/02/5lX3Cx6DMEbzZvy.png)

#### 如何选择 DAX 和 ElasticCache

如果您的需要是需要客户直接访问 DynamoDB，那么选择 DAX，DAX 会缓存表中的项目，查询缓存，查询和扫描操作的结果缓存

如果您客户端检索的数据是需要花费一些时间进行大量的计算，然后又希望对这些计算的结果进行缓存的话，那么可以使用 ElasticCache，存储计算后聚合的结果，并确保后续客户端在访问 DynamoDB 前访问 ElasticCache 获取计算后的结果

![59.png](https://s2.loli.net/2022/11/02/NV4mMjinZP32EuR.png)

## CloudFront

### 源站 Origins

1. S3
    + 一般是静态文件
    + 访问身份 `OAI`，限制用户直接对 S3 存储桶访问，只通过 CloudFront 访问您的文件
    + 可以通过 CloudFront 作为入口上传内容至 S3，S3 的传输加速功能
2. S3 网站
    + 需要在 S3 存储桶开启了静态网站托管配置
3. 自定义源站
    + ALB
    + EC2
    + API 网关
    + 任何 HTTP 后端，包括在本地的 HTTP 服务器
4. 源组
    + 高可用的场景
    + 指定一个主源一个备源
    + 如果主源出现问题，会自动故障转移至备源

### 使用 S3 作为源站的架构

OAI 

| 名称           | 访问结果   |
| -------------- | ---------- |
| S3 DNS         | 不允许访问 |
| CloudFront DNS | 允许访问   |



![60.png](https://s2.loli.net/2022/11/02/bnsz28BAe3HOKCE.png)

### 使用 EC2 作为源站的架构

![61.png](https://s2.loli.net/2022/11/02/qd6oaDQ4K7Lf2Tb.png)

### CloudFront VS S3 跨区域复制

CloudFront

+ 将内容缓存在全球各地的边缘节点
+ 内容会缓存在边缘节点上，直到 `TTL` 到期，失效后，边缘节点会重新访问源站进行回源
+ 适合于将静态内容分发到全球让全球的用户 `低延迟`、`快速` 的访问

S3 跨区域复制功能

+ 需要为复制的内容到哪个区域，都要单独的配置复制
+ 区域之间的复制近似实时
+ 不支持双向复制
+ 适用于您的 `动态内容` 需要在 AWS 不同区域中以低延迟的方式访问的场景

### 地理限制

+ 限制类型
    1. 无限制
    2. 允许列表（白名单）
    3. 阻止列表（黑名单）

+ 使用第三方 `GeoP` 数据库确定您的用户的位置
+ 阻止非用户国家/地区的恶意攻击或版权问题

### CloudFront 签名 URL 和签名 Cookie 提供私有内容

+ 通过互联网分发 **`付费内容`** 给在世界各地的付费用户
+ 创建签名 URL 或签名 Cookie 时，可以附加策略
    1. 指定 URL/内容的过期时间
    2. 指定能访问内容的用户的 IP 地址或 IP 地址范围
    3. 受信任签名用户 --- 即哪个用户可以创建签名 URL 或签名 Cookie
+ 确定内容的有效期是多久？
    1. 共享内容：电影/音乐，可以配置在很短时间内有效（几分钟 - 几小时不等）
    2. 私有内容：比如向员工、投资者分发一些私有内容，可以配置有效时间较长的签名 URL （可能数年）

#### 签名 URL VS 签名 Cookie

签名 URL

+ 适用于对单个文件的访问，比如应用程序的安装程序下载，必须为每一个文件创建一个签名的 URL

签名 Cookie

+ 适用于对多个文件的访问，一个签名 Cookie 可以对应多个文件

### CloudFront 签名 URL 工作流

![64.png](https://s2.loli.net/2022/11/03/Jsc47geEMmOVjuw.png)



### CloudFront 签名 URL VS S3 预签名 URL

+ CloudFront
    + 提供给用户允许访问的路径，与源站无关，隔离了源站，而 S3 预签名 URL 只能用于 S3
    + 签名使用账户级密钥对，只有 root 用户可以管理
    + 可以配置过期时间，及通过 IP、路径、日期进行限制
    + CloudFront 缓存的特性 --- 两个用户访问同一个签名 URL 时，第二个用户直接可以在边缘节点获取而不用去源站进行获取

![62.png](https://s2.loli.net/2022/11/03/v8sWT72HBCDKmo1.png)

+ S3 预签名 URL
    + 以预签名 URL 的创建者发出请求
    + 指定 IAM 凭证，访问签名 URL 的用户与这个 IAM 凭证有相同的权限
    + 有限的生命周期

![63.png](https://s2.loli.net/2022/11/03/NxGSEtZV8Y1nTR9.png)

### Lambda@Edge

+ 使用 CloudFront 部署了 CDN

+ 你想在在您的全球边缘节点运行一个 Lambda 函数

+ 实现当请求到达您的应用前 `过滤` 一些请求

+ 可以使用 Lambda@Edge

    + 将 Lambda 函数部署在 CloudFront 分配的边缘节点上
    + 构建更具响应性的应用程序
    + 不需要自行管理服务器，它是无服务器架构，全球部署
    + 根据需要自定义 CDN 的内容
    + 只为使用的资源付费

+ 可直接对查看器进行响应，无需将请求发送到源站

    比如创建一个 Lambda 函数并部署到边缘节点，与查看器请求事件关联之后，如果有查看器请求过来，Lambda 可直接对其进行响应，而无需将请求发送到源站，向用户发送错误，或者 403 无权限等等这些情况，这样可以减少源站的请求量，减少资源

+ 没有缓存

#### Lambda@Edage 与 CloudFront 相关联时，CloudFront 在 CloudFront 边缘站点中捕获请求的和响应请求

当发生以下四种 CloudFront 时间时，可以执行 Lambda 函数：

1. 在 CloudFront 收到查看器的请求时，执行 Lambda 函数（查看器请求）
2. 在 CloudFront 将请求转发到源站之前，执行 Lambda 函数捕获请求和响应（源请求）
3. 在 CloudFront 收到来自源站的响应时，执行 Lambda 函数捕获请求和响应（源响应）
4. 在 CloudFront 将响应返回到查看器之前，执行 Lambda 函数捕获请求和响应（查看器响应）

![65.png](https://s2.loli.net/2022/11/03/FU5vCuTtPhfJclD.png)

#### Labmda@Edge 认证和授权



## ElasticCache

内存缓存服务 

+ 用来托管 Redis 或者 MemeCached
+ 缓存（Caches） 是内存数据库，具有非常高的性能和低延迟
+ 减少读取密集型工作负载的数据库负载
    + 比如将任何查询非常频繁的或者非常昂贵的计算可以缓存到 ElasticCahce，这样的话数据库本身可以收到很少的查询，减轻数据库负载。
+ ElasticCache 还可以帮助使您应用程序 `无状态化`
+ ElastiCache 是一个托管服务，AWS 负责操作系统级的维护，打补丁，优化配置，监控底层硬件，故障恢复和备份
    + 客户负责使用层面的个性化配置，安全组放行对 ElasticCache 的访问等等
+ 使用 ElastiCache 需要您对您的应用程序的代码进行对应修改和调整

### 常见的使用 ElasticCache 架构 - DB 缓存

1. 当应用程序请求数据时



## VPC

+ CIDR: Classless Inter-Domain Routing
    1. 创建 VPC 时，必须指定 IPv4 CIDR 块，允许的块大小介于 /16 和 /28 之间
    2. 很多配置会使用 CIRD 块，比如 VPC、子网、路由表、安全组
+ 私有 IP
    1. A类：10.0.0.0~10.255.255.255	大型网络
    2. B类：172.10.0.0~172.31.255.255 **AWS 默认**
    3. C类：192.168.0.0~192.168.255.255 
+ 公有 IP

### NAT 网关 和 NAT 实例

NAT 实例

+ 需要部署在公有子网

+ 添加 0.0.0.0/0 路由将其它流量至您的 NAT 实例

    Private Subnet EC2 ---> NAT 实例 ---> Public Subnet ---> Internet Gateway

+ 不是高可用的，需要您自行处理故障转移

+ 带宽受限、便宜

NAT 网关 --- 建议

+ 带宽自动扩展，网关每小时费率及数据处理费率
+ 在每个可用区是高度可用
+ 在每个可用区中创建一个 NAT 网关可确保架构不依赖于可用去
+ 需要绑定一个弹性 IP 地址，访问的外部服务看到的来源清求是 NAT 网关的弹性 IP 地址

### Network ACL

+ 在子网级别定义的无状态的防火墙，用来控制子网进出的流量
+ 无状态，对于允许出站请求的返回的流量，也必须在规则中明确允许，反之亦然
+ 支持添加允许和拒绝规则，可以非常快速的配置 NACL 拒绝某个 IP 地址的流量

### 安全组

+ 运行在实例级别，只能添加允许规则，无法添加拒绝规则
+ 有状态，如果从实例发送一个请求，都将允许该请求的响应流量流入
+ 可以在规则中引用统一区域的其它安全组

### VPC 流日志

+ 可以捕获有关传入和传出 VPC 中网络接口的 IP 流量的信息
+ 可以发布到 Cloud Watch Logs 或 S3
+ 可以为 VPC、子网、ENI 创建流日志

### 流日志

+ 通过堡垒机登录私有子网的实例

+ 需要自己配置和管理堡垒机，并自行负责堡垒机的故障迁移和安全性

+ 可以使用 SSM 的会话管理服务登录管理实例

    需要在实例上安装 SSM 代理

### VPC 终端节点

> 东京区域的一台 EC2，如果要访问 S3 存储桶，那么通常会通过 Internt 网关 ---> Internet ---> S3
>
> 但是有终端节点后，可以通过终端节点 ---> S3

1. VPC 终端节点使您能够将 VPC 通过 AWS 的私有网络连接到支持的 AWS 服务，而不需要通过 internet

2. 终端节点是虚拟设备，他们是水平扩展、荣誉和高度可用的 VPC 组件

3. 访问 AWS 服务无需 Internet 网关、NAT 网关、NAT 实例

4. 网关终端节点只支持 S3 和 DynamoDb

5. 排查终端节点问题

    1. DNS 解析配置
    2. 路由表中是否有到网关节点的路由

6. 需要更新路由表，在其中创建到 AWS 服务的路由

7. 网关终端节点，是在 VPC 级别定义的

    ![image-20221107165630428](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221107165630428.png)

8. 必须在 VPC 中启用 DNS 解析

9. **无法将终端节点连接扩展到 VPC 之外**

    比如创建了对等连接，对方是不能访问到你的终端节点服务的

![image-20221107164534642](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221107164534642.png)

#### VPC 终端节点 --- 网关终端节点

#### VPC 终端节点 --- 接口终端节点

1. 提供了一个弹性网络接口 ENI，具有来自所属子网 IP 地址范围的私有 IP 地址

#### 网关负载均衡器终端节点



### AWS Site-to-Site VPN 的路由配置

![image-20221108104123110](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221108104123110.png)

左侧为数据中心、右侧为 AWS

网络 CIDR 块为非重叠的

私有子网的网络需要访问本地数据中心的资源

1. 静态路由
    + 在公司数据中心的路由表，将去往 VPC 私有子网 10.0.0.0/24 的通信指向客户网关的路由
    + 在 AWS 端，将去往公司数据中心 10.2.0.0/20 的通信指向虚拟专用网关 VGW
    + 之后网络发生变化，还需要手动配置
2. 动态路由（BGP）
    + 边际网关协议，允许你的网络之间自动交换彼此网络路由信息
    + 需要在客户网关以及虚拟专用网关配置 ASN(自治系统编号 65000)
    + 不需要手动配置路由表，BGP 自动更新

### AWS Site-to-Site VPN 与 Internet 访问

![image-20221108105758273](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221108105758273.png)

1. 本地公司数据中心的服务器不能通过客户网关，然后通过 VGW，在通过 NAT 网关 和 Internet 网关访问 Internet

![image-20221108110332130](C:\Users\XuShengjin\Desktop\我的文档\typora\我的坚果云\上海神州数码\AWS.assets\image-20221108110332130.png)

2. 本地公司数据中心的服务器可以通过客户网关，然后通过 VGW，在通过 NAT 实例和 Internet 网关访问 Interne
    + 因为使用 NAT 实例

### CloudHub

