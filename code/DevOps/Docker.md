# Docker 的基本概念
* 容器（Container）：轻量化的运行实例，包含应用代码、运行时环境和依赖库。基于镜像创建于其他容器隔离，共享主机操作系统内核（比虚拟机更高效）
* 镜像（Image）:只读模版，定义了容器的运行环境（如操作系统、软件配置等）。通过分层存储（layer）优化空间和构建速度
* 仓库（Registry）: 存储和分发镜像等平台如：Docker Hub(官方公共仓库)或私有仓库（如Harbor）
![[docker.png]]
# Docker 命令
## 拉取镜像
 > docker pull docker.io/library/nginx:latest

### 解析docker命令
 ```markdown
docker.io:拉取的仓库地址；这里指官方仓库，如果是官方的仓库可以省略
library: 命名空间，这个是 官方的可以不写
latest: 标签（版本号）不写默认是最新的
所以上面的命令可以简化为docker pull nginx
 ```

> docker pull --platform =xxxxx nginx
### 解析docker命令
```markdown
--platform = xxxx ：拉取特定的平台的（大部分情况下docker会自动选择适合本机的）
```


####  示例：下载一个开源工具n8n
> docker pull docker.n8n.io/n8nio/n8n
## 查看安装了哪些镜像
> docker imagesd

## 删除镜像
> docker rmi 镜像名字/ID

## 使用镜像创建并运行容器
> docker run
> docker run -d
> docker run -p 80:80
> 

### 命令解析
```markdown
-d:datached mode 分离模型 不会占当前的终端
-p 80:80 :前面是宿主机的端口，后面是docker容器运行的端口，将宿主机80 端口转发到容器的80端口处理
```
## 查看正在运行的容器
> docker ps
## 挂载卷
> docker run -v 宿主机的目录：容器内的目录

# Docker 实战
## 后台运行一个名为nginx 的容器端口使用本地的80，挂载到
``` yaml
services:
  nginx-gateway:
    image: nginx:alpine
    container_name: nginx-dev
    restart: always
    ports:
      - "80:80"     # 映射 HTTP
      - "443:443"   # 映射 HTTPS
    volumes:
      # 1. 配置文件挂载（最重要：修改这里，容器即生效）
      - ./nginx/conf.d:/etc/nginx/conf.d
      
      # 2. 静态资源挂载（把你的前端代码丢进这里的 html 文件夹即可）
      - ./nginx/html:/usr/share/nginx/html
      
      # 3. 日志挂载（方便你在 Mac 上直接查看日志，不用进容器）
      - ./nginx/logs:/var/log/nginx
    networks:
      - db-network
```