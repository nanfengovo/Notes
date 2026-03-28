# 搭建使用VSCode 进行dotnet core 开发的环境和docker环境
## 下载4个开发C# 的插件
* c# 
 ![[csharp 插件.png]]*NuGet Gallery
 ![[nuget 插件.png]]
 * vscode-solution-explorer
 ![[vscode-solution-explorer.png]]
 * C# Extensions
 ![[CsharpExtensions.png]]*
# git 管理代码
创建远程仓库后
git init
*  git init

关联远程地址
> # 将 URL> 替换为你远程仓库的地址（HTTPS 或 SSH）
git remote add origin https://github.com/用户名/仓库名.git
## 
> git remote add origin https://github.com/nanfengovo/dotnet-core-6-code.git

## 检查关联的状态
> git remote -v

```
feng@nanfengdeMacBook-Pro dotnetCore % git remote add origin https://github.com/nanfengovo/dotnet-core-6-code.git

feng@nanfengdeMacBook-Pro dotnetCore % git remote -v

origin https://github.com/nanfengovo/dotnet-core-6-code.git (fetch)

origin https://github.com/nanfengovo/dotnet-core-6-code.git (push)

feng@nanfengdeMacBook-Pro dotnetCore %
```

# 编写docker文件来进行容器化开发
## 编写Dockerfile 和devcontainer.json
### Dockerfile
``` yaml
FROM mcr.microsoft.com/dotnet/sdk:6.0

  

# 1. 环境准备

ARG USERNAME=vscode

RUN groupadd --gid 1000 $USERNAME \

&& useradd --uid 1000 --gid 1000 -m $USERNAME \

&& apt-get update \

&& apt-get install -y sudo curl git procps \

&& echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME

  

# 2. 安装 Node.js

RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \

&& apt-get install -y nodejs build-essential

  

# 3. 预装工具 (注意：创建 npm 全局目录并赋权)

RUN mkdir -p /home/vscode/.npm-global \

&& chown -R vscode:vscode /home/vscode/.npm-global

  

USER vscode

  

RUN npm config set prefix /home/vscode/.npm-global

# 环境变量配置

ENV PATH="${PATH}:/home/vscode/.dotnet/tools:/home/vscode/.npm-global/bin"

  

# 明确工作目录

WORKDIR /workspaces

```

### devcontainer.json

```json
{

"name": "Full-Stack-Dev-Container",

"build": {

"dockerfile": "Dockerfile" // 确保 Dockerfile 在同一级或正确路径

},

"remoteUser": "vscode",

"runArgs": [

"--name", "full_stack_dev_workstation", // 容器名建议用下划线，不要有空格

"--network", "dev_infrastructure_net" // 确保该网络已通过 docker network create 创建

],

"customizations": {

"vscode": {

"extensions": [

"ms-dotnettools.csdevkit",

"ms-dotnettools.csharp",

"patcx.vscode-nuget-gallery",

"fernandoescolar.vscode-solution-explorer",

"anthropic.claude-code",

"vue.volar"

],

"settings": {

"terminal.integrated.shell.linux": "/bin/bash"

}

}

},

"mounts": [

// 挂载 SSH 密钥（拉取代码必备）

"source=${localEnv:HOME}/.ssh,target=/home/vscode/.ssh,type=bind,readonly",

// 挂载配置目录（包含 Claude, Gemini 等 token）

"source=${localEnv:HOME}/.config,target=/home/vscode/.config,type=bind",

"source=${localEnv:HOME}/.claude,target=/home/vscode/.claude,type=bind"

],

"containerEnv": {

"ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}",

"GOOGLE_API_KEY": "${localEnv:GOOGLE_API_KEY}"

},

// 建议在创建后自动更新一次 npm 工具

"postCreateCommand": "npm install -g @anthropic-ai/claude-code codex-cli"

}

```
## dockerfile 和devcontainer.json文件有什么区别？

``` markdown
在 VS Code 的远程开发模式下，它们分工明确：`Dockerfile` 负责让程序能跑起来，而 `devcontainer.json` 负责让你开发起来更舒服。

---

## 1. Dockerfile：定义“运行环境”

`Dockerfile` 是 Docker 的标准文件，它不依赖于任何编辑器。它的目标是构建一个自包含的镜像。

- **核心职责：** 安装操作系统、安装 .NET 6 运行时/SDK、拷贝源码、暴露端口、定义启动命令。
    
- **适用场景：** 无论是本地开发、CI/CD 流水线，还是生产环境服务器，用的都是这个文件。
    
- **内容示例：**
    
    Dockerfile
    
    ```
    FROM mcr.microsoft.com/dotnet/sdk:6.0
    WORKDIR /app
    RUN apt-get update && apt-get install -y git
    ```
    

---

## 2. devcontainer.json：定义“开发体验”

这是 VS Code 专有的配置文件（现在也逐渐被 JetBrains 等支持）。它告诉编辑器：“当我进入这个容器时，请帮我配置好我的开发工具”。

- **核心职责：**
    
    - **挂载目录：** 自动将你 Mac 上的代码目录映射到容器里。
        
    - **安装插件：** 自动在容器内安装 C# 扩展、GitLens 等。
        
    - **编辑器配置：** 自动设置容器内的 `settings.json`（比如自动保存、代码格式化）。
        
    - **权限与用户：** 指定是以 `root` 还是普通用户身份进入终端。
        
- **内容示例：**
    
    JSON
    
    ```
    {
      "name": "C# (.NET 6) Dev Environment",
      "build": { "dockerfile": "Dockerfile" }, // 引用上面的 Dockerfile
      "customizations": {
        "vscode": {
          "extensions": ["ms-dotnettools.csharp", "jchannon.csharpextensions"]
        }
      },
      "remoteUser": "vscode"
    }
    ```
    

---

## 3. 核心区别对比

|**特性**|**Dockerfile**|**devcontainer.json**|
|---|---|---|
|**本质**|基础设施即代码 (IaC)|编辑器/IDE 的环境配置|
|**通用性**|通用（Docker 命令行、Kubernetes 都能用）|专用（主要用于 VS Code / GitHub Codespaces）|
|**主要内容**|操作系统、SDK、环境变量、依赖库|插件列表、IDE 设置、端口转发、磁盘挂载|
|**最终产物**|Docker Image (镜像)|一个配置好的远程开发会话|
|**DevOps 意义**|确保“开发环境”与“生产环境”一致|确保“团队内所有成员”的开发工具链一致|
```

# 准备.gitignore和vscode中调试需要用到的文件
## .gitignore
``` json
# Build output

bin/

obj/

  

# User-specific files

*.user

*.rsuser

*.suo

  

# IDE state

.vs/

!.vscode/launch.json

!.vscode/tasks.json

!.vscode/extensions.json

  

# Test and coverage output

TestResults/

coverage/

*.coverage

*.coveragexml

  

# Logs

*.log

  

# OS files

.DS_Store

Thumbs.db
```
## 调试需要的
### launch.json
``` json
{

"version": "0.2.0",

"configurations": [

{

"name": ".NET Launch SecondDemo.API",

"type": "coreclr",

"request": "launch",

"preLaunchTask": "build SecondDemo.API",

"program": "${workspaceFolder}/Demo/SecondDemo.API/bin/Debug/net6.0/SecondDemo.API.dll",

"args": [],

"cwd": "${workspaceFolder}/Demo/SecondDemo.API",

"stopAtEntry": false,

"serverReadyAction": {

"action": "openExternally",

"pattern": "\\bNow listening on:\\s+(https?://\\S+)"

},

"env": {

"ASPNETCORE_ENVIRONMENT": "Development"

},

"envFile": "${workspaceFolder}/Demo/SecondDemo.API/.env",

"launchBrowser": {

"enabled": false

}

}

]

}
```
### tasks.json
```json
{

"version": "2.0.0",

"tasks": [

{

"label": "build SecondDemo.API",

"type": "process",

"command": "dotnet",

"args": [

"build",

"${workspaceFolder}/Demo/SecondDemo.API/SecondDemo.API.csproj"

],

"problemMatcher": "$msCompile",

"group": "build"

}

]

}
```
