# 生成https证书
## 安装`dev-certs` 工具
> dotnet tool install --global dotnet-dev-certs

## 创建目录文件夹（保存https证书的位置）
> mkdir -p /home/vscode/.aspnet/https/

## 创建https证书
> dotnet dev-certs https --export-path /home/vscode/.aspnet/https/aspnetapp.pfx --password "aaaa2624434145"

## 优化证书生成的相关的
> 现在的问题是每个新的项目都需要在容器中生成https证书，尝试将一个容器中生成的证书拿出来作为通用证书

### 在mac终端执行
```markdown 
# 创建 Mac 本地的证书存放目录
mkdir -p ~/.aspnet/https

# 找到你当前运行的容器 ID
docker ps

# 将证书拷贝出来（假设容器名为 my_dev_container）
docker cp my_dev_container:/home/vscode/.aspnet/https/aspnetapp.pfx ~/.aspnet/https/
```
### 在宿主机建立永久信任
```markdown
### 第二阶段：在 Mac 宿主机建立“永久信任”

这一步是为了消除浏览器里的 `ERR_CERT_AUTHORITY_INVALID` 报错。

1. **打开“钥匙串访问” (Keychain Access)：**
    
    - 按下 `Command + Space`，输入 `Keychain Access` 并回车。
        
2. **导入证书：**
    
    - 在左侧选择 **“登录” (Login)** 钥匙串。
        
    - 点击菜单栏的 **“文件” -> “导入项目”**。
        
    - 定位到 `~/.aspnet/https/aspnetapp.pfx`。
        
    - 输入你生成证书时设置的密码：`aaaa2624434145`。
        
3. **设置为始终信任：**
    
    - 在列表中找到刚导入的证书（通常名称为 **localhost**）。
        
    - **双击**该证书，展开 **“信任” (Trust)** 选项。
        
    - 将 **“使用此证书时” (When using this certificate)** 下拉框改为 **“始终信任” (Always Trust)**。
        
    - 关闭窗口并输入 Mac 开机密码确认更改
```

### 第三阶段：配置多项目复用（一劳永逸）
为了避免以后每个新项目都重新生成，我们通过 `devcontainer.json` 进行物理挂载。

1. **修改你的 `.devcontainer/devcontainer.json`：** 在文件中添加 `mounts` 和 `containerEnv` 配置：
    

JSON

```
{
  "name": "C# (.NET) Dev",
  "mounts": [
    // 关键：将 Mac 本地的证书文件夹映射进容器，保持路径一致
    "source=${localEnv:HOME}/.aspnet/https,target=/home/vscode/.aspnet/https,type=bind"
  ],
  "containerEnv": {
    // 告诉 .NET 运行时去哪里找证书，以及密码是什么
    "ASPNETCORE_Kestrel__Certificates__Default__Password": "aaaa2624434145",
    "ASPNETCORE_Kestrel__Certificates__Default__Path": "/home/vscode/.aspnet/https/aspnetapp.pfx",
    "ASPNETCORE_URLS": "https://+;http://+"
  },
  "forwardPorts": [5000, 5001, 5173] // 确保后端 HTTPS 端口在内
}
```

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

"source=${localEnv:HOME}/.gemini,target=/home/vscode/.gemini,type=bind",
"source=${localEnv:HOME}/.aspnet/https,target=/home/vscode/.aspnet/https,type=bind"

],

"containerEnv": {
"ASPNETCORE_Kestrel__Certificates__Default__Password": "aaaa2624434145",

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