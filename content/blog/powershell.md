+++
authors = ["青石"]
title = "powershell与Ubuntu shell强化"
description = ""
date = 2026-04-18
[taxonomies]
tags = ["shell"]

+++

在开发过程中经常会用到命令行操作，接下来要对他们进行强化。

## Powershell强化

我们的目标是要让powershell有自动补全，查看历史记录，命令高亮和Git提示，样式优化。

安装模块
```bash
# 模糊搜索工具
winget install junegunn.fzf
# 验证安装
fzf --version
# 智能目录跳转
winget install ajeetdsouza.zoxide
# 验证安装
zoxide --version
# Git 状态显示 需要管理员权限
Install-Module posh-git -Force
# 验证安装
Get-InstalledModule posh-git
# 安装Oh My Posh
winget install JanDeDobbeleer.OhMyPosh
#验证安装
oh-my-posh version
```
安装图标字体
```bash
oh-my-posh font install
# 推荐选择：JetBrainsMono
```

写入配置文件
```bash
notepad $PROFILE
```
配置文件
```
# ---------- PSReadLine ----------
Import-Module PSReadLine
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView

# ---------- Git ----------
Import-Module posh-git

# ---------- fzf ----------
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'

# ---------- zoxide ----------
Invoke-Expression (& { (zoxide init powershell | Out-String) })

# ---------- oh-my-posh ----------

oh-my-posh init pwsh | Invoke-Expression

# ---------- Bash-style line editing ----------

# Ctrl+A → 行首
Set-PSReadLineKeyHandler -Key Ctrl+a -Function BeginningOfLine
# Ctrl+E → 行尾
Set-PSReadLineKeyHandler -Key Ctrl+e -Function EndOfLine
# Ctrl+U → 删除从光标到行首
Set-PSReadLineKeyHandler -Key Ctrl+u -Function BackwardDeleteLine
# Ctrl+K → 删除从光标到行尾
Set-PSReadLineKeyHandler -Key Ctrl+k -Function ForwardDeleteLine
```
保存后重启终端可看到美化和其他功能均出现，但有可能出现乱码，此为字体问题，修复步骤如下  
1、打开终端  
2、输入ctrl+,  
3、点击配置文件下的默认值  
4、点击默认值中的外观  
5、选择字体为JetBrainsMono Nerd Font（极端情况下需要安装字体包，如果字体包安装完后依然没有显示该字体，可以直接输入这个字体名进行使用）  
6、保存  
设置完可以看到终端正常显示了

## Ubuntu shell强化

首先我们来了解一下Terminal（终端），shell和命令行的区别：  
终端是界面工具，负责显示和输入，它本身不会执行命令，他只是把你的输入交给shell处理
shell是核心执行者，负责解析你输入的命令并交给操作系统执行  
命令行是一种操作方式，即指令本身

由此可见shell是一切的核心，而我们的强化可以从他开始。  
主流的shell是bash，zsh，fish，而他们三个的优缺点如下：  
1、bash的兼容性最强最稳定，学习资源最多，但是交互体验比较原始，自动补全等提示不够智能  
2、zsh有强大的自动补全，支持插件系统，但是配置比较复杂，写脚本时有些细节和bash会有所不同  
3、fish最简单，语法更直观，对人最友好，但是缺点也是最大最明显的，他不兼容bash脚本，并且社区和生态都比较差  

由此可以看出我们的选择是zsh

配置zsh的步骤如下  
1、打开终端  
2、输入以下命令安装zsh  
```bash
sudo apt update
sudo apt install zsh -y
zsh --version
```
3、设置zsh为默认shell
```bash
chsh -s $(which zsh)
```
第三步为linux系统正常步骤，但是由于我使用的是wsl，所以需要其他方法设置，步骤如下
```
1、进入到wsl
2、输入以下命令
nano ~/.bashrc
3、在最后一行加上exec zsh
4、重启wsl
```

4、进入后看到以下内容
```
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~).  This function can help you with a few settings that should
make your use of the shell easier.

You can:

(q)  Quit and do nothing.  The function will be run again next time.

(0)  Exit, creating the file ~/.zshrc containing just a comment.
     That will prevent this function being run again.

(1)  Continue to the main menu.

(2)  Populate your ~/.zshrc with the configuration recommended
     by the system administrator and exit (you will need to edit
     the file by hand, if so desired).

--- Type one of the keys in parentheses ---
```
选择0，创建一个最干净的 .zshrc 文件，防止与我们后续设置有冲突  
5、输入echo $0，查看是为zsh  
6、输入nano ~/.zshrc  
7、在改文件中输入以下配置
```
# Created by newuser for 5.9
# ========================
#  Locale & Editor
# ========================
export EDITOR=nano

export PATH="$HOME/.atuin/bin:$PATH"

# ========================
#  History
# ========================
HISTFILE="${XDG_STATE_HOME:-$HOME/.local/state}/zsh/history"
mkdir -p "$(dirname "$HISTFILE")"
HISTSIZE=10000
SAVEHIST=10000
setopt HIST_IGNORE_DUPS HIST_IGNORE_SPACE SHARE_HISTORY INC_APPEND_HISTORY

# ========================
#  Tools Integration
# ========================
if [[ -o interactive ]]; then
  # AtuIn
  if command -v atuin >/dev/null 2>&1; then
    eval "$(atuin init zsh)"
  fi

  # zoxide
  if command -v zoxide >/dev/null 2>&1; then
    eval "$(zoxide init zsh)"
  fi

  # starship
  if command -v starship >/dev/null 2>&1; then
    eval "$(starship init zsh)"
  fi
fi

# ========================
#  Colors & Aliases
# ========================
autoload -U colors && colors

alias ls='eza --icons=auto --group-directories-first'
alias ll='eza -lh --icons=auto --group-directories-first'
alias la='eza -lha --icons=auto --group-directories-first'

alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'

# ========================
#  Completion
# ========================
autoload -Uz compinit
mkdir -p ~/.cache/zsh
compinit -d ~/.cache/zsh/zcompdump
setopt CORRECT

if [[ -o interactive ]]; then
  source ~/.config/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
  source ~/.config/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
  source ~/.config/zsh/plugins/fzf-tab/fzf-tab.plugin.zsh

  ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=8'
  ZSH_AUTOSUGGEST_STRATEGY=(history completion)

  bindkey -e
fi

# ========================
#  Prompt
# ========================
if ! command -v starship >/dev/null 2>&1; then
  PROMPT='%F{green}%n@%m%f:%F{blue}%~%f %# '
  RPROMPT='%F{yellow}[%D{%H:%M}]%f'
fi
```
8、重启终端，正常linux到这一步不会报错，但是wsl还需要手动下载三个包
wsl会显示以下信息
```bash
compinit:527: no such file or directory: /usr/share/zsh/vendor-completions/_docker
/home/zy/.zshrc:source:62: no such file or directory: /home/zy/.config/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
/home/zy/.zshrc:source:63: no such file or directory: /home/zy/.config/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
/home/zy/.zshrc:source:64: no such file or directory: /home/zy/.config/zsh/plugins/fzf-tab/fzf-tab.plugin.zsh
zy@DESKTOP-QQE1B0K:/mnt/c/Users/zy %
```
输入如下命令
```bash
mkdir -p ~/.config/zsh/plugins && \
cd ~/.config/zsh/plugins && \
git clone https://github.com/zsh-users/zsh-autosuggestions && \
git clone https://github.com/zsh-users/zsh-syntax-highlighting && \
git clone https://github.com/Aloxaf/fzf-tab

sudo apt install fzf -y
sudo apt install eza -y
```
这三个插件分别是：  
zsh-autosuggestions右箭头自动补全（你最想要的）  
zsh-syntax-highlighting命令高亮（绿色 / 红色）  
fzf-tab TAB 补全（模糊搜索 + 下拉选择）  

eza是ls的增强版，支持图标、颜色等。

运行完后输入source ~/.zshrc，重启终端即可

9、接下来安装上键唤起历史记录
```bash
curl --proto '=https' --tlsv1.2 -sSf https://setup.atuin.sh | sh
```
注意安装过程中会出现多个选项，每个选项都需要仔细查看一下，避免误选到不需要的功能，徒增麻烦

10、安装终端美化工具Starship
```bash
curl -sS https://starship.rs/install.sh | sh
```
11、创建美化配置文件
```bash
nano ~/.config/starship.toml
```
进入后粘贴以下配置文件
```bash
add_newline = false

[character]
success_symbol = "[❯](green)"
error_symbol = "[❯](red)"

[directory]
style = "blue"
truncation_length = 3

[git_branch]
symbol = "🌿 "
style = "purple"

[git_status]
style = "red"

[nodejs]
symbol = "🟢 "

[python]
symbol = "🐍 "

[cmd_duration]
min_time = 500
format = "⏱️ [$duration]($style)"
```
粘贴完后保存推出，最后重启终端配置完成

最后如果想安装fastfetch，可以按照下面的命令安装
```bash
sudo add-apt-repository ppa:zhangsongcui3371/fastfetch
sudo apt update
sudo apt install fastfetch
```
