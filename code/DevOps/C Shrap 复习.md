# 环境架构
dotnet/ (工作区根目录)
├── .devcontainer/
│   ├── Dockerfile          <-- 自定义镜像说明书
│   └── devcontainer.json   <-- VS Code 配置文件
└── projects/               <-- 存放各个子项目的文件夹
│   ├── MySolution.sln          <-- 解决方案（之后创建）
# 编写容器配置文件
## Dockerfile
```yaml
# 使用微软官方 .NET 8 SDK 镜像
FROM mcr.microsoft.com/devcontainers/dotnet:8.0

# 切换到 vscode 用户进行操作，避免 root 权限污染
USER vscode

# 安装 ABP CLI (锁定 8.3.0 稳定版)
RUN dotnet tool install -g Volo.Abp.Cli --version 8.3.0

# 将 .NET 工具路径永久加入环境变量
ENV PATH="${PATH}:/home/vscode/.dotnet/tools"

```
## devcontainer.json
```json
{
    "name": "dev_sdk",
    "build": {
        "dockerfile": "Dockerfile" // 指向同目录下的 Dockerfile
    },
    "runArgs": [
        "--name", "dev_sdk",
        "--network=dev_infrastructure_net" // 连接你的 SQL Server 网络
    ],
    "customizations": {
        "vscode": {
            "extensions": [
                "ms-dotnettools.csdevkit",
                "ms-dotnettools.csharp",
                "humao.rest-client"
            ],
            "settings": {
                "terminal.integrated.defaultProfile.linux": "bash"
            }
        }
    },
    "remoteUser": "vscode",
    "workspaceFolder": "/workspaces/dotnet"
}
```
# 开发demo
## 创建一个不使用顶级语句的控制台项目
```
# -n 指定项目名，-o 指定文件夹
# --use-program-main 是关键参数，它告诉 .NET 不要使用顶级语句
dotnet new console -n consoledemo -o consoledemo --use-program-main
```
## 创建abp 后端demo
> abp new ApiDemo -u none --version 8.3.4
### 修改数据库连接字符串
> "Server=sqlserver-dev;Database=ApiDemo;User Id=sa;Password=你的密码;TrustServerCertificate=True"

由于abp需要使用npm
### 修改Dockerfile
```yaml
FROM mcr.microsoft.com/dotnet/sdk:8.0

# 1. 创建用户并安装基础工具 (Root 权限)
ARG USERNAME=vscode
RUN groupadd --gid 1000 $USERNAME \
    && useradd --uid 1000 --gid 1000 -m $USERNAME \
    && apt-get update \
    && apt-get install -y sudo curl \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME

# 2. 安装 Node.js (Root 权限)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs

# 3. 切换用户并安装 ABP (vscode 用户权限)
USER vscode
RUN dotnet tool install -g Volo.Abp.Cli --version 8.3.4
ENV PATH="${PATH}:/home/vscode/.dotnet/tools"
```
**修改后：**

1. 按下 `Cmd + Shift + P`。
    
2. 输入 **"Dev Containers: Rebuild Container"**。
    
3. 等待容器重构完成后，NPM 就会永久存在于你的开发环境中了。

