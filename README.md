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

导出的链接别名为 `Q_VLESS+mKCP`、`Q_VLESS+REALITY`、`Q_Hysteria2`。

~~~sh
sudo liteproxy
sudo liteproxy info
sudo liteproxy client add reality alice
sudo liteproxy url
sudo liteproxy stats
sudo liteproxy reality scan --apply
sudo liteproxy cert status
sudo liteproxy restart
~~~

菜单提供入站、客户端、流量统计、凭据修改、REALITY 测速、证书管理和 BBR 调优。IP 证书自动续期需要保持 TCP 80 可从公网访问。手动 stop 会保持停止，直到 start 或下次开机。

端口规则恢复遇到其他管理任务时最多等待 600 秒，获得锁后再执行；等待超时或规则执行失败会明确报错，安装器自动显示最近的服务日志。

## 当前限制

- 为兼容传统 mKCP seed/header，固定 Xray 25.8.3；旧版本不含后续修复，不能直接替换为最新版而保留旧 mKCP 配置。
- REALITY 选择当次探测中支持 TLS 1.3、HTTP/2 且证书有效的低延迟目标，不保证目标始终适用。
- mKCP seed 使用预共享密钥封装，区别于 TLS 证书认证。BT 过滤无法保证识别加密 BT。
- 已进行本地配置和数据传输验证；Linux 安装、真实 IP 证书跨期续期及 512 MB 压力测试尚未在真实 VPS 完成。

维护时每次修改都会重新构建并验证安装脚本，再同步本仓库的 install.sh 和 README.md。
