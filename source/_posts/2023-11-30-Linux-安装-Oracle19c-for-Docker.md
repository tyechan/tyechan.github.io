---
title: Linux 安装 Oracle19c for Docker
date: 2023-11-30 11:11:21
tags:
- linux
- ubuntu
- docker
- oracle
- 环境配置
categories:
---

Linux 安装 Oracle 数据库比较繁琐耗时，为了更快的进入学习状态，采用 Docker 快速安装部署。  
为什么选择 [Oracle19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/index.html) ？一、19c 和 12c 差不太多；二、有现成的Docker Image可以使用。  
如需安装其他版本 Oracle 数据库，可以使用 Oracle 官方 [Dockerfile](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance#how-to-build-and-run) 自己编译安装。

参考资料：[Linux下安装docker以及docker安装Oracle19c的全部详细过程及各种问题解决](https://blog.csdn.net/suixinfeixiangfei/article/details/127364822)

<!--more-->

- [安装 Linux](#安装-linux)
  - [CentOS](#centos)
  - [Ubuntu](#ubuntu)
- [安装 Docker](#安装-docker)
  - [CentOS 通过 yum 安装](#centos-通过-yum-安装)
  - [Ubuntu 2204 Server 自带 Docker](#ubuntu-2204-server-自带-docker)
    - [方法一、使用 Docker apt 仓库](#方法一使用-docker-apt-仓库)
    - [方法二、使用 deb 文件安装](#方法二使用-deb-文件安装)
    - [方法三、使用脚本自动安装](#方法三使用脚本自动安装)
- [安装 Oracle19c](#安装-oracle19c)
  - [拉取镜像](#拉取镜像)
  - [安装容器](#安装容器)
  - [进入容器](#进入容器)
  - [创建数据库](#创建数据库)
  - [使用 A5M2 连接数据库](#使用-a5m2-连接数据库)


# 安装 Linux

## CentOS
[CentOS 7 将于2024年6月30日结束维护](https://www.centos.org/cl-vs-cs/)，届时将无法获得官方补丁安装支持和系统升级，推荐个人用户选择 Ubuntu。

## Ubuntu
本次使用 [Ubuntu 2204 Server](https://www.releases.ubuntu.com/jammy/)版本，被称为[目前最安全的 Ubuntu 版本](https://zhuanlan.zhihu.com/p/540742163)。

# 安装 Docker

## CentOS 通过 yum 安装
```shell
yum install -y docker
```

安装后查看 Docker 是否安装成功。

```shell
yum list installed |grep docker
```

## Ubuntu 2204 Server 自带 Docker
Ubuntu 2204 Server 安装引导中，可选安装 Docker 。  
若是未安装，可以通过以下方法安装 Docker 。
参考自[官方资料](https://docs.docker.com/engine/install/ubuntu/#installation-methods)。

### 方法一、使用 Docker apt 仓库
先配置仓库，之后通过该仓库安装或升级 Docker。

1. 配置 Docker apt 仓库
```shell
# 安装官方 GPG Key
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 添加 apt 仓库到 apt 源
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. 安装 Docker 软件包
    - 安装最新版本
    ```shell
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    - 安装指定版本

        1. 首先列出存储库中的可用版本
        ```shell
        # List the available versions:
        apt-cache madison docker-ce | awk '{ print $3 }'

        5:24.0.0-1~ubuntu.22.04~jammy
        5:23.0.6-1~ubuntu.22.04~jammy
        ...
        ```

        2. 然后选择所需的版本并安装
        ```shell
        VERSION_STRING=5:24.0.0-1~ubuntu.22.04~jammy
        sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
        ```

3. 检查是否安装成功  
通过运行 hello-world 映像来验证 Docker Engine 安装是否成功。
```shell
sudo docker run hello-world
```

### 方法二、使用 deb 文件安装
如果方法一无法安装，可以通过官方发布的 deb 文件进行安装或升级。

1. 前往：<https://download.docker.com/linux/ubuntu/dists/>。
2. 选择所需版本进行下载。
3. 进入 `pool/stable/` 选择软件架构，一般家用电脑使用amd64。
4. 下载以下 deb 文件。
    - `containerd.io_<version>_<arch>.deb`
    - `docker-ce_<version>_<arch>.deb`
    - `docker-ce-cli_<version>_<arch>.deb`
    - `docker-buildx-plugin_<version>_<arch>.deb`
    - `docker-compose-plugin_<version>_<arch>.deb`
5. 安装.deb 包。将下面示例中的路径修改为下载 deb 文件的位置。
```shell
sudo dpkg -i ./containerd.io_<version>_<arch>.deb \
  ./docker-ce_<version>_<arch>.deb \
  ./docker-ce-cli_<version>_<arch>.deb \
  ./docker-buildx-plugin_<version>_<arch>.deb \
  ./docker-compose-plugin_<version>_<arch>.deb
```
6. 使用 hello-world 镜像，检查是否安装成功。
```shell
sudo service docker start
sudo docker run hello-world
```

### 方法三、使用脚本自动安装
`curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`

# 安装 Oracle19c
## 拉取镜像
```shell
docker pull registry.cn-hangzhou.aliyuncs.com/zhuyijun/oracle:19c
```
该步骤可以省略， `docker run` 可以自动拉取镜像

## 安装容器
```shell
docker run -d -it --name orcl19c_03 -p 1521:1521 -p 5500:5500 -e ORACLE_SID=orcl -e ORACLE_PDB=orclPDB1 -e ORACLE_PWD=123456 -e ORACLE_EDITION=standard -e ORACLE_CHARACTERSET=AL32UTF8 -v /home/mydata/oracle/oracleData:/opt/myData/oracle/oracleData registry.cn-hangzhou.aliyuncs.com/zhuyijun/oracle:19c
```
上面的命令中  
- `ORACLE_SID=orcl` 是CDB数据库的服务名
- `ORACLE_PDB=orclPDB1` 是PDB数据库的服务名
- `ORACLE_PWD=123456` 是管理员密码

## 进入容器
1. 查看容器 ID
```shell
docker ps -a
```
`-a` 显示全部，包括未启动的容器。

2. 进入容器
```shell
docker exec -it [1.容器id] bash
```

## 创建数据库
1. 连接 Oracle
```shell
sqlplus system/123456@127.0.0.1/orclPDB1
```
1. 创建用户
```sql
create user test1 identified by 123456;
```
1. 赋予权限
```sql
grant connect, resource to test1;
```
1. 设置表领域配额上限
```sql
alter user test1 quota unlimited on users;
```
1. 设置密码不过期
```sql
alter profile default limit password_life_time unlimited;
```

## 使用 A5M2 连接数据库
1. 连接方式选择直接连接
2. 填写主机名，端口
3. 服务名：orclPDB1
4. 用户ID：test1
5. 密码：123456
6. 勾上保存密码
7. 点击测试连接
8. 测试连接通过，点 OK 保存

> 施工中...
