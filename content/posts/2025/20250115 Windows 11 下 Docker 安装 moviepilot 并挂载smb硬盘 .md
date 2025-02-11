---
title: Windows Nas 安装 MoviePilot及挂载smb盘
date: 2025-01-16T16:46:31.000
slug: /docker-compose-moviepilot
draft: false
tags:
  - Docker
  - MoviePilot
categories:
  - Docker
  - 家庭影院
series:
  - Docker
  - 家庭影院
enableTOC: false
---

### 背景

使用Windows 11作为媒体服务器，已经安装好`Docker Desktop`以及 `wsl2（ Ubuntu 24.04.1 LTS）`。

### 介绍

https://wiki.movie-pilot.org/

项目本身在官方文档中已经有详细的介绍了，本文主要是对自己安装和使用过程中遇到的一些问题做额外的补充。

### 准备工作

#### WSL挂载smb硬盘

由于我本机仅作为服务器使用，没有额外的硬盘，所以挂载了NAS的硬盘来完成资源的存储和刮削，如果有同样需求的可以参考执行。

1. 安装 cifs-utils 工具包

   ```bash
   sudo apt update
   sudo apt install cifs-utils
   ```

2. 挂载 SMB 共享到 WSL 中

   ```bash
   # 创建一个挂载文件夹
   sudo mkdir /mnt/moviepilot
   # 挂载到指定的目录下
   sudo mount -t cifs //10.0.0.88/nas/MoviePilot /mnt/moviepilot -o guest,iocharset=utf8,file_mode=0777,dir_mode=0777
   ```

3. 持久化挂载

   ```bash
   vim /etc/fstab
   # 添加以下行
   //10.0.0.88/nas/MoviePilot /mnt/moviepilot cifs guest,iocharset=utf8,file_mode=0777,dir_mode=0777,nofail 0 0
   ```

4. 测试挂载: 运行 `sudo mount -a` 命令测试是否可以根据 /etc/fstab 文件挂载所有文件系统，如果没有报错，则表示配置成功。



**需要注意的是挂载的时候一定要指定编码——`iocharset=utf8` ，否则在挂载盘中创建的文件会在其他挂载smb盘的系统中显示为乱码。**



### MoviePilot-V2安装

在wsl中采用compose的方式进行安装，登录到 wsl 中，然后在任意目录创建compose文件：

下面给出的是极简版，带有注释的部分请参照：[安装指引 | MoviePilot Wiki](https://wiki.movie-pilot.org/zh/install) 以及详细的环境变量的说明：[配置参考 | MoviePilot Wiki](https://wiki.movie-pilot.org/zh/configuration)

```yml
services:
    moviepilot:
        stdin_open: true
        tty: true
        container_name: moviepilot-v2
        hostname: moviepilot-v2
        network_mode: bridge
        ports:
            - target: 3000
              published: 3000
              protocol: tcp
        volumes:
            - './config:/config'
            - './core:/moviepilot'
            - '/var/run/docker.sock:/var/run/docker.sock:ro'
            - '/mnt/moviepilot:/media'
        environment:
            - 'NGINX_PORT=3000'
            - 'PORT=3001'
            - 'PUID=0'
            - 'PGID=0'
            - 'UMASK=000'
            - 'TZ=Asia/Shanghai'
            - 'MOVIEPILOT_AUTO_UPDATE=true'
            - 'SUPERUSER=admin'
        restart: always
        image: jxxghp/moviepilot-v2:latest
```

其中 `./config` 和 `./core` 的映射可以修改为自己设置的其他路径，这里用的是和 `compose.yml` 文件在同一文件夹下的相对路径。

`/mnt/moviepilot` 的挂载时在WSL中进行的，路径也是WSL的路径，所以执行命令的时候要登录到WSL中进行操作。

执行：

```bash
docker compose up -d
```

启动完成后，查看初始化分配的管理员密码：

```bash
docker logs moviepilot-v2
```

### 容器自动更新

为了统一管理docker容器的更新，通过watchtower来对moviepilot进行更新。

安装指南请参照 [[Docker 部署 Watchtower 自动更新容器]]



### 下载器

下载器如果选择 docker 部署的话，可以参照下面的 compose 文件进行部署。

#### qBittorrent

```yaml
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=0
      - PGID=0
      - TZ=Asia/Shanghai
      - UMASK=002
      - WEBUI_PORT=9995
      - TORRENTING_PORT=13666
    volumes:
      - "./appdata:/config"
      - "/mnt/moviepilot:/media"
    ports:
      - 9995:9995
      - 13666:13666
      - 13666:13666/udp
    restart: unless-stopped
```

其中 `environment` 中的 `WEBUI_PORT` 和 `TORRENTING_PORT` 的默认值分别是 8080 和 6881，这两个值最好都改掉，8080 端口会导致启动后无法在宿主机访问 WEBUI，而 6881 端口是默认被运营商封禁的，我这里使用了 9995 和 13666，可以根据自己的需求执行修改，修改的时候要注意 对应 ports 下面的映射也要改掉。



启动完成后可以执行 `docker logs qbittorrent` 来查看默认分配的管理员密码，然后登录 WEBUI 即可。

![image-20250211144315208](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/image-20250211144315208.webp)

PS：一开始没改 8080 端口，在启动完成后需要修改 `qBittorrent.conf` 文件，在 `[Preferences]` 下面增加下面的两行才可以，折腾了好久。

```ini
WebUI\HostHeaderValidation=false
WebUI\CSRFProtection=false
```

#### transmission

Docker Compose

```yaml
services:
  transmission:
    image: ddsderek/transmission:v3 # latest是V4，这里我们需要安装V3，一些站是不允许V4的
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=022
      - TZ=Asia/Shanghai
      - USER=admin
      - PASS=123456
      - TRANSMISSION_WEB_HOME=/trguing-cn/ # 设置 Web UI 界面 可选 /transmission-web-control/ /transmissionic/ /combustion/ /kettu/ /flood/ /trguing/ /trguing-cn/
    volumes:
      - "./config:/config"
      - "/mnt/moviepilot:/downloads" # 这里挂载了SMB硬盘
      - "./watch:/watch"
    ports:
      - 9091:9091
      - 51413:51413
      - 51413:51413/udp
    restart: unless-stopped
```

这里的镜像用的是[大神的修改版](https://github.com/DDS-Derek/transmission-Docker)，除了版本问题，其他没什么需要注意的了，启动后就可以愉快的使用了。









