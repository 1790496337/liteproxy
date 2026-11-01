# LiteProxy

适合小内存 VPS 的轻量命令行代理管理器，无 Web 面板。

默认安装 VLESS+REALITY（8899/TCP）、Hysteria 2（8899/UDP）、VLESS+mKCP（8900/UDP）。自动识别公网 IPv4，为 Hy2 申请短期 IP 证书并定期续期；安装后自动启动、开机自启，进程退出后等待 10 秒自动重启。

支持 Debian 12+ / Ubuntu 22.04+、systemd、amd64/arm64。放行 TCP 80、TCP/UDP 8899–8999；VPS 本机防火墙也需允许这些端口。

## 安装

在 VPS 的 SSH 终端执行以下一条命令：

~~~sh
curl -fL https://raw.githubusercontent.com/1790496337/liteproxy/main/install.sh -o /tmp/liteproxy-install.sh && sudo sh /tmp/liteproxy-install.sh
~~~

安装文件只包含轻量管理程序。运行后从官方仓库下载并校验 Xray、Hysteria 2、acme.sh；没有内嵌代理二进制。

每次安装、重装及 start/restart 先检查系统、CPU、内存、磁盘和所需端口（含 TCP 80、本机 API）。至少需 384 MiB 系统可见内存、512 MiB 可用磁盘、64 MiB 部署可用内存。

占用所需端口的旧代理，只有服务名和程序路径都能确认是 x-ui/3x-ui、Xray、V2Ray、Hysteria 时才会自动停用、备份并卸载程序。备份位于 `/var/backups/liteproxy/legacy-*`，含原路径和服务状态；原配置、证书、客户端资料保留。清理失败会尝试恢复原服务；后续新部署失败时备份保留。包管理器单元、额外启动触发器、符号链接及未知业务服务需手动处理。

重装保留 LiteProxy 已有节点、密钥、证书和统计；查看状态、流量、链接不会触发卸载。

## 管理

导出的链接别名为 `服务器IP_VLESS+mKCP`、`服务器IP_VLESS+REALITY`、`服务器IP_Hysteria2`。

~~~sh
sudo liteproxy
sudo liteproxy info
sudo liteproxy client add reality alice
sudo liteproxy url
sudo liteproxy url mkcp default
sudo liteproxy url --base64
sudo liteproxy qr
sudo liteproxy qr mkcp
sudo liteproxy qr hy2 default
sudo liteproxy qr reality default --output /root/reality.png
sudo liteproxy stats
sudo liteproxy reality scan --apply
sudo liteproxy cert status
sudo liteproxy restart
~~~

菜单提供入站、客户端、流量统计、凭据修改、REALITY 测速、证书管理和 BBR 调优。IP 证书自动续期需要保持 TCP 80 可从公网访问。手动 stop 会保持停止，直到 start 或下次开机。

分享提供两种方式：菜单 **6 分享链接**、**13 二维码分享**。`url` 可一次导出所有链接，`qr [入站名称] [客户端名称]` 每次显示一个客户端的二维码，默认 `reality default`。可用入站名称为 `mkcp`、`reality`、`hy2`（自建入站使用其实际名称），客户端默认 `default`。手机使用 v2rayNG 或 Shadowrocket 的扫一扫导入。

二维码由系统软件源提供的 `qrencode` 在 VPS 本地生成，无后台进程，不调用在线二维码网站；二维码内容与对应分享链接相同。终端请保持等宽字体、不自动折行，显示不完整可缩小字体或使用 `--output /root/node.png` 保存 PNG，再通过 SFTP 下载查看。图片以 600 权限保存，内含节点凭据，按需分享。老版本直接执行 `liteproxy install` 不会安装系统依赖，需重跑一键脚本或安装 `sudo apt-get install -y qrencode`。

`url` 仅输出纯链接，每个节点一行，方便从剪贴板导入。v2rayNG 使用右上角“+ → 从剪贴板导入”；Shadowrocket 使用首页的剪贴板导入提示。请勿使用“添加订阅地址”导入单节点链接，也不要复制终端说明、Markdown 包装、`mailto:` 或 `vless\://` 中的反斜杠。聊天工具改写链接时，可用 `url --base64` 导出整段 Base64；v2rayNG 1.10.23 的批量导入代码支持该形式，其他客户端需确认支持后使用。

已核对 v2rayNG 1.10.23 官方解析代码中的 VLESS KCP/seed、REALITY 和 Hy2 参数；尚未在 Android/iOS 实机验证。客户端能导入不代表其内核支持服务端的传统 mKCP seed/header，需要单独验证连接。

v2rayNG 1.10.23 批量导入按行匹配协议前缀，不会先清理行首空格；旧安装摘要中链接前的两个空格会导致识别失败。现在安装摘要、新增节点和 `url` 输出中的链接均从行首开始。遇到“成功导入 0 个配置”，先用纯链接或 Base64 导出重试；仍失败时检查当前订阅分组的正则过滤规则，并提供具体操作步骤。

端口规则恢复遇到其他管理任务时最多等待 600 秒，获得锁后再执行；等待超时或规则执行失败会明确报错，安装器自动显示最近的服务日志。

自动续期同样最多等锁 600 秒，定时统计最多等待 30 秒；失败会记录到对应 systemd 服务日志。统计每五分钟落盘，采样失败或采样期间进程重启时保留上次数据。突然断电、进程崩溃仍可能损失最后一个采样周期的流量，统计不用于精确计费。

为限制异常内存增长，Xray 的 MemoryMax 为 192 MiB，每个 Hy2 实例为 128 MiB；超过上限可能由内核终止并由 systemd 重启，连接会中断。需要内核启用 memory cgroup；多个 Hy2 实例各有独立限额，512 MB 机器建议保留默认一个实例。这些上限不保证整机不会 OOM，也不代表可承载连接数。续期只重启当时运行的 Hy2，手动停止的实例保持停止。

日常检查：

~~~sh
sudo liteproxy status
sudo liteproxy test
sudo systemctl list-timers --all 'liteproxy-*'
sudo journalctl -u liteproxy-renew.service -n 30 --no-pager
~~~

## 当前限制

- 为兼容传统 mKCP seed/header，固定 Xray 25.8.3；旧版本不含后续修复，不能直接替换为最新版而保留旧 mKCP 配置。
- REALITY 选择当次探测中支持 TLS 1.3、HTTP/2 且证书有效的低延迟目标，不保证目标始终适用。
- mKCP seed 使用预共享密钥封装，区别于 TLS 证书认证。BT 过滤无法保证识别加密 BT。
- 已进行本地配置和数据传输验证，用户已反馈 VPS 安装运行正常；真实 IP 证书跨期续期及 512 MB 压力测试尚未完成。
- 自动重启处理进程退出，不检测进程卡住或公网不可达，也没有外部故障通知；需要保持 IP 稳定、TCP 80 开放并检查续期日志。

维护时每次修改都会重新构建并验证安装脚本，再同步本仓库的 install.sh 和 README.md。
