### 项目概述 

Watchtower 是一款开源的 Docker 容器自动更新工具，能够持续监控运行中的容器是否具有可用的新镜像版本。当检测到更新时，Watchtower 会自动拉取新镜像，并采用优雅的方式（发送 SIGTERM 信号）重启容器。

主要特性包括： 
- 支持定时任务调度（Cron 表达式） 
- 可配置自动清理旧镜像 
- 邮件/Webhook 等通知集成 
- 精确控制更新范围（通过标签或容器名称） 
- 兼容 Docker Swarm 集群环境

### docker-compose 搭建 watchtower

【注】下面代码块中给出的代码都是`compose.yml`的完整代码，复制到一个新建的`compose.yml`文件中，然后在所在的目录下命名行执行 `docker compose up -d` 即可启动项目。 

#### 简版（无通知，自动监测所有容器更新）

```yaml
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - WATCHTOWER_SCHEDULE=0 0 2 * * *
      - WATCHTOWER_CLEANUP=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

参数说明：
- `WATCHTOWER_SCHEDULE` 使用六位的 cron 表达式，控制 watchtower 的检查时间
- `WATCHTOWER_CLEANUP` 参数控制是否在更新后删除旧镜像
- 必须挂载 Docker 套接字以控制容器

其中 environment 中的几个配置也可以写成 command 的形式：

```yaml
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --schedule "0 0 2 * * *" --cleanup
```

`--schedule` 等价于 `WATCHTOWER_SCHEDULE`，`--cleanup` 等价于 `WATCHTOWER_CLEANUP`，这两者如何对应可以参照[官方文档](https://containrrr.dev/watchtower/arguments/)，为了方便统一观看，本文后续统一采用 Environment Variable  的方式。

![image-20250210182513381](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/image-20250210182513381.webp)

 
#### 指定要更新的项目

如果我们只想要自动更新指定的几个项目的话，有两种方案：

第一种是**在启动的时候指定**：
`docker run`

```bash
docker run -d \
    --name watchtower \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    allinone ddns-go
```

`docker compose`
```yaml
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - WATCHTOWER_SCHEDULE=0 0 2 * * *
      - WATCHTOWER_CLEANUP=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command:
      - allinone
      - ddns-go
```

上面的命令指定了要更新的两个容器名称：`allinone` 和 `ddns-go`，更新的时候只会检查这两个容器。

启动日志：

```bash
docker logs watchtower

level=info msg="Watchtower 1.7.1"
level=info msg="Using no notifications"
level=info msg="Only checking containers which name matches \"allinone\" or \"ddns-go\""
level=info msg="Scheduling first run: 2025-02-12 02:00:00 +0800 CST"
level=info msg="Note that the first check will be performed in 16 hours, 36 minutes, 10 seconds"
```

这种方式有个弊端，就是当我们手动执行单次监听更新的时候，还是会监控更新所有容器的：

```bash
docker exec watchtower /watchtower --run-once

......
level=info msg="Checking all containers (except explicitly disabled with label)"
......
```



第二种方式，**启用标签监控 (WATCHTOWER_LABEL_ENABLE=true)**，然后在要更新的容器的配置里手动增加 label 来表示这个容器需要自动更新，如果你要更新的项目在 watchtower 之后部署的，那么推荐这种方式来精准定位要更新的容器，这也是最一劳永逸的。

```yaml
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - WATCHTOWER_SCHEDULE=0 0 2 * * *
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_LABEL_ENABLE=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

然后在要自动更新的容器的 `compose.yml` 中增加：`labels: - "com.centurylinklabs.watchtower.enable=true"`即可。

```bash
services:
  someimage:
    container_name: someimage
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
```

如果是正在运行的容器需要在 `compose.yml` 中新增 `label`，然后重新创建下容器：

```bash
# 清理旧容器（保留数据卷）
docker compose down
# 强制重建容器（使用新配置）
docker compose up -d --force-recreate
```

再次执行测试的时候就可以看到只会监听我们设置好label的容器了。

```bash
docker exec watchtower /watchtower --run-once

level=info msg="Watchtower 1.7.1"
level=info msg="Using no notifications"
level=info msg="Only checking containers using enable label"
level=info msg="Running a one time update."
level=info msg="Session done" Failed=0 Scanned=2 Updated=0 notify=no
level=info msg="Waiting for the notification goroutine to finish" notify=no
```



#### 通知

Watchtower 可以在每次执行后通过邮件或者其他类型的消息通知，邮件通知的版本如下：

```yaml
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - WATCHTOWER_SCHEDULE=0 0 2 * * *
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_LABEL_ENABLE=true
      # 通知的部分
      - WATCHTOWER_NOTIFICATIONS=email  # 启用邮件通知
      - WATCHTOWER_NOTIFICATION_EMAIL_FROM=xxxx@163.com  # 发件人邮箱
      - WATCHTOWER_NOTIFICATION_EMAIL_TO=xxxx@qq.com  # 收件人邮箱
      - WATCHTOWER_NOTIFICATION_EMAIL_SERVER=smtp.163.com  # 邮件服务器地址
      - WATCHTOWER_NOTIFICATION_EMAIL_SERVER_PORT=587  # 邮件服务器端口
      - WATCHTOWER_NOTIFICATION_EMAIL_SERVER_USER=xxxx@163.com  # 邮箱
      - WATCHTOWER_NOTIFICATION_EMAIL_SERVER_PASSWORD=shydihdlneq  # 邮件服务器密码
      - WATCHTOWER_NOTIFICATION_EMAIL_DELAY=30  # 邮件通知延迟，单位：秒
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
