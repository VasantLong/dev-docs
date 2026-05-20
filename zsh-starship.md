# Zsh + Starship（win + wsl）

## 官网：

[Starship](https://starship.rs/zh-CN/)

[Nerd Fonts - Iconic font aggregator, glyphs/icons collection, & fonts patcher](https://www.nerdfonts.com/)



完整组合方案：Zsh + Starship + 插件

这个方案的目标是：一个启动快、信息清晰、跨平台配置统一的命令行环境。

#### 1. 安装 Zsh 并设为默认 Shell
```bash
sudo apt update && sudo apt install zsh -y
chsh -s $(which zsh)
```
重启终端后生效。首次打开 Zsh 时会出现配置菜单，选 `q` 跳过，我们后面手动管理配置。

#### 2. 安装 Oh My Zsh（Zsh 的框架，方便管理插件和主题）
```bash
# 确保已安装 git
sudo apt install git -y

# 克隆仓库到 ~/.oh-my-zsh
git clone git@github.com-personal:ohmyzsh/ohmyzsh.git ~/.oh-my-zsh

# 复制默认配置文件
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

# 切换到 Zsh
chsh -s $(which zsh)
```

#### 3. 安装两个核心插件
这两个插件会显著提升输入体验。
```bash
# 自动补全建议（灰色提示，按 → 键补全）
git clone git@github.com-personal:zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# 语法高亮（正确命令绿色，错误红色）
git clone git@github.com-personal:zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

#### 4. 启用插件
用编辑器打开 `~/.zshrc`，找到 `plugins=(git)` 这一行，修改为：
```bash
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```
保存后执行 `source ~/.zshrc`。

#### 5. 安装 Starship 提示符

**首先，在你的 WSL2 终端里依次执行以下命令**。第一行是添加 Starship 的官方软件源，第二行是更新软件列表，第三行是正式安装。

```bash
sudo sh -c 'echo "deb https://starship.rs/deb/ any main" > /etc/apt/sources.list.d/starship.list'
sudo apt update
sudo apt install starship -y
```
安装完成后`starship --version`去确认，在 `~/.zshrc` 末尾添加一行来启动它：
```bash
eval "$(starship init zsh)"
```
> **注意**：之前 `~/.zshrc` 里有一行 `ZSH_THEME="robbyrussell"`，可以在前面加 `#` 注释掉，避免和 Starship 冲突。

#### 6. 创建 Starship 配置文件
Starship 的默认配置已经很不错了，下面是一份更适合开发者的开箱即用配置。
```bash
mkdir -p ~/.config && nano ~/.config/starship.toml
```
粘贴以下内容：
```toml
# 增加命令之间的空行，阅读更清晰
add_newline = true

# 使用 Nerd Font 图标预设
format = """$all"""

[character]
success_symbol = "[➜](bold green)"
error_symbol = "[✗](bold red)"

[directory]
truncation_length = 3
style = "bold cyan"

[git_branch]
symbol = "🌱 "
style = "bold purple"

[git_status]
style = "bold yellow"

[nodejs]
symbol = "⬡ "

[python]
symbol = "🐍 "

[time]
disabled = false
format = " [🕒 $time]($style)"
style = "bright-black"
```
保存后运行 `source ~/.zshrc` 或重启终端。

#### 7. 在 Windows Terminal 中应用字体
你已经下载了 Meslo 字体，按下面步骤操作：

-   **安装**：在 Windows 中右键字体文件 → “为所有用户安装”。
-   **应用**：打开 Windows Terminal → `Ctrl + ,` 进入设置 → 左侧选择你的 WSL2 Ubuntu 配置文件 → 外观 → 字体 → 选择 **“MesloLGS NF”**。

做完这一步，图标才能正常显示。

---

### 你现在可以体验什么？
-   **智能提示**：输入命令时，灰色文字是历史命令建议，按 `→` 键直接补全。
-   **语法检查**：命令正确时绿色，错误时红色，敲错立刻知道。
-   **信息面板**：进入 Git 仓库，会显示当前分支、是否有未提交更改；进入 Node/Python 项目，会有对应图标。

如果想回到原来的 Bash，直接运行 `bash` 即可，不影响现有环境。