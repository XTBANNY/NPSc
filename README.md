# NPSc

> 一个基于多种内核的多协议代理节点服务端，功能完备，配置灵活。

[![Release](https://img.shields.io/github/v/release/XTBANNY/NPSc)](https://github.com/XTBANNY/NPSc/releases)
[![License](https://img.shields.io/github/license/XTBANNY/NPSc)](LICENSE)

## 简介

NPSc 是一个高性能的多协议代理节点服务端，支持 Vmess/Vless、Trojan、Shadowsocks、Hysteria1/2 等多种协议，可同时运行多种内核（sing-box、Xray、Hysteria2），支持单实例对接多节点，无需重复启动。

**注意：本项目需要搭配兼容 V2board 的后端面板使用。**

## 核心特性

| 特性 | 说明 |
|------|------|
| 多协议支持 | Vmess、Vless、Trojan、Shadowsocks、Hysteria1/2 |
| 多内核 | 支持 sing-box、Xray、Hysteria2 同时运行 |
| 单实例多节点 | 一次启动管理多个节点 |
| 在线 IP 限制 | 支持单节点 / 跨节点 IP 数限制 |
| 连接数限制 | 限制 TCP 连接数 |
| 用户限速 | 节点端口级别 + 用户级别限速 |
| TLS 自动证书 | 自动申请、自动续签 |
| 审计规则 | 支持访问审计 |
| 自定义 DNS | 独立配置 DNS 解析 |
| 配置热重载 | 修改配置文件自动重启实例 |

## 功能对照

| 功能 | v2ray | trojan | shadowsocks | hysteria1/2 |
|------|-------|--------|-------------|-------------|
| 自动申请 TLS 证书 | ✓ | ✓ | ✓ | ✓ |
| 自动续签 TLS 证书 | ✓ | ✓ | ✓ | ✓ |
| 在线人数统计 | ✓ | ✓ | ✓ | ✓ |
| 审计规则 | ✓ | ✓ | ✓ | ✓ |
| 自定义 DNS | ✓ | ✓ | ✓ | ✓ |
| 在线 IP 数限制 | ✓ | ✓ | ✓ | ✓ |
| 连接数限制 | ✓ | ✓ | ✓ | ✓ |
| 跨节点 IP 数限制 | ✓ | ✓ | ✓ | ✓ |
| 用户级别限速 | ✓ | ✓ | ✓ | ✓ |
| 动态限速 | ✓ | ✓ | ✓ | ✓ |

## 一键安装（推荐）

### 全自动安装

```bash
wget -N https://raw.githubusercontent.com/XTBANNY/NPSc-script/master/install.sh && bash install.sh
```

安装后会自动进入交互式菜单，按提示选择即可：

1. 选择是否需要自动配置（输入 `y`）
2. 输入面板地址
3. 输入 API Key
4. 输入节点 ID
5. 选择节点类型和协议
6. 配置 TLS 证书（如需要）

完成后 NPSc 将自动启动，并设置开机自启。

### 管理脚本

安装后可使用 `NPSc` 或 `npsc` 命令进行管理：

```bash
NPSc              # 显示交互菜单
NPSc start        # 启动
NPSc stop         # 停止
NPSc restart      # 重启
NPSc status       # 查看状态
NPSc log          # 查看日志
NPSc enable       # 开机自启
NPSc disable      # 取消开机自启
NPSc generate     # 交互式生成配置文件
NPSc update       # 更新
NPSc uninstall    # 卸载
```

### 手动安装

1. 从 [Releases](https://github.com/XTBANNY/NPSc/releases) 下载 `NPSc-linux-64.zip`

2. 解压并安装（注意：Release 包是 zip 格式，内层有 NPSc/ 子目录）：

```bash
unzip NPSc-linux-64.zip
mv NPSc/NPSc /usr/local/bin/NPSc
chmod +x /usr/local/bin/NPSc
mkdir -p /etc/NPSc/
cp NPSc/*.json /etc/NPSc/
cp NPSc/*.dat /etc/NPSc/ 2>/dev/null
cp NPSc/*.db /etc/NPSc/ 2>/dev/null
rm -rf NPSc
```

3. 如果 Release 包不含 geoip/geosite，手动下载：

```bash
wget -O /etc/NPSc/geoip.dat https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/geoip.dat
wget -O /etc/NPSc/geosite.dat https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/geosite.dat
```

4. 如果使用 sing 核心，创建 `sing_origin.json`：

```bash
cat > /etc/NPSc/sing_origin.json << 'EOF'
{
  "dns": {
    "servers": [{"tag": "cf", "address": "1.1.1.1"}],
    "strategy": "ipv4_only"
  },
  "outbounds": [
    {"tag": "direct", "type": "direct"},
    {"type": "block", "tag": "block"}
  ],
  "route": {
    "rules": [
      {"ip_is_private": true, "outbound": "block"},
      {"outbound": "direct", "network": ["udp","tcp"]}
    ]
  }
}
EOF
```

5. 编辑配置文件 `/etc/NPSc/config.json`，修改 ApiHost、ApiKey、NodeID、NodeType 等

6. 创建 systemd 服务并启动：

```bash
cat > /etc/systemd/system/NPSc.service << 'EOF'
[Unit]
Description=NPSc Service
After=network.target nss-lookup.target
Wants=network.target

[Service]
User=root
Group=root
Type=simple
LimitNOFILE=65535
WorkingDirectory=/etc/NPSc/
ExecStart=/usr/local/bin/NPSc server --config /etc/NPSc/config.json
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable NPSc
systemctl start NPSc
```

7. 验证运行：

```bash
systemctl status NPSc
journalctl -u NPSc -f
```

> **常见错误排查**：
> - `open sing_origin.json: no such file` → 执行步骤4
> - `Failed to load GeoIP: geoip.dat` → 执行步骤3
> - `Is a directory` → 用 `mv NPSc/NPSc /usr/local/bin/NPSc`（注意嵌套目录）
> - `Load config file failed` → 检查 JSON 格式


## 编译安装

确保已安装 [Go 1.25](https://go.dev/) 或更高版本：

```bash
git clone https://github.com/XTBANNY/NPSc.git
cd NPSc

# 编译所有内核（推荐）
GOEXPERIMENT=jsonv2 go build -o NPSc \
  -tags "sing xray hysteria2 with_quic with_grpc with_utls with_wireguard with_acme with_gvisor" \
  -trimpath -ldflags "-s -w -buildid="

# 仅编译 sing-box 内核（更小体积）
GOEXPERIMENT=jsonv2 go build -o NPSc -tags "sing" -trimpath -ldflags "-s -w"
```

## 配置文件说明

### 基本结构

```json
{
  "Log": {
    "Level": "error",
    "Output": "/var/log/npsc.log"
  },
  "Cores": [
    {
      "Type": "sing",
      "Log": { "Level": "error" }
    },
    {
      "Type": "xray",
      "Log": { "Level": "error" }
    }
  ],
  "Nodes": [
    {
      "Core": "sing",
      "ApiHost": "https://your-panel.com",
      "ApiKey": "your-api-key-here",
      "NodeID": 1,
      "NodeType": "vless",
      "Timeout": 30,
      "ListenIP": "0.0.0.0",
      "CertConfig": {
        "CertMode": "http",
        "CertDomain": "your-domain.com"
      }
    }
  ]
}
```

### 配置项说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `Core` | ✓ | 内核类型：`sing` / `xray` / `hysteria2` |
| `ApiHost` | ✓ | 面板地址，如 `https://example.com` |
| `ApiKey` | ✓ | 面板 API Token |
| `NodeID` | ✓ | 节点在面板中的 ID |
| `NodeType` | ✓ | 协议类型：`vless` / `vmess` / `trojan` / `shadowsocks` / `hysteria2` |
| `Timeout` | - | API 超时时间（秒） |
| `ListenIP` | - | 监听 IP |
| `CertConfig.CertMode` | - | 证书模式：`http` / `dns` / `self` / `none` |
| `CertConfig.CertDomain` | - | 证书域名 |

## 构建说明

支持通过 Go build tags 选择编译的内核：

```bash
# 仅 sing-box (~15MB)
GOEXPERIMENT=jsonv2 go build -tags "sing" -o NPSc

# 仅 Xray (~20MB)
GOEXPERIMENT=jsonv2 go build -tags "xray" -o NPSc

# 仅 Hysteria2 (~10MB)
GOEXPERIMENT=jsonv2 go build -tags "hysteria2" -o NPSc

# 全部内核 (~40MB)
GOEXPERIMENT=jsonv2 go build -tags "sing xray hysteria2 with_quic with_grpc with_utls with_wireguard with_acme with_gvisor" -o NPSc
```

## 常见问题

**Q: 启动报错 "Load config file failed"**

检查配置文件路径是否正确，以及 JSON 格式是否合法。

**Q: 配置文件修改后会自动生效吗？**

是的，默认会监听配置文件变化，修改后自动重载。

**Q: 日志在哪里？**

默认输出到 stdout。可在配置文件的 `Log.Output` 指定日志文件路径。

**Q: 如何放行端口？**

如果使用 firewalld/ufw，请手动放行节点监听端口。或使用一键脚本菜单中的「放行所有端口」功能。

## 免责声明

- 本项目仅供学习和研究使用，请勿用于非法用途。
- 作者不对任何人使用本项目造成的任何后果承担责任。

## 许可证

MPL-2.0
