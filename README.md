# v2

适合小内存 VPS 的固定三协议代理脚本，无 Web 面板：

| 协议 | 独立端口 |
| --- | --- |
| VLESS+REALITY | 8899/TCP |
| Hysteria 2 | 8900/UDP |
| VLESS+mKCP（预共享 seed） | 8901/UDP |

只保留安装、查看协议、分享链接／二维码、状态与启停、证书续期和 BBR。没有节点新增／修改、客户端管理、管理 API、流量统计或端口跳跃。

## 安装／重新部署

支持 Debian 12+ / Ubuntu 22.04+、systemd、amd64/arm64。系统可见内存至少 384 MiB、可用内存至少 64 MiB、可用磁盘至少 512 MiB。云安全组及本机防火墙放行 TCP 80、8899 和 UDP 8900、8901；本机 19080 仅供 Hy2 接入过滤，不向公网开放。

在 VPS SSH 终端执行（root 可省略 sudo）：

~~~sh
curl -fL https://raw.githubusercontent.com/1790496337/v2/main/install.sh -o /tmp/v2-install.sh && sudo sh /tmp/v2-install.sh
~~~

**每次部署都生成全新随机 UUID、密码、mKCP seed、REALITY 密钥与 short ID，旧链接失效，需重新导入。** 不迁移或备份旧节点凭据。仍有效且匹配当前 IP 的证书继续使用，避免重复申请。

安装器只打包 Python 标准库管理程序，从官方仓库下载并校验 Xray、Hysteria 2 和 acme.sh；不捆绑代理二进制或地理数据库。安装前检查系统资源、端口。只自动处理服务名和程序路径都能确认的冲突旧代理（x-ui/3x-ui、Xray、V2Ray、Hysteria）；第三方程序备份在 `/var/backups/v2/legacy-*`，未知冲突会停止部署并提示。

覆盖安装自动停用并删除旧版统计、端口跳跃服务，删除旧统计／事务文件和 v2 专用 nftables 表，停用旧 Hy2 实例，再部署固定三协议。不保留旧节点凭据和统计，不改动其他业务服务或防火墙表；保留证书及 ACME 账户以便续期。

项目已从 LiteProxy 更名为 `v2`：命令为 `v2`，配置目录 `/etc/v2`，程序目录 `/usr/local/lib/v2`，服务前缀 `v2-`。覆盖旧 LiteProxy 时，会停用并删除旧服务，迁移证书和 ACME 账户、更新续期路径；新版启动成功后删除旧命令与安装目录。

## 使用与分享

运行 `sudo v2` 打开中文菜单，或直接执行：

~~~sh
sudo v2 info
sudo v2 url
sudo v2 url mkcp
sudo v2 qr
sudo v2 qr hy2
sudo v2 qr mkcp --output /root/mkcp.png
sudo v2 status
sudo v2 restart
sudo v2 stop
sudo v2 start
sudo v2 cert status
sudo v2 log hy2
sudo v2 test
~~~

`url` 输出完整纯链接，每个协议一行，别名为 `IP_协议名称`；`url --base64` 可输出 Base64 链接列表。`qr` 默认显示 REALITY 二维码，可指定 `reality`、`hy2` 或 `mkcp`。

手机使用 v2rayNG／Shadowrocket 扫一扫。二维码由 `qrencode` 在 VPS 本地生成，不上传在线网站。终端使用等宽字体、避免折行；显示不全可保存 PNG 后通过 SFTP 下载，图片权限为 600，含连接凭据。剪贴板导入请只复制完整单行链接；之前的手机剪贴板问题尚未实机定位，可使用扫码。

## 长期运行

- 安装后自动启动、开机自启；核心退出后等待 10 秒重启。手动 stop 保持停止，直到 start 或下次开机。自动重启不检测进程卡住或公网不可达。
- IP 证书约六天有效，每小时检查续期，需公网 IP 稳定、TCP 80 可访问。续期验证证书后切换，只重启正在运行的 Hy2。
- TCP 使用内核支持的 BBR+fq；Hy2 使用 QUIC BBR，mKCP 使用自身拥塞控制。性能取决于网络，不保证提速。
- Xray 硬内存上限 192 MiB，Hy2 为 128 MiB，需 memory cgroup；超限可能重启并中断连接，上限不保证整机不会 OOM。
- 保留 BT／私网过滤，Hy2 经本机认证 SOCKS 接口应用 Xray 过滤；不能保证识别所有加密 BT。

~~~sh
sudo systemctl list-timers --all 'v2-*'
sudo journalctl -u v2-renew.service -n 30 --no-pager
~~~

为兼容传统 mKCP seed/header，Xray 固定 25.8.3，未包含后续修复；不能直接替换最新版而沿用旧配置。REALITY 在部署时选择通过 TLS 1.3、HTTP/2 和证书校验的低延迟候选目标。IP 证书跨期续期和 512 MB 压力测试尚未完成。

GitHub 仅发布 `install.sh` 与 `README.md`。
