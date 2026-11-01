# LiteProxy

适合小内存 VPS 的命令行代理管理器，无 Web 面板。默认部署：

| 协议 | 端口 |
| --- | --- |
| VLESS+REALITY | 8899/TCP |
| Hysteria 2 | 8899/UDP |
| VLESS+mKCP（预共享 seed） | 8900/UDP |

自动选择低延迟 REALITY 目标、申请及续期 IP 证书，支持 BBR、入站与客户端管理、流量统计、链接和二维码分享。安装后自动启动、开机自启，进程退出后等待 10 秒重启。

## 安装与更新

支持 Debian 12+ / Ubuntu 22.04+、systemd、amd64/arm64。系统可见内存至少 384 MiB，可用内存至少 64 MiB，可用磁盘至少 512 MiB。云安全组和本机防火墙需放行 TCP 80、TCP/UDP 8899–8999。

在 VPS 的 SSH 终端执行，普通账户加 sudo，root 可省略 sudo：

~~~sh
curl -fL https://raw.githubusercontent.com/1790496337/liteproxy/main/install.sh -o /tmp/liteproxy-install.sh && sudo sh /tmp/liteproxy-install.sh
~~~

重跑同一命令更新管理器，保留已有节点、密钥、证书和统计。安装器只包含轻量管理程序；首次部署从官方仓库下载并校验 Xray、Hysteria 2、acme.sh。

安装及 start/restart 会检查资源与端口。仅自动处理服务名和程序路径都能确认的冲突旧代理（x-ui/3x-ui、Xray、V2Ray、Hysteria），备份至 `/var/backups/liteproxy/legacy-*` 后停用、卸载程序，保留原配置、证书及客户端资料。未知服务或无法安全清理的冲突会停止部署并提示处理；查询信息不会触发卸载。

## 管理与分享

执行 `sudo liteproxy` 打开中文菜单，也可直接使用命令：

~~~sh
sudo liteproxy info
sudo liteproxy client add reality alice
sudo liteproxy stats
sudo liteproxy reality scan --apply
sudo liteproxy cert status
sudo liteproxy restart
~~~

更多参数执行 `sudo liteproxy help`。支持 v2ray 命令别名（不覆盖已有命令）。手动 stop 后保持停止，直到 start 或下次开机。

菜单 **6 分享链接**、**13 二维码分享**；链接别名为 `IP_协议名称`。命令中的入站默认可选 `mkcp`、`reality`、`hy2`，客户端默认 `default`：

~~~sh
sudo liteproxy url                     # 所有纯链接，每个一行
sudo liteproxy url mkcp default        # 单个客户端链接
sudo liteproxy url --base64            # Base64 链接列表
sudo liteproxy qr                      # 默认 reality/default 二维码
sudo liteproxy qr mkcp                 # mKCP 二维码
sudo liteproxy qr hy2 default --output /root/hy2.png
~~~

手机使用 v2rayNG 或 Shadowrocket 扫一扫导入；剪贴板导入请只复制完整单行链接。若提示导入 0 个配置，可用二维码；当前剪贴板问题尚未在手机实机定位。

二维码由 `qrencode` 在 VPS 本地生成，无后台服务或在线上传。终端使用等宽字体，避免自动折行；显示不全时保存 PNG，通过 SFTP 下载。PNG 权限为 600，含连接凭据，请按需分享。

## 运行维护

- IP 证书约六天有效，续期任务每小时检查；需保持公网 IP 稳定、TCP 80 可访问。续期只重启当时运行的 Hy2 实例。
- 流量每五分钟保存，采样失败保留上次数据；突然断电或进程崩溃可能损失最近一周期的数据，不能用于精确计费。
- Xray 内存硬上限 192 MiB，每个 Hy2 实例 128 MiB（需要 memory cgroup）；超限会终止重启并中断连接。512 MB VPS 建议只保留默认三个入站，上限不保证整机不会 OOM。
- 自动重启处理进程退出，不检测进程卡住或公网不可达。

~~~sh
sudo liteproxy status
sudo liteproxy test
sudo systemctl list-timers --all 'liteproxy-*'
sudo journalctl -u liteproxy-renew.service -n 30 --no-pager
~~~

## 兼容性

- 为兼容传统 mKCP seed/header，固定 Xray 25.8.3，未包含后续版本修复；不能直接升级内核而保留旧 mKCP 配置。
- REALITY 目标按当次 TLS 1.3、HTTP/2 与有效证书探测择优，目标后续可能变化。mKCP seed 区别于 TLS 证书认证；BT 过滤无法保证识别加密 BT。
- 已有 VPS 安装运行反馈和本地数据转发验证；IP 证书跨期续期及 512 MB 压力测试尚未完成，导入成功后仍需验证客户端内核能否连接。

本仓库仅发布 `install.sh` 与 `README.md`；更新前重新构建、验证安装包。
