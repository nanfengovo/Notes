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