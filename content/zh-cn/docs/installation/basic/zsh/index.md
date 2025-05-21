---
title: "zsh"
linkTitle: "zsh"
date: 2024-01-18
weight: 40
description: >
  安装配置zsh和ohmyzsh!
---

##  安装配置 zsh

### 安装zsh

首先安装 zsh：

```bash
sudo apt install zsh zsh-doc
```

### 安装 Oh my zsh!

在 zsh 终端执行:

```bash
# 如果被墙则增加代理设置
# export all_proxy=socks5://192.168.2.1:7891

sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

> DNS 污染问题：
>
> 如果遇到 DNS 污染，导致 raw.githubusercontent.com 被解析到 127.0.0.1 或者 0.0.0.1 导致无法访问。需要修改 hosts 文件:
>
> sudo vi /etc/hosts
>
> 增加一行：
>
> 199.232.68.133 raw.githubusercontent.com

中途询问是否把zsh作为默认 shell 时选择Y：

```bash
Do you want to change your default shell to zsh? [Y/n] Y
Changing the shell...
```

### 安装插件

安装 wd 插件：

```bash
sh -c "$(curl -fsSL https://github.com/mfaerevaag/wd/raw/master/install.sh)"
```

配置以下插件：

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-history-substring-search.git $ZSH_CUSTOM/plugins/history-substring-search
git clone https://github.com/Pilaton/OhMyZsh-full-autoupdate.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/ohmyzsh-full-autoupdate
```

修改 zsh 配置

```bash
vi ~/.zshrc
```

注释掉这一行：

```bash
DISABLE_MAGIC_FUNCTIONS="true"
```

修改 plugins 为

```bash
plugins=(    
    git
    golang
    rust
    docker
    docker-compose 
    kubectl
    npm
    node
    mvn
    sudo
    helm
    redis-cli
    wd 
    zsh-autosuggestions
    zsh-syntax-highlighting
    history-substring-search
    ohmyzsh-full-autoupdate
)

# User configuration

ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets pattern cursor root line)
ZSH_HIGHLIGHT_PATTERNS=('rm -rf *' 'fg=white,bold,bg=red')
```

重启 zsh。

```bash
Updating plugins and themes Oh My ZSH
--------------------------------------

Updating Plugin — ohmyzsh-full-autoupdate -> https://github.com/Pilaton/OhMyZsh-full-autoupdate
Already up to date.

Updating Plugin — zsh-autosuggestions -> https://github.com/zsh-users/zsh-autosuggestions
Already up to date.

Updating Plugin — zsh-syntax-highlighting -> https://github.com/zsh-users/zsh-syntax-highlighting
Already up to date.

Updating Theme — powerlevel10k -> https://github.com/romkatv/powerlevel10k
Already up to date.
```

备注： 这个自动更新可能会因为 github.com 被墙无法访问而失败。可以修改 .zshrc 的设置，默认开启代理避免更新时被墙：

```bash
# auto start proxy on
export all_proxy=socks5://192.168.2.1:7891;export http_proxy=http://192.168.2.1:7890;export https_proxy=http://192.168.2.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16
```

然后手工更新 ohmyzsh：

```bash
# proxyon
omz update
```

执行完 ohmyzsh 的更新之后，关闭所有的终端，再重新打开，就会触发 zsh plugins 的自动更新。

## 配置网络代理

修改 zsh 配置

```bash
vi ~/.zshrc
```

增加以下内容：

```bash
# set proxy for different locations
alias proxyon-nansha='export all_proxy=socks5://192.168.0.1:7891;export http_proxy=http://192.168.0.1:7890;export https_proxy=http://192.168.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyon-tianhe='export all_proxy=socks5://192.168.2.1:7891;export http_proxy=http://192.168.2.1:7890;export https_proxy=http://192.168.2.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyon-fenghu='export all_proxy=socks5://192.168.3.1:7891;export http_proxy=http://192.168.3.1:7890;export https_proxy=http://192.168.3.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyon-local='export all_proxy=socks5://127.0.0.1:7897;export http_proxy=http://127.0.0.1:7897;export https_proxy=http://127.0.0.1:7897;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
# set default proxy by this line
alias proxyon='proxyon-local'
alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
# uncomment next line to enable proxy by default when zsh is opened
# proxyon
```



