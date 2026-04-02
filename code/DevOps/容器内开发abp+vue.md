# 生成https证书
## 安装`dev-certs` 工具
> dotnet tool install --global dotnet-dev-certs

## 创建目录文件夹（保存https证书的位置）
> mkdir -p /home/vscode/.aspnet/https/

## 创建https证书
> dotnet dev-certs https --export-path /home/vscode/.aspnet/https/aspnetapp.pfx --password "aaaa2624434145"


# 创建容器配置文件夹
## 在根目录下新建.devcontainer
## 在.devcontainer下新建devcontainer.json
```json
{

"name": "RCS-TemplateV1-Dev-Container",

"build": {

"dockerfile": "Dockerfile" // 确保 Dockerfile 在同一级或正确路径

},

"remoteUser": "vscode",

"runArgs": [

"--name",

"rcs_templatev1_dev_workstation", // 容器名建议用下划线，不要有空格

"--network",

"dev_infrastructure_net" // 确保该网络已通过 docker network create 创建

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

"source=${localEnv:HOME}/.claude,target=/home/vscode/.claude,type=bind",

"source=${localEnv:HOME}/.codex,target=/home/vscode/.codex,type=bind",

"source=${localEnv:HOME}/.gemini,target=/home/vscode/.gemini,type=bind"

],

"containerEnv": {

"ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}",

"GOOGLE_API_KEY": "${localEnv:GOOGLE_API_KEY}",

"GEMINI_API_KEY": "${localEnv:ANTHROPIC_API_KEY}",

"OPENAI_API_KEY": "${localEnv:ANTHROPIC_API_KEY}"

},

// 建议在创建后自动更新一次 npm 工具

"postCreateCommand": "npm install -g @anthropic-ai/claude-code @openai/codex @google/gemini-cli opencode-ai"

}

```

## 在根目录下新建Dockerfile
```yaml
FROM mcr.microsoft.com/dotnet/sdk:8.0

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
RUN dotnet tool install -g Volo.Abp.Cli --version 8.3.4
# 明确工作目录
WORKDIR /workspaces
```
# 容器内的连接字符串
```
"Server=sqlserver-dev;Database=数据库名（一般和项目名一致）;User Id=sa;Password=abc,123;TrustServerCertificate=True"
```