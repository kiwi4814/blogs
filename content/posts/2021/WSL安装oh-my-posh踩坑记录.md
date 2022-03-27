+++
title = "WSL安装oh-my-posh踩坑记录"
date = 2021-11-21T20:35:47+08:00
draft = false
slug = "/wsl-oh-my-posh"
tags = ["Windows", "软件"]
categories = ["软件与工具"]
+++

整体思路参考此文：[Windows Terminal Custom Prompt Setup | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows/terminal/tutorials/custom-prompt-setup)

### 一、安装[NerdFonts](https://www.nerdfonts.com/font-downloads)

推荐安装**Caskaydia Cove Nerd Font**

![image-20211121161954784](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/image-20211121161954784.png)

### 二、安装oh-my-posh

文章中采用了两种方式，一种是为powershell单独安装，另外一种是使用包管理器Winget同时为WSL和PowerShell安装。下面采用的是第二种：

#### 1. 安装Winget

官方中文文档：[Windows 程序包管理器](https://docs.microsoft.com/zh-cn/windows/package-manager/)，应用商店直接搜索 app installer安装即可。

#### 2. 安装oh-my-posh

安装命令如下：

```
winget install JanDeDobbeleer.OhMyPosh
```

此命令将会安装：

- oh-my-posh windows程序
- oh-my-posh-wsl
- oh-my-posh 主题

安装日志如下，仅供参考

```powershell
PS C:\Users\heqifeng> winget install JanDeDobbeleer.OhMyPosh
"msstore"源要求在使用前查看以下协议。
Terms of Transaction: https://aka.ms/microsoft-store-terms-of-transaction
源要求发送当前计算机的地理区域才能正常工作。

是否同意所有源协议条款?
[Y] 是  [N] 否: Y
已找到 Oh My Posh [JanDeDobbeleer.OhMyPosh] 版本 6.8.0
此应用程序由其所有者授权给你。
Microsoft 对第三方程序包概不负责，也不向第三方程序包授予任何许可证。
Downloading https://github.com/JanDeDobbeleer/oh-my-posh/releases/download/v6.8.0/install-amd64.exe
  ██████████████████████████████  15.8 MB / 15.8 MB
已成功验证安装程序哈希
正在启动程序包安装...
已成功安装
```

##### 2.1 **查看主题**

官方文档：[prompt themes in the Oh My Posh docs](https://ohmyposh.dev/docs/themes)

文章给出的命令如下：

```
Get-ChildItem -Path "~\AppData\Local\Programs\oh-my-posh\themes\*" -Include '*.omp.json' | Sort-Object Name | ForEach-Object -Process {
$esc = [char]27
Write-Host ""
Write-Host "$esc[1m$($_.BaseName)$esc[0m"
Write-Host ""
oh-my-posh --config $($_.FullName) --pwd $PWD
Write-Host ""
}
```

这里注意路径需要根据实际情况去修改。

> WSL下的路径为：/mnt/c/Users/heqifeng/AppData/Local/Programs/oh-my-posh/bin/oh-my-posh.wsl【这里列出来没有实际作用，只是想说明，wsl的文件也在windows下的安装路径上】
>
> Windows下的路径为：C:\Users\heqifeng\AppData\Local\Programs\oh-my-posh

此处如果使用第一种方式，即单独为powershell安装的话，可以使用`Get-PoshThemes`命令直接查看主题。

【注】第二种方式没有试过这个命令，不知道是否可行（因为是重新按照第一种方式单独执行了命令，在这之前没有执行过主题命令）

##### 2.2 增加启动脚本

选择好自己的主题之后，分别在PowerShell和WSL中修改配置文件即可。

###### 2.2.1 PowerShell

修改PowerShell的启动配置文件，在末尾加上脚本：

```powershell
oh-my-posh --init --shell pwsh --config ~/jandedobbeleer.omp.json | Invoke-Expression
Set-PoshPrompt -Theme jandedobbeleer
```

其中第一行代表初始化oh-my-posh，第二行代表设置主题。

按照文档的说法在powershell中执行命令：`notepad $PROFILE`。提示记事本无法找到路径，因为该文档本身并不存在。两种解决方法：

- 执行一个判断命令，没有则创建一个

  ```powershell
  if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }
  ```

- 使用VS Code打开【[排查]([windows - Powershell: $profile is pointing to a path that I can't find and setting permanent path - Stack Overflow](https://stackoverflow.com/questions/8997316/powershell-profile-is-pointing-to-a-path-that-i-cant-find-and-setting-permane))】

  ```
  code $PROFILE
  ```

**执行`code $PROFILE`命令时遇到的第一个问题是：**

```
无法加载文件 D:\Users\heqifeng\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1，因为在此系统上禁止运行
脚本。有关详细信息，请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
所在位置 行:1 字符: 3
+ . 'D:\Users\heqifeng\Documents\WindowsPowerShell\Microsoft.PowerShell ...
+   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) []，PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

按照[文档]([关于执行策略 - PowerShell | Microsoft Docs](https://docs.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2))排查后，执行下面命令解决：

```powershell
# 查询当前的执行策略
get-executionpolicy
# 修改执行策略（以管理员身份打开powershell）
set-executionpolicy remotesigned
```

解决完执行策略的问题之后，修改完配置文件后重新打开windows terminal。

**这里遇到第二个问题：**

```powershell
Set-PoshPrompt : 无法将“Set-PoshPrompt”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。请检查名称的拼写，如果包括路径，请确保路径正确，然后再试一次。
```

**此问题没有找到原因，怀疑是第二种方式本身的安装方式有问题。解决的方法是按照第一种方式重新安装了一次：**

```powershell
Install-Module oh-my-posh -Scope CurrentUser
```

最终效果：

![image-20211121173415026](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/image-20211121173415026.png)

###### 2.2.2 WSL

对于WSL来说需要更新`.bashrc`配置文件。

使用命令 `nano .bashrc` 打开配置文件 并将下面的脚本放进去。（下面脚本中`paradox.omp.json` 为主题部分，可自行替换，你可以在`.poshthemes`这个文件夹下面看到所有的主题。 ）

```bash
eval "$(oh-my-posh-wsl --init --shell bash --config ~/.poshthemes/paradox.omp.json)"
```

参考链接：[Oh My Posh documentation](https://ohmyposh.dev/docs/windows).

【注意】这里的路径要根据实际情况变更，直接放进去会报错。我这里的安装日志如下：

```bash
# 编辑配置文件
vim .bashrc
# 增加下面的命令到末尾
eval "$(oh-my-posh-wsl --init --shell bash --config /mnt/c/Users/heqifeng/AppData/Local/Programs/oh-my-posh/themes/paradox.omp.json)"
# 使配置生效
source .bashrc
```

![image-20211121181139938](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/image-20211121181139938.png)

当然了，WSL下是可以使用oh-my-zsh的，这里预留个坑位： [WSL安装oh my zsh.md](WSL安装oh my zsh.md) 

###### 2.2.3 附第一种方式的安装说明

如果不考虑在WSL上使用oh-my-posh，可以完全参考下面这篇文章操作。

[Windows Terminal美化（oh-my-posh3）](https://zhuanlan.zhihu.com/p/354603010)

***下面是官方文章中对于第一种方式的说明。***

1. ***Using PowerShell, install Oh My Posh with the command:***

   ```powershell
   Install-Module oh-my-posh -Scope CurrentUser
   ```

2. ***Browse the prompt themes, with the command:***

   ```powershell
   Get-PoshThemes
   ```

3. ***Choose a theme and update your PowerShell profile with this command. (You can replace `notepad` with the text editor of your choice.)***

   ```powershell
   notepad $PROFILE
   ```

4. ***Add the following to the end of your PowerShell profile file to set the `paradox` theme. (Replace `paradox` with the theme of your choice.)***

   ```powershell
   Import-Module oh-my-posh
   Set-PoshPrompt -Theme paradox
   ```

### 三、安装posh-git

经过上面的安装记录，这里就非常简单了，按照步骤操作即可。

1. Install posh-git using PowerShell with the command:

   ```powershell
   Install-Module posh-git -Scope CurrentUser
   ```

2. Update your PowerShell profile file: `notepad $PROFILE`. (You can replace nodepad with the text editor of your choice).

   In your PowerShell profile, add the following to the end of the file:

   ```powershell
   Import-Module posh-git
   ```
   
3. git设置和取消代理

   ```
   git config --global https.proxy http://127.0.0.1:1080
   git config --global https.proxy https://127.0.0.1:1080
   git config --global --unset http.proxy
   git config --global --unset https.proxy
   ```

   