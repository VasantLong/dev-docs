# WSL2 开发环境全流程配置指南

> **目标**：在 Windows 上建立以 WSL2 Ubuntu 为核心的开发环境，用于“信息系统”项目开发，集成 Docker、VS Code、Git、Python、Jupyter、Node.js、Claude Code、CC Switch，并做好安全与多账号管理。
>
> **原则**：开发主战场在 WSL2 内部，Windows 仅负责 GUI 工具（浏览器、VS Code 界面、CC Switch 桌面应用通过 WSLg 显示）。所有命令行、编译、运行、版本控制都在 WSL2 里完成。

---

## 一、基础环境确认

### 1.1 确认 WSL2 与Docker Desktop
在 Windows PowerShell 中执行：
```powershell
wsl --list --verbose
```
确保 Ubuntu-26.04 为 `Running` 状态，且 Version 为 `2`。

![image-20260516183247019](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516183247019.png)

如果不是 WSL2，升级：
```powershell
wsl --set-version Ubuntu-26.04 2
```

然后打开 Docker Desktop → Settings → Resources → WSL Integration，确保 Ubuntu-26.04 开启。

### 1.2 理解 WSL2 的用户权限

WSL2 安装后默认创建的是一个**普通用户**（你安装时设置的用户名），**不是 root**。

**查看当前用户**：
```bash
whoami
# 输出你的用户名，不是 root
```

![image-20260516183410390](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516183410390.png)

**日常开发原则**：

- **所有开发操作一律用普通用户执行**（`pip install`、`npm install`、`git`、`code .` 等）
- **只有系统级操作才用 sudo**（`apt install`、`apt update`、修改系统文件等）
- **严禁以 root 身份运行日常开发命令**，否则会导致文件权限混乱（root 创建的文件普通用户改不了）

**遇到权限错误时**：
```bash
# 不要 sudo 硬来，先看文件归属
ls -la

# 如果某文件/目录属主是 root，改回来
sudo chown -R $USER:$USER ~/某目录
```

**Claude Code Agent 安全提醒**：
- 在项目目录启动 `claude`，它继承的是**当前用户权限**
- 它能读写的范围等同于你这个用户能访问的范围
- 只要不 `sudo claude`，Agent 就无法执行需要 root 的操作
- **如果 Claude Code 请求执行 `sudo` 命令，务必人工审查**

**sudo 安全原则**：
- 默认 sudo 需要输入密码（WSL2 默认行为）
- 不要在脚本或配置里免密 sudo
- Agent 建议的 sudo 命令，先在脑子里过一遍再执行

---

## 二、WSL2 内部基础配置

以下所有操作在 **WSL2 Ubuntu 终端** 中执行。

### 2.1 配置国内源

```bash
# apt 源
sudo tee /etc/apt/sources.list << 'EOF'
deb https://mirrors.aliyun.com/ubuntu/ resolute main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ resolute-updates main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ resolute-backports main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ resolute-security main restricted universe multiverse
EOF
# 清理旧的配置
sudo rm /etc/apt/sources.list


# pip 源（清华镜像）
mkdir -p ~/.pip
cat > ~/.pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF

# npm 源（淘宝镜像）
npm config set registry https://registry.npmmirror.com
```

Ubuntu 代号是 **resolute**（对应 26.04 LTS）

![image-20260515213235356](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515213235356.png)



### 2.2 创建项目目录结构

```bash
mkdir -p ~/projects/info-system
mkdir -p ~/projects/personal   # 个人项目（如需）
mkdir -p ~/projects/work       # 工作项目（如需）
```

**重要**：所有代码必须放在 WSL2 文件系统内（`~/projects/...`），**不要放在 `/mnt/c/`**。

---

## 三、开发工具安装

### 3.1 Node.js（通过 nvm）
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# 关闭并重新打开终端，或执行：
source ~/.bashrc
```

这个命令不科学上网用不了

换的gitee的镜像源下下来了，但是一要安装又走的GitHub

![image-20260515184206603](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515184206603.png)

> 失败流程
>
> `nslookup github.com 114.114.114.114`找GitHub IP 
>
> ![image-20260515184529103](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515184529103.png)
>
> 然后IP 写入 WSL2 的 hosts 文件，把域名和能通的 IP 强制绑定
>
> ![image-20260515184829090](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515184829090.png)
>
> 结果安装也没用
>
> ![image-20260515185243935](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515185243935.png)

后面改用gitee源码镜像（先把hosts 设置和代理都关了）

`export NVM_SOURCE=https://gitee.com/mirrors/nvm.git && bash /tmp/nvm_install.sh`：环境变量 `NVM_SOURCE`指向 Gitee 的镜像仓库

![image-20260515185340860](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515185340860.png)

![image-20260515185512847](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515185512847.png)

=> Appending nvm source string to /home/vasant/.bashrc
=> Appending bash_completion source string to /home/vasant/.bashrc

NVM 在安装完成后，自动在你的 `~/.bashrc` 文件末尾添加了两行代码，在wsl终端开启后自动加载

全局模块警告：

- NVM 在 Ubuntu 里检测到，你之前在 **Windows 系统**里通过 Anaconda 装过一个 Node.js（路径是 `D:\Anaconda\node-v22.13.0-win-x64`），并且用 `npm -g` 安装过一些全局模块（比如 pnpm、tailwindcss）。

- 它提醒你：**这些 Windows 下的全局模块，不会自动迁移到 WSL2 里的 NVM 管理中**。

- 如果你主要在 WSL2 里用 Node，可以**在 WSL2 里用 NVM 重新全局安装**这些工具

- Windows 下的那套，你以后如果不再需要，可以按提示卸载：

  ```bash
  nvm use system    # 切回系统自带的 Node（在 WSL2 里其实是 Windows 那个）
  npm uninstall -g a_module   # 卸载不要的全局模块
  ```

然后设置node镜像源

```bash
echo 'export NVM_NODEJS_ORG_MIRROR=https://npmmirror.com/mirrors/node' >> ~/.bashrc
source ~/.bashrc
```

![image-20260515190517779](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515190517779.png)

成功安装node.js

```bash
nvm install 24
node -v 
npm -v
```

![image-20260515190753173](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515190753173.png)

### 3.2 Python 与 venv

```bash
sudo apt install -y python3-venv python3-pip
```

![image-20260515191433216](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515191433216.png)

### 3.3 Git（Ubuntu 自带，但确认版本）

```bash
sudo apt install -y git
git --version
```

![image-20260515200737296](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515200737296.png)

### 3.4 Claude Code CLI 安装

```bash
npm install -g @anthropic-ai/claude-code
```

> **注意**：之前在 Windows cmd 用 npm 装的 Claude Code 不再使用，主力放在 WSL2 内。

![image-20260515200921525](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515200921525.png)

### 3.5 配置 Claude Code 的模型供应商

Claude Code 的模型配置保存在 `~/.claude.json` 中。你可以通过 CC Switch 桌面应用（见第四章）在图形界面中管理供应商配置，也可以直接手动编辑配置文件。

**如果要手动配置**，编辑 `~/.claude.json`，填入你的 API 供应商信息（如 DeepSeek 等兼容 OpenAI 接口的服务）

## 四、CC Switch —— AI CLI 供应商管理与切换（桌面应用）

### 4.1 CC Switch 简介

**CC Switch** 是一个**开源桌面应用**，用于统一管理多个 AI 编程 CLI（Claude Code、Codex、Gemini CLI、OpenCode、OpenClaw）的供应商配置。它把 API Key、模型选择、MCP/Skills 同步、会话管理等收进一个图形界面，避免手动编辑 JSON、TOML 或 .env 文件。

- **官网**：https://ccswitch.io/zh/
- **GitHub**：https://github.com/farion1231/cc-switch

- **安装位置**：CC Switch **必须安装在 WSL2  内部**（Linux 端），而不是 Windows 侧。因为我claude code安到 WSL2里了
- **图形界面**：安装后，它会通过 WSL2 自带的 **WSLg** 功能在 Windows 桌面上显示完整的图形界面，与 Windows 原生应用体验一致。

### 4.2 前置

#### WSLg

WSLg 是较新版本 WSL2 的内置功能，无需额外安装。按以下步骤确认其状态：

```bash
echo $DISPLAY
```

![image-20260515201016390](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515201016390.png)

**判断结果**：

- 输出类似 `:0` 或 `:1`，表示 WSLg 已启用，可以继续。
- 输出为空，则需要更新 WSL2。在 Windows 的 PowerShell 中执行：`wsl --update`完成后重启 Ubuntu 终端再试。

#### nodejs

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

![image-20260515202312594](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515202312594.png)

### 4.3 安装 CC Switch

#### Ubuntu安装包

在 WSL2 Ubuntu 终端中，进入临时目录并下载最新的 Linux 版 `.deb` 安装包：

> **获取最新版本号**：打开 https://github.com/farion1231/cc-switch/releases ，查看最新的 `CC-Switch-v{版本号}-Linux.deb` 文件名，替换上面命令中的版本号。wget没安装成功

```bash
# 使用 dpkg 安装 .deb 包
sudo dpkg -i CC-Switch-v{版本号}-Linux.deb

# 如果有依赖问题
sudo apt-get install -f
```

![image-20260515211208715](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515211208715.png)

![image-20260515211437807](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515211437807.png)

```bash
# 检查安装是否成功
which cc-switch
# 应输出类似 /usr/bin/cc-switch 的路径
```

####  运行
```bash
cc-switch
```

![image-20260515211558733](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515211558733.png)

稍等片刻，CC Switch 的图形界面窗口就会出现在 Windows 桌面上（通过 WSLg 显示）。

![image-20260515214412166](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515214412166.png)

开启应用级画面之后（余额是我claude code测试了一次）

![image-20260516185327856](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516185327856.png)

#### 出现方块：需要安装必要的中文字体

```bash
sudo apt update
sudo apt install -y fonts-noto-cjk fonts-wqy-microhei fonts-wqy-zenhei
```

![image-20260515212911542](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515212911542.png)

**可选**：首次启动后，你可以将 `cc-switch` 添加到 WSL2 的启动脚本中，让它在每次打开终端时自动在后台运行：

```bash
# 在 ~/.bashrc 末尾添加（可选）
echo 'cc-switch &>/dev/null &' >> ~/.bashrc
```

#### 卸载方式

```bash
# Debian/Ubuntu
sudo apt remove cc-switch
```



wsl链接代理（好像没用）

![image-20260515221320297](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515221320297.png)



#### GitHub hosts

`sudo nano /etc/hosts`文件内添加

```
140.82.113.4    github.com
140.82.113.4    gist.github.com
185.199.108.153 assets-cdn.github.com
151.101.1.194   github.global.ssl.fastly.net
```

1. `Ctrl + X` - 退出（会提示是否保存）
2. `Y` - 确认保存修改
3. `Enter` - 确认文件名（直接回车即可）

再重启

![image-20260515224143196](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515224143196.png)

刚发现公钥好像没整

```
# 1. 测试 DNS 解析
nslookup github.com

# 2. 测试网络连通性
curl -I https://github.com

# 3. 测试 Git 连接
ssh -T git@github.com  # SSH 方式
git ls-remote https://github.com/user/repo.git  # HTTPS 方式
```

强制所有 GitHub HTTPS 转换为 SSH

`git config --global url."git@github.com:".insteadOf "https://github.com/"`



测试与GitHub连接

![image-20260515225728373](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260515225728373.png)

> 第1跳: 192.168.3.1     - 你的路由器 (正常)
> 第2跳: 192.168.18.1    - 可能是光猫/上级设备 (正常)
> 第3跳: 172.16.128.1    - 运营商内网 (开始出现丢包 *)
> 第6跳: 219.158.113.49  - 中国联通骨干网
> 第11跳: 104.44.235.186 - 微软网络 (GitHub 使用 Azure)
> 第15跳: 104.44.20.51   - 接近目标
> 第18跳: 51.10.12.48    - GitHub 服务器段
>
> 大量 `* * *` 表示中间节点不响应 ICMP，但流量实际到达了。`curl` 失败是因为 HTTPS 443 端口被拦截。你的路由器（`192.168.3.1`）可能在 **应用层** 对 HTTPS 流量进行过滤，允许 ICMP (traceroute/SSH) 但拦截 HTTPS 请求中的某些特征。

### 4.4 工作原理与使用方式

- **不要手动设置环境变量**：CC Switch 的 GUI 会自动接管配置，手动修改 Windows 或 WSL2 的环境变量可能导致冲突。
- **数据存储位置**：所有配置文件和 API Key 都安全地保存在 WSL2 Ubuntu 内的 SQLite 数据库中（`~/.cc-switch/cc-switch.db`）。
- **切换供应商后**：Claude Code 支持热切换，通常无需重启终端；其他 CLI 工具需重启。

---

## 五、Git 多账号配置

### 5.1 生成 SSH 密钥对

**个人账号密钥**：

```bash
ssh-keygen -t ed25519 -C "157333230+VasantLong@users.noreply.github.com" -f ~/.ssh/id_ed25519_personal
```

**工作账号密钥**（如需）：

```bash
ssh-keygen -t ed25519 -C "工作邮箱@company.com" -f ~/.ssh/id_ed25519_work
```

将各自的公钥（`~/.ssh/id_ed25519_xxx.pub`）添加到对应的 GitHub 账号 SSH Keys 设置中。

![image-20260516190856248](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516190856248.png)

### 5.2 配置 SSH Config
```bash
nano ~/.ssh/config
```
写入：
```
# 个人 GitHub
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

# 工作 GitHub
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

验证：
```bash
ssh -T git@github.com-personal
ssh -T git@github.com-work
```

就先创了一个personal

![image-20260516122655116](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516122655116.png)

### 5.3 创建子 Git 配置文件

```bash
# 个人账号
cat > ~/.gitconfig-personal << 'EOF'
[user]
    name = xxxxx
    email = xxxxx
EOF

# 工作账号（如需）
cat > ~/.gitconfig-work << 'EOF'
[user]
    name = 你的工作姓名
    email = 工作邮箱@company.com
EOF
```

### 5.4 编辑主 Git 配置
```bash
nano ~/.gitconfig
```
写入：
```ini
[alias]
    lg = log --oneline --graph --decorate --all
    st = status
    co = checkout
    br = branch

[core]
    autocrlf = input
    editor = code --wait

[color]
    ui = auto

[includeIf "gitdir:/home/你的用户名/projects/personal/"]
    path = ~/.gitconfig-personal

[includeIf "gitdir:/home/你的用户名/projects/work/"]
    path = ~/.gitconfig-work
```

> **重要**：将 `你的用户名` 替换为实际用户名，路径末尾 `/` 不能省。

### 5.5 验证多账号配置
```bash
# 创建测试仓库
cd ~/projects/personal && mkdir test && cd test && git init
git config user.name   # 应输出个人用户名
cd ~/projects/work && mkdir test && cd test && git init
git config user.name   # 应输出工作姓名
# 测试完删除
rm -rf ~/projects/personal/test ~/projects/work/test
```

![image-20260516124119843](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516124119843.png)

ubuntu里的git最新版本是2.54.0  [Git - Install for Linux](https://git-scm.com/install/linux)

### 5.6 Git Clone 注意事项

**① 路径必须在 WSL2 内部**

```bash
# 正确：先进入 WSL2 文件系统
cd ~/projects/
git clone git@github.com-personal:用户名/仓库.git

# 错误：绝对不要在 /mnt/c/ 下操作
# cd /mnt/c/Users/...  ← 不要这样做
```

**② 使用 SSH 别名克隆**

克隆时必须使用 `~/.ssh/config` 中定义的 Host 别名，否则无法匹配对应的 SSH 密钥：
```bash
# 个人项目
git clone git@github.com-personal:用户名/个人仓库.git

# 工作项目
git clone git@github.com-work:公司名/工作仓库.git
```

**③ 克隆后检查行尾符**
```bash
cd 仓库名
git config core.autocrlf input
```

**④ 验证用户身份**
```bash
git config user.name   # 确认输出与你期望的账号一致
git config user.email
```

如果输出不对，检查 `~/.gitconfig` 中 `includeIf` 的路径是否正确（末尾必须带 `/`）。

---

## 六、VS Code 集成

### 6.1 安装 Remote-WSL 扩展
在 Windows 的 VS Code 中安装 **Remote - WSL** 扩展（Microsoft 官方）。

![image-20260516191554122](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516191554122.png)

### 6.2 使用方式
在 WSL2 终端中任意目录`code .`即可进入

VS Code 左下角会显示 `WSL: Ubuntu-26.04`，终端自动为 WSL2 bash。

![image-20260516191241378](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516191241378.png)

---

## 七、Python 与 Jupyter 环境配置

需要先整个虚拟环境

![image-20260516124725712](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516124725712.png)

### miniforge

无奈在win端下载https://mirrors.tuna.tsinghua.edu.cn/github-release/conda-forge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh

```bash
# 复制 Windows 下载的 Mambaforge（找地址）
cp /mnt/d/Edge/Miniforge3-Linux-x86_64.sh ~/

# 确认文件存在
ls -lh ~/Miniforge3-Linux-x86_64.sh

# 安装
bash ~/Miniforge3-Linux-x86_64.sh
```

- `Enter` 查看许可
- 输入 `yes` 接受许可
- `Enter` 确认安装位置（默认 `~/miniforge3`）
- 输入 `yes` 允许初始化

```
# 重新加载
source ~/.bashrc

# 验证
mamba --version
```

### 在base配置jupyter

```bash
# 在 base 环境中安装 nb_conda_kernels
mamba install -n base jupyterlab nb_conda_kernels

# 创建其他环境时不需要装 jupyterlab
mamba create -n data_science python=3.11 numpy pandas matplotlib -y
mamba create -n ml_project python=3.11 scikit-learn tensorflow -y

# 启动 JupyterLab 后，可以在界面中切换不同环境的内核
jupyter lab
```

```
不用切环境，直接往指定环境装包：
mamba install -n ml tensorflow -y
```

### 7.2 在其他环境安装ipykernel

`mamba install ipykernel -y`

> #### 以前的内核管理方式
> pip install ipykernel
>
> pip install pandas fastapi uvicorn
>
> 可以修改kernel名字
>
> python -m ipykernel install --user --name=info-sys --display-name="Python (info-sys)"
> deactivate
>
> **其他项目同理**，只需修改 `--name` 和 `--display-name`。

![image-20260516164848520](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516164848520.png)

### 7.4 管理 Kernel

检查kernel`python -m nb_conda_kernels list`

![image-20260516161021112](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516161021112.png)

还有一个命令**`jupyter kernelspec list`**

但是没有上述效果，只有各自环境自己的kernel，不知道什么问题

![image-20260516165401480](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516165401480.png)

删除环境内的包：`mamba remove -n 环境名 环境包含的包`

删除整个环境：`mamba remove -n 环境名 --all`

### 7.3 日常使用

**终端：以特定路径启动 Jupyter**：

```bash
jupyter lab --notebook-dir=~/projects
```
Windows 浏览器访问 `http://localhost:8888`，新建 Notebook 后在右上角切换 Kernel。

> 可在~/.bashrc中设置`alias jlab='jupyter lab --notebook-dir=~/projects'`定义一个常用工作路径，`source ~/.bashrc`之后用jlab运行

**在 VS Code 中使用**：
直接打开 `.ipynb` 文件，右下角点击选择 Kernel，所有注册过的环境都会显示。

---

## （未测试）八、Docker 容器化 Jupyter 环境：特定项目使用方案

### 8.1 项目结构示例
```
~/projects/info-system/
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
└── notebooks/           # 挂载目录，存 .ipynb 文件
```

### 8.2 Dockerfile 模板
```dockerfile
FROM python:3.12-slim
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple jupyterlab
# 按需添加项目依赖
# RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pandas fastapi
WORKDIR /workspace
EXPOSE 8888
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", "--NotebookApp.token=mysecret"]
```

### 8.3 docker-compose.yml 模板
```yaml
version: '3'
services:
  jupyter:
    build: .
    ports:
      - "18888:8888"
    volumes:
      - ./notebooks:/workspace
```
> 不同项目使用不同宿主机端口，避免冲突（如 18888、28888）。

### 8.4 启动与访问
```bash
cd ~/projects/info-system
docker compose up -d
```
浏览器访问 `http://localhost:18888`，输入 token `mysecret`。

---

## 九、Claude Code Agent 安全守则

1. **限定工作目录**：只在项目目录（如 `~/projects/info-system`）启动 Claude Code，杜绝从 `~` 或 `/` 启动。
2. **审查命令**：不要盲目按 Y，尤其涉及 `rm`、`mv`、`chmod`、`curl ... | bash` 的命令。
3. **版本控制保底**：所有代码进 Git，每次 Agent 任务后执行 `git diff` 审查改动。
4. **敏感文件隔离**：
   - `.env`、密钥文件放项目外，或用 `.gitignore` 排除
   - 不在 Claude Code 会话中暴露高权限 Token
5. **隐私意识**：代码和提示词会发送到模型 API（DeepSeek），机密逻辑需脱敏或考虑本地模型。

![image-20260516171613549](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516171613549.png)

---

## extra：安装回收站`trash-cli`

在开始前，建议先在 WSL 终端里更新一下软件源：
```bash
sudo apt update
```

### 安装（apt）
这是最简单、最通用的方式，适用于 Ubuntu 的稳定发行版。

```bash
sudo apt install trash-cli
```
也可以通过 Pip 安装（获取更新版本）

安装完，用下面命令看看是否成功：
```bash
trash-put --version
```

![image-20260516173002182](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516173002182.png)

最常用的几个命令：

-   **安全删除（移入回收站）**：`trash-put 文件名`
-   **查看回收站内容**：`trash-list`
-   **恢复文件**：`trash-restore` （会进入交互界面让你选择恢复哪个）
-   **清空回收站**：`trash-empty` （后面可加数字表示清空多少天前的，如 `trash-empty 7` 清空7天前的）

**小技巧**：如果你习惯用 `rm`，可以在 `~/.bashrc` 里加一行 `alias rm='trash-put'`，这样每次打 `rm` 其实都在安全删除。不过要小心别在没装这个工具的服务器上养成习惯。

### 示例

以`/home/vasant/projects/personal/jupyter/Untitled.ipynb` 为例，演示 `trash-cli` 的核心操作。

#### 1. 删除文件（移入回收站）
将文件安全地移到回收站，而不是永久删除。

```bash
trash-put /home/vasant/projects/personal/jupyter/Untitled.ipynb
```
如果当前就在 `jupyter` 目录下，直接用相对路径也行：
```bash
trash-put Untitled.ipynb
```

#### 2. 查看回收站里的文件
想确认是否删除成功，或者查看所有已删除的文件。

```bash
trash-list
```
输出包含完整路径和删除时间：

![image-20260516182605332](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516182605332.png)

#### 3. 恢复文件
```bash
trash-restore
```
会列出所有被删文件

![image-20260516182647807](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516182647807.png)

然后输入对应数字 `0` 并回车，文件就原路恢复了。

#### 4. 清空回收站（真正的永久删除）
如果确定这些文件不再需要了。

```bash
# 清空所有文件
trash-empty

# 只清空7天前删除的文件（7天内的会保留）
trash-empty 7
```
执行后，这些文件才被真正永久删除。

![image-20260516182834951](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516182834951.png)

---

**小补充**：如果需要用通配符删除一批 `.ipynb` 文件，比如 `trash-put *.ipynb`，建议先用 `trash-list` 查看确认一下这些文件你都不需要恢复了，然后再清空。

### 回收路径

1. 文件在 WSL 自己的虚拟磁盘内

`/home/vasant/.local/share/Trash`

- `files/` 目录：存放被删文件的原始数据，文件名可能变成 `Untitled.ipynb`。
- `info/` 目录：存放对应的 `.trashinfo` 元数据文件，记录了原路径和删除时间。

查看

```
# 查看回收站里的文件和元数据
ls -la /home/vasant/.local/share/Trash/files
ls -la /home/vasant/.local/share/Trash/info
```

![image-20260516182736958](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20260516182736958.png)

2. 文件在外部或挂载的分区（如 `/mnt/c`）

如果你删除 `/mnt/c/somefile.txt`（即 Windows 文件），`trash-cli` 会把文件移动到**那个分区的根目录**下的回收站里。

- **回收站位置**：
  `/mnt/c/.Trash-1000/`
  （`1000` 是你的用户ID，可通过 `id -u` 查看）
- **内部结构**：同样包含 `files` 和 `info` 子目录。

## 十、日常开发工作流速查

| 步骤           | 操作                                                         |
| -------------- | ------------------------------------------------------------ |
| 打开项目       | WSL2 终端：`cd ~/projects/info-system && code .`             |
| Git 操作       | 在 VS Code 终端或 WSL2 终端执行，注意使用 SSH 别名           |
| Python 开发    | `source .venv/bin/activate`，`pip install ...`               |
| Jupyter        | `jupyter lab --ip=0.0.0.0 --port=8888 --no-browser`          |
| 前端开发       | `npm start`（跑在 WSL2 内，Win 浏览器访问 `localhost:3000`） |
| Docker 服务    | `docker compose up -d`                                       |
| Claude Code    | 在项目目录执行 `claude`，开始对话                            |
| 管理模型供应商 | 启动 CC Switch 桌面应用进行切换和配置                        |

---

## 十一、配置检查清单

| 检查项              | 命令                                         |
| ------------------- | -------------------------------------------- |
| WSL2 版本           | `wsl --list --verbose`（PowerShell）         |
| 当前用户（非 root） | `whoami`                                     |
| WSLg 状态           | `echo $DISPLAY`（应输出 `:0` 或 `:1`）       |
| Docker 镜像         | `docker info \| grep "Registry Mirrors"`     |
| Node.js             | `node -v`（应显示 v20.x）                    |
| Python              | `python3 --version`                          |
| Git                 | `git --version`                              |
| SSH 个人            | `ssh -T git@github.com-personal`             |
| SSH 工作            | `ssh -T git@github.com-work`                 |
| Git 多用户          | 在对应目录执行 `git config user.name`        |
| Jupyter Kernel      | `jupyter kernelspec list`                    |
| Claude Code         | `claude --version`                           |
| CC Switch           | 在终端执行 `cc-switch`，确认图形界面正常启动 |

---

以上即为你从 Windows 迁移到 WSL2 开发环境的全流程文档，已修正所有关于 CC Switch 的错误描述（CC Switch 是桌面应用，不是命令），补充了在 WSL2 中通过 WSLg 安装和启动 CC Switch 的完整流程，并整合了用户权限说明和 Git Clone 注意事项。如需对某个环节做进一步定制，可以继续告诉我。