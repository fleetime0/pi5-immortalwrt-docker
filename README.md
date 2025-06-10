# pi5-immortalwrt-docker
在树莓派 5 上通过 Docker 部署 ImmortalWRT 系统。

## 部署步骤

### 1. 创建 macvlan 网络

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan_net
```

注意，--subnet的IP地址需要修改为你当前局域网网段，--gateway需要指向实际路由器的IP，-o parent 指向物理网卡名称。

## 2. 宿主机和docker通信

> 默认 macvlan 宿主机无法访问该网段，通过添加 macvlan 接口解决

```bash
sudo ip link add macvlan0 link eth0 type macvlan mode bridge
sudo ip addr add 192.168.1.2/24 dev macvlan0
sudo ip link set macvlan0 up
```

## 3. 启动容器运行 ImmortalWRT

```bash
docker run -d --name immortalwrt \
  --network macvlan_net \
  --privileged \
  --device /dev/net/tun \
  --restart unless-stopped \
  ghcr.io/fleetime0/immortalwrt-pi5 /sbin/init
```

> /sbin/init 确保系统以 init 方式完整启动，支持系统服务等。
