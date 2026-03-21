# 关于开发语言、开发环境和开发工具
* C#
* Vue
* React
* Python
* Go
* Docker
* VsCode(配置隔离)
* Rider
* Git (宿主机安装)
* .NET SDK(Docker中)
* NodeJS(Docker中)
* SqlServer(Docker中)
* MySql(Docker中)
* PostgresSql(Docker中)
* Sqlite(Docker中)
# 搭建开发环境
## 安装基础包管理器
### Homebrew 安装
> /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
### 将 Homebrew 加入到你的 PATH 中
> echo >> /Users/feng/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> /Users/feng/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv zsh)"

### 验证是否成功
> brew --version
### 安装git
>brew install git

### 验证git 是否安装完成
> git --version

### git 基础配置
>feng@nanfengdeMacBook-Pro ~ % git config --global user.name "nanfeng"

feng@nanfengdeMacBook-Pro ~ % git config --global user.email "nanfengovo@gmail.com"
### 查看配置
> git config --list
### 配置SSHKey :用于Github/GitLab 免密登陆
>ssh-keygen -t ed25519 -C "nanfengovo@gmail.com"

### 查看
>cat ~/.ssh/id_ed25519.pub

复制到：

- GitHub → Settings → SSH Keys

#### 使用git将本地文件夹推送到github
```
feng@nanfengdeMacBook-Pro ~ % ls

Desktop Downloads Movies Pictures

Documents Library Music Public

feng@nanfengdeMacBook-Pro ~ % cd Documents

feng@nanfengdeMacBook-Pro Documents % ls

Notes

feng@nanfengdeMacBook-Pro Documents % cd Notes

feng@nanfengdeMacBook-Pro Notes % git init

Initialized empty Git repository in /Users/feng/Documents/Notes/.git/

feng@nanfengdeMacBook-Pro Notes %
```
##### 添加文件并提交
###### 添加文件到暂存区
> git add .
###### 提交更改
> git commit -m "Inital commit"

###### 创建GitHub仓库
- 登录到 GitHub 网站。
    
- 在右上角点击 **New repository**。
    
- 给仓库起个名字（比如 `Notes`），并选择是否公开/私有，然后点击 **Create repository**。
###### 将本地仓库与GitHub仓库关联
- 复制 GitHub 仓库的远程地址（比如：`https://github.com/your-username/Notes.git`）。
    
- 在终端中执行以下命令，将本地仓库与远程 GitHub 仓库关联：
> git remote add origin https://github.com/your-username/Notes.git
#### 推送到GitHub
> git push -u -origin main

输入用户名和密码这里输入密码无效GitHub 现在不支持通过用户名和密码进行身份验证，而是要求使用 **Personal Access Token (PAT)** 来进行身份验证。

避免每次都提交：
> git config --global credential.helper cache


## Docker
### 验证Docker的可用性
> docker --version
> docker compose version

``` docker compose 是什么
`docker compose version` 用来查看 **Docker Compose 的版本**

---

# Docker Compose 是什么

**Docker Compose = 管理多个容器的工具**

你可以把它理解为：

|工具|作用|
|---|---|
|docker run|启动一个容器|
|docker compose|一次启动一整套服务|

---

# 举个实际例子

你要跑一个系统：

- 后端（.NET）
    
- 前端（Vue）
    
- 数据库（MySQL）
    

如果不用 compose，你要写 3 条 `docker run`，还要自己处理网络，很麻烦。

用 compose：

version: "3.9"  
services:  
  api:  
    image: my-api  
  web:  
    image: my-web  
  db:  
    image: mysql:8

然后只需要一条命令：

docker compose up

```
#### 跑一个实例来看看是否真可用
> docker run hello-world
### 以配置一个数据库为例来学习docker
```
docker run -d \
  --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -p 3306:3306 \
  mysql:8
```
####  验证目前在run的容器
> docker ps
###  Docker Compose(容器化编排)

新建一个 `docker-compose.yml`
``` yaml
version: "3.9"
services:
  mysql:
    image: mysql:8
    container_name: mysql-dev
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    ports:
      - "3306:3306"

```

- `version: "3.9"`：指定 Docker Compose 配置文件版本。
    
- `image: mysql:8`：指定使用 MySQL 8 版本的镜像。
    
- `restart: always`：指定容器停止后会自动重启。
    
- `docker compose up -d`：启动 `docker-compose.yml` 文件中定义的服务，并在后台运行。
#### 拉取sqlserver 和mysql的镜像
```yaml
version: "3.9"

services:
  mysql:
    image: mysql:8
    container_name: mysql-dev
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: abc,123  # 设置 MySQL root 用户的密码
    ports:
      - "3306:3306"                     # 映射容器的3306端口到宿主机
    networks:
      - db-network

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: sqlserver-dev
    restart: always
    environment:
      SA_PASSWORD: "abc,123"  # 设置 SQL Server 的系统管理员密码
      ACCEPT_EULA: "Y"                    # 同意 SQL Server 的 EULA
    ports:
      - "1433:1433"                       # 映射容器的1433端口到宿主机
    networks:
      - db-network

networks:
  db-network:
    driver: bridge

```

#### 创建yml文件
> touch docker-compose.yml

复制粘贴上面的文本到文件中
#### 启动MySQL 和 SQL Server 容器：
> docker compose up -d

#### 拉取和启动遇到的问题
```
feng@nanfengdeMacBook-Pro docker compose % docker compose up -d
WARN[0000] /users/feng/Documents/Code/demo/docker compose/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] up 19/21
 ✔ Image m... Pulled                                                    177.8s
[+] up 19/21. Pulled                                                    416.4s
 ✔ Image m... Pulled                                                    177.8s
[+] up 21/21. Pulled                                                    416.4s
 ✔ Image m... Pulled                                                    177.8s
 ✔ Image m... Pulled                                                    416.4s ✔ Network... Created                                                   0.0sed ✔ Contain... Started                                                   0.5se  ✔ Contain... Started                                                   0.5sed ! sqlserver  The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested 0.0s
feng@nanfen
```
这里的警告（Warning）和最后那个 `!` 错误是由于你的 **MacBook Pro 是 M 系列芯片（Apple Silicon）** 导致的。

### 1. 核心问题：平台不匹配 (Platform Mismatch)

错误提示 `linux/amd64` 不匹配 `linux/arm64/v8`。 这是因为你的电脑是 **ARM 架构**，而你拉取的 `sqlserver` 镜像是为 **Intel/AMD 架构**设计的。
#### 修改yml文件来解决
``` yaml
services:
  mysql:
    image: mysql:8
    container_name: mysql-dev
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: abc,123  # 设置 MySQL root 用户的密码
    ports:
      - "3306:3306"                     # 映射容器的3306端口到宿主机
    networks:
      - db-network

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    platform: linux/amd64 # <--- 加上这一行
    container_name: sqlserver-dev
    restart: always
    environment:
      SA_PASSWORD: "abc,123"  # 设置 SQL Server 的系统管理员密码
      ACCEPT_EULA: "Y"                    # 同意 SQL Server 的 EULA
    ports:
      - "1433:1433"                       # 映射容器的1433端口到宿主机
    networks:
      - db-network

networks:
  db-network:
    driver: bridge
```

重新启动
> docker compose up -d

#### sqlserver密码太简单但是我就想用简单的怎么办
修改yml文件
```yaml
services:

mysql:

image: mysql:8

container_name: mysql-dev

restart: always

environment:

MYSQL_ROOT_PASSWORD: abc,123 # 设置 MySQL root 用户的密码

ports:

- "3306:3306" # 映射容器的3306端口到宿主机

networks:

- db-network

  

sqlserver:

image: mcr.microsoft.com/mssql/server:2022-latest

platform: linux/amd64 # <--- 加上这一行

container_name: sqlserver-dev

restart: always

environment:

SA_PASSWORD: "abc,123" # 设置 SQL Server 的系统管理员密码

MSSQL_ENFORCE_PASSWORD_POLICY: "OFF" # <--- 关键：禁用密码策略

ACCEPT_EULA: "Y" # 同意 SQL Server 的 EULA

ports:

- "1433:1433" # 映射容器的1433端口到宿主机

networks:

- db-network

  

networks:

db-network:

driver: bridge
```
停掉服务重新启动
```
docker compose down
docker compose up -d
```
### 在Docker中拉取.NET8和NodeJs
####  拉.NET8 sdk (开发用)
> docker pull mcr.microsoft.com/dotnet/sdk:8.0

##### 验证拉取的镜像
> docker run -it mcr.microsoft.com/dotnet/sdk:8.0 dotnet --version

#### 拉取运行时（生产环境）
> docker pull mcr.microsoft.com/dotnet/aspnet:8.0

#### 进入容器
> docker run -it mcr.microsoft.com/dotnet/sdk:8.0 /bin/bash

> dotnet new webapi

#### 配置开发容
```
docker run -it --name dotnet-dev \
  -p 5000:5000 \
  -v ~/Documents/demo:/app \
  -w /app \
  mcr.microsoft.com/dotnet/sdk:8.0
```
#### 拉NodeJs镜像





