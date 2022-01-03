+++
title = "Homebrew卡在 Updating Homebrew 的解决方法"
date = "2019-12-19T23:46:30+08:00"
tags = ["Homebrew","MacOS"]
slug = "/brew-update"
draft = false
categories = ["技术"]
+++


在使用 Homebrew install的过程中，经常会卡在`Updating Homebrew...`这个过程中。

## 解决办法一：修改配置文件，取消检查更新的操作

```bash
vim ~/.zshrc

# 新增一行
export HOMEBREW_NO_AUTO_UPDATE=true
```

## 解决办法二：按住 `control （⌃）+ c` 取消本次更新操作

```bash
brew install composer
Updating Homebrew...
^C
```

按住 control + c 之后命令行会显示 **^C**，就代表已经取消了 Updating Homebrew 操作

大概不到 1 秒钟之后就会去执行我们真正需要的安装操作了

```bash
~ brew install nginx
Updating Homebrew...
^C==> Installing dependencies for nginx: pcre
==> Installing nginx dependency: pcre
==> Downloading https://mirrors.aliyun.com/homebrew/homebrew-bottles/bottles/pcre-8.43.catalina.bottle.tar.gz
######################################################################## 100.0%
==> Pouring pcre-8.43.catalina.bottle.tar.gz
🍺  /usr/local/Cellar/pcre/8.43: 204 files, 5.5MB
==> Installing nginx
==> Downloading https://mirrors.aliyun.com/homebrew/homebrew-bottles/bottles/nginx-1.17.6.catalina.bottle.tar.gz
######################################################################## 100.0%
==> Pouring nginx-1.17.6.catalina.bottle.tar.gz
==> Caveats
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
==> Summary
🍺  /usr/local/Cellar/nginx/1.17.6: 25 files, 2MB
==> Caveats
==> nginx
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
```

这个方法是临时的、一次性的

## 解决办法三：替换镜像源（推荐）

平时我们执行 brew 命令安装软件的时候，跟以下 3 个仓库地址有关：

`brew.git` 

`homebrew-core.git` 

`homebrew-bottles`

通过以下代码依次将这三个仓库的镜像源更换为不同的国内镜像源（`zsh`下）

### 1. 官方镜像源

```bash
# 替换brew.git:
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git
# 替换homebrew-core.git:
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git
# 替换homebrew-cask.git:
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git
# 应用生效
brew update
# 删除homebrew-bottles
vi ~/.zshrc
#### 按i进入输入模式，输入模式下删除 HOMEBREW_BOTTLE_DOMAIN 这一行配置，然后按:wq保存并退出
source ~/.zshrc
```

### 2. 阿里镜像源

```bash
# 替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
# 替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
# 应用生效
brew update
# 替换homebrew-bottles:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

### 3. 清华镜像源

```bash
# 替换brew.git:
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
# 替换homebrew-core.git:
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
# 替换homebrew-cask.git:
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
# 应用生效
brew update
# 替换homebrew-bottles（根据镜像地址猜测）:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

### 4. 中科大镜像源

```bash
# 替换brew.git:
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
# 替换homebrew-core.git:
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
# 替换homebrew-cask.git:
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
# 应用生效
brew update
# 替换homebrew-bottles:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

关于`homebrew-bottles`，分为以下两种情况：

【1】`bash`用户和`zsh`用户的命令不同，以上示例中关于`homebrew-bottles`的替换仅适用于`zsh`用户。

【2】`bash`用户请将配置文件`zshrc`换为`bash_profile`

下面给出清华大学镜像源给出的替换`homebrew-bottles`在`bash`下的代码，其他以此类推。

```bash
### 临时替换
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles

### 长期替换
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```