机器：R1安装了飞牛 OS

网卡信息：

> 1. **br-84c6cfce7a94:**
>    - 
>    - **类型：** Bridge (网桥) 接口，通常由 Docker 等虚拟化工具创建。
>    - **标志位:** UP, BROADCAST, RUNNING, MULTICAST，表示接口已启用、支持广播、正在运行、支持组播。
>    - **IPv4 地址：** 172.19.0.1，子网掩码 255.255.0.0，广播地址 172.19.255.255。
>    - **IPv6 地址：** 有一个 Link-local IPv6 地址 fe80::42:45ff:fef2:fac7。
>    - **MAC 地址：** 02:42:45:f2:fa:c7。
>    - **流量信息:** 有接收和发送数据包，说明该网桥接口在工作。
>    - **总结：** 这是一个由 Docker 创建的桥接网络接口。
> 2. **br-8cf9796bedaf:**
>    - 
>    - **类型：** Bridge (网桥) 接口，通常由 Docker 等虚拟化工具创建。
>    - **标志位:** UP, BROADCAST, MULTICAST，表示接口已启用、支持广播、支持组播。
>    - **IPv4 地址：** 172.18.0.1，子网掩码 255.255.0.0，广播地址 172.18.255.255。
>    - **MAC 地址：** 02:42:6b:51:79:cf。
>    - **流量信息:** 没有接收和发送的数据包，说明可能这个网桥没有容器在使用。
>    - **总结：** 另一个由 Docker 创建的桥接网络接口。
> 3. **docker0:**
>    - 
>    - **类型：** Bridge (网桥) 接口，由 Docker 创建。
>    - **标志位:** UP, BROADCAST, MULTICAST，表示接口已启用、支持广播、支持组播。
>    - **IPv4 地址：** 172.17.0.1，子网掩码 255.255.0.0，广播地址 172.17.255.255。
>    - **MAC 地址：** 02:42:e7:12:c4:f8。
>    - **流量信息:** 没有接收和发送的数据包，说明可能没有容器在使用该网络。
>    - **总结：** 这是 Docker 默认的桥接网络接口。
> 4. **enp2s0:**
>    - 
>    - **类型：** 以太网接口（物理网卡）。
>    - **标志位:** UP, BROADCAST, MULTICAST，表示接口已启用、支持广播、支持组播。
>    - **MAC 地址：** 00:90:27:f1:52:40。
>    - **流量信息:** 没有接收和发送的数据包，说明该网卡可能没有连接网线或没有配置 IP 地址。
>    - **总结：** 这是一个物理网卡，但当前没有配置网络。
> 5. **enp3s0:**
>    - 
>    - **类型：** 以太网接口（物理网卡）。
>    - **标志位:** UP, BROADCAST, RUNNING, MULTICAST，表示接口已启用、支持广播、正在运行、支持组播。
>    - **IPv4 地址：** 10.0.0.200，子网掩码 255.255.255.0，广播地址 10.0.0.255。
>    - **IPv6 地址：** 有 Link-local IPv6 地址和 Global IPv6 地址。
>    - **MAC 地址：** 00:90:27:f1:52:41。
>    - **流量信息:** 有大量的接收和发送数据包，说明该网卡正在使用，且有网络连接。
>    - **总结：** 这是你的主要网络接口，有 IP 地址且正在工作。
> 6. **enp4s0:**
>    - 
>    - **类型：** 以太网接口（物理网卡）。
>    - **标志位:** UP, BROADCAST, MULTICAST，表示接口已启用、支持广播、支持组播。
>    - **MAC 地址：** 00:90:27:f1:52:42。
>    - **流量信息:** 没有接收和发送的数据包，说明该网卡可能没有连接网线或没有配置 IP 地址。
>    - **总结：** 这是一个物理网卡，但当前没有配置网络。
> 7. **enp5s0:**
>    - 
>    - **类型：** 以太网接口（物理网卡）。
>    - **标志位:** UP, BROADCAST, MULTICAST，表示接口已启用、支持广播、支持组播。
>    - **MAC 地址：** 00:90:27:f1:52:43。
>    - **流量信息:** 没有接收和发送的数据包，说明该网卡可能没有连接网线或没有配置 IP 地址。
>    - **总结：** 这是一个物理网卡，但当前没有配置网络。
> 8. **lo:**
>    - 
>    - **类型：** Loopback (回环) 接口，用于本机内部通信。
>    - **标志位:** UP, LOOPBACK, RUNNING，表示接口已启用、回环、正在运行。
>    - **IPv4 地址：** 127.0.0.1。
>    - **IPv6 地址：** ::1。
>    - **总结：** 这是一个标准的本地回环接口，用于本机网络测试。
> 9. **vethe9eadc3:**
>    - 
>    - **类型：** 虚拟以太网接口 (virtual Ethernet interface)，通常与 Docker 网络一起创建。
>    - **标志位:** UP, BROADCAST, RUNNING, MULTICAST，表示接口已启用、支持广播、正在运行、支持组播。
>    - **IPv6 地址：** 有 Link-local IPv6 地址
>    - **流量信息:** 有接收和发送数据包
>    - **总结：** 这是 Docker 创建的虚拟网络接口，和容器的网络相关。
>
> **总结：**
>
> - 
> - br-xxxx 和 docker0 开头的接口是 Docker 创建的桥接网络接口，用于容器之间的通信。
> - enp2s0, enp3s0, enp4s0, enp5s0 是你的物理网卡接口。
> - enp3s0 有 IP 地址，并有数据包接收和发送，说明它当前正在被使用。
> - lo 是本地回环接口，用于本机内部通信。
> - vethe9eadc3 是虚拟网络接口。

在 飞牛（Debian） 系统上使用多网卡方案，在 Docker 中安装 OpenWrt 作为旁路由，并使用 enp5s0 网卡，且设置 OpenWrt 的 IP 地址为 10.0.0.10。主路由器的 IP 地址为 10.0.0.1，Debian 系统使用的 IP 地址为 10.0.0.200 (在 enp3s0 上)。

**详细步骤：**

1. **确认网卡状态:**

   - 使用 ip link 或 ifconfig -a 命令确认 enp5s0 确实没有被使用（没有 IP 地址）。

     ```
     ip link
     # 或
     ifconfig -a
     ```

   - 确保 enp5s0 上没有连接网线（或者暂时断开），避免 IP 冲突。

2. **设置 enp5s0 为混杂模式:**

   ```
   sudo ip link set enp5s0 promisc on
   ```

3. **创建 macvlan 网络:**

   - `--subnet` 参数：指定 OpenWrt 网络使用的子网，这里是你的主路由器所在的网段。
   - `--gateway` 参数：指定 OpenWrt 网络的网关，即你的主路由器的 IP 地址。
   - `parent` 参数：使用 enp5s0 作为父级网卡。
   - `macnet`：你创建的 macvlan 网络的名称。

   ```
   docker network create -d macvlan --subnet=10.0.0.0/24 --gateway=10.0.0.1 -o parent=enp5s0 macnet
   ```

4. **验证 macvlan 网络:**

   ```
   docker network ls
   ```

   确认列表中有 macnet。

5. **拉取 OpenWrt 镜像:**

   ```
   docker pull registry.cn-hangzhou.aliyuncs.com/zzsrv/openwrt:latest
   ```

6. **创建并运行 OpenWrt 容器:**

   - `--name openwrt`：给容器命名为 openwrt。
   - `-d`：后台运行。
   - `--network macnet`：指定使用 macnet 网络。
   - `--privileged`：给予容器特权权限。
   - `registry.cn-hangzhou.aliyuncs.com/zzsrv/openwrt /sbin/init`：使用的镜像和启动命令。

   ```
   docker run --restart always --name openwrt -d --network macnet --privileged registry.cn-hangzhou.aliyuncs.com/zzsrv/openwrt /sbin/init
   
   docker run --restart always --name openwrt -d --network macnet --privileged zzsrv/openwrt /sbin/init
   ```

   ```
   docker run --restart always --name istoreos -d --network macnet --ip 10.0.0.10 --privileged fingerboy/istoreos-x86-64 /sbin/init
   docker run --restart always --name istoreos -d --network macnet --privileged --dns none fingerboy/istoreos-x86-64 /sbin/init
   
   ```

   

7. **进入 OpenWrt 容器:**

   ```
   docker exec -it openwrt bash
   docker exec -it istoreos bash
   ```

8. **配置 OpenWrt 网络 (关键步骤):**

   - 编辑 `/etc/config/network` 文件。

     ```
     vi /etc/config/network
     ```

     

   - 配置以下内容，确保以下配置中 OpenWrt 的 IP 地址 10.0.0.10 和你主路由器的 IP 地址 10.0.0.1 在同一个网段，且不相同。

     ```
     config interface 'loopback'
         option proto 'static'
         option ipaddr '127.0.0.1'
         option netmask '255.0.0.0'
         option device 'lo'
     
     config globals 'globals'
         option packet_steering '1'
     
     config interface 'lan'
         option proto 'static'
         option netmask '255.255.255.0'
         option ip6assign '60'
         option ipaddr '10.0.0.10'  # 修改为 10.0.0.10
         option gateway '10.0.0.1'  # 修改为 10.0.0.1 (你的主路由 IP)
         option dns '10.0.0.1'   # 修改为 10.0.0.1 (你的主路由 IP)
         option device 'br-lan'    # 保持不变，使用桥接接口 br-lan
     
     config device
         option name 'br-lan'
         option type 'bridge'
         list ports 'eth0'
     ```

     

     - **loopback 接口：** 这是回环接口，配置正确。
     - **globals 配置：** `option packet_steering '1'` 表示开启数据包转向，一般保留。
     - **lan 接口：**
       - `option proto 'static'`：使用静态 IP 地址。
       - `option netmask '255.255.255.0'`：子网掩码。
       - `option ip6assign '60'`：IPv6 地址分配前缀长度。
       - `option ipaddr '192.168.0.8'`：当前 IP 地址，你需要修改为 10.0.0.10。
       - `option gateway '192.168.0.1'`：当前网关，你需要修改为 10.0.0.1 (你的主路由 IP)。
       - `option dns '192.168.0.1'`：当前 DNS，你需要修改为 10.0.0.1 (你的主路由 IP) 或者公共 DNS。
       - `option device 'br-lan'`：使用的接口是 br-lan（桥接接口）。
     - **device 配置：**
       - `option name 'br-lan'`：桥接接口名称。
       - `option type 'bridge'`：接口类型为桥接。
       - `list ports 'eth0'`：桥接接口包含 eth0 接口。

9. **重启 OpenWrt 容器:**

   ```
   etc/init.d/network restart
   exit
   docker container restart openwrt
   ```

   

10. **配置 OpenClash：**

    - 访问 OpenWrt 的 LuCI 管理界面（通常是 http://10.0.0.10，如果访问不了，请检查你电脑的网络设置，确保你电脑的网关也设置为了 10.0.0.1），安装 OpenClash 插件。
    - 配置 OpenClash，使其只负责流量转发，而不是作为 DHCP 服务。
    - 根据你的需求进行相关设置，例如选择节点和代理模式。

11. **配置主路由器或客户端：**

    - **方法一 (路由器配置)：**
      - 在你的主路由器中设置静态路由，将需要走代理的流量（例如特定 IP 地址或域名）指向 OpenWrt 的 IP 地址 10.0.0.10。具体设置方法请参考你主路由器的说明书。
      - 例如，如果你的主路由器支持设置“策略路由”，你可以设置目标网络为你要代理的 IP 或域名，下一跳网关设置为 10.0.0.10。
    - **方法二 (客户端配置):**
      - 对于需要走旁路网关的设备（例如你的电脑），手动设置其网关为 10.0.0.10。
      - 或者，你可以设置 DNS 服务器为 10.0.0.10，并使用 OpenClash 的 DNS 功能进行分流。

**重要注意事项：**

- **IP 地址冲突：** 确保 10.0.0.10 没有被网络中的其他设备占用。
- **防火墙：** 确保宿主机 (Debian) 的防火墙没有阻止 OpenWrt 的网络流量。
- **网络连通性测试：** 完成配置后，尝试在需要走旁路网关的设备上访问外网，检查是否可以通过 OpenWrt 的 OpenClash 进行代理。
- **OpenWrt 访问：** 如果你无法访问 OpenWrt 的 LuCI 管理界面，请检查你的电脑是否和 OpenWrt 在同一个网段，或者电脑的网关是否设置正确。

**配置总结：**

1. 使用独立的物理网卡 enp5s0。
2. 创建 macvlan 网络。
3. 在 OpenWrt 中配置正确的 IP 地址 10.0.0.10、网关 10.0.0.1 和 DNS 10.0.0.1。
4. 在主路由器或者客户端设备上，设置需要走代理的流量指向 OpenWrt 的 IP 地址 10.0.0.10。