TOW
===

Transparent Over the Wall for Tomato/OpenWRT Router 

# TOW (	Transparent Over the Wall )

TOW 是一个安装在 Tomato/OpenWRT 系统上的软件包，安装之后，可以保证连接在这个路由器上的所有客户端透明翻墙。

当前版本：1.0 [CHANGELOG]

## 功能

TOW 的设计目标是透明化/自动化，理想情况下客户端用户无需关心哪些网站无法访问，可直连网站也不会因为使用二级代理而降低访问速度。

- 使用 pdnsd 特性防止 DNS 污染
- 支持 GoAgent, SOCKS5, [shadowsocks](https://github.com/clowwindy/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E) 和 Obfuscated ssh 等代理服务器
- 使用 gfwlist 和 ipset 配合 iptables 处理被墙网站，仅对被墙网站使用代理

# 依赖

- 一台能安装 Tomato 或者 OpenWRT 的路由器
- Tomato/OpenWRT 内含的 iptables/dnsmasq 必须编译支持 ipset

安装：(均以 Tomato 为例）

准备好 Putty 和 WinSCP 工具；


如果 Flash 空间足够，启用 JFFS ，格式化后，SSH 登陆路由器：

代码:
```sh
cd /jffs
mkdir opt
mount -o bind /jffs/opt /opt
```
下载软件包，tow-1.0.tar.gz，用 WinSCP 上传到路由器 tmp 目录，然后执行：

代码:
```sh
cd /
tar xvzf /tmp/tow-kms-1.0-jffs-final.tar.gz
```
如果 Flash 空间不够，请挂载 U 盘并且在 U 盘上创建 opt 目录，步骤可 Google，例如：

代码:
```sh
mount -o bind /mnt/sda1/opt /opt
cd /
tar xvzf /tmp/opt-optware-backup.tar.gz
cp /opt/.autorun /mnt/sda1/   ###注意这一步不要漏掉，自启动脚本必须放在挂载点根目录
```
请确保 U 盘挂载的分区格式为 ext2/ext3/ext4，optware 不能安装运行在 ntfs 格式上！

pdnsd 内置 114 DNS 服务器解析本地域名，如果对速度不满意，可以修改 /opt/etc/pdnsd.conf 文件第 27 行：

代码:
```
	ip = 114.114.114.114,114.114.115.115;
```
改为你本地 ISP 的 DNS 服务器地址。

路由器Web界面设置：

1.高级设置-DHCP/DNS-自定义设置：

代码:
```
conf-dir=/opt/etc/dnsmasq/custom/
```
2. 系统管理-JFFS-挂载后执行：

代码:
```
mount -o bind /jffs/opt /opt
```
如果使用 U 盘，则在：USB and NAS-USB 支持-挂载后运行：

代码:
```
mount -o bind /mnt/sda1/opt /opt
```
3. 系统管理-定时重启/连接-自定义(每12小时）：

代码:
```
/opt/etc/init.d/S10dnsmasqc
```

重启路由器

再次 SSH 登入路由器，运行：

代码:
```sh
netstat -lnp
```

应该可以看到 tcp 有 8099 和 7070 端口在监听；udp 有 5454 在监听；

运行：

代码:
```sh
iptables -t nat -nvL
```

应该看到有 REDSOCKS 或者 SHADOWSOCKS 的 Chian 存在。

给出的软件包默认工作在 `ShadowSocks Redir` + 全子网模式，也就是在路由器子网内所有客户端透明翻墙。

请修改： `/opt/etc/py/ga/proxy.ini` 填入你自己的 GoAgent 账号。

请修改：`/opt/etc/shadowsocks.json` 文件，填入你自己的服务器地址端口和密码，本地监听端口7070不可改！

# 致谢

贡献代码：

- autogfwlist 维护的黑名单；
- lantern 维护的黑名单；
- n0wa11 维护的白名单；
- panda 维护的白名单；
- semigodking 修改的 redsocks2；
- OpenWRT RA-MOD；
- fqdns/西厢3的 dns 钓鱼脚本；

