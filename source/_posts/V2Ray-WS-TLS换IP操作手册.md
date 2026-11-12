---
title: V2Ray WS+TLS 换 IP 操作手册
date: 2026-11-10
categories: 运维
tags:
    - v2ray
    - nginx
    - websocket
    - tls
    - 运维
#description: 服务器公网 IP 被墙后，从诊断、切换出口 IP、改 DNS 到验证与排坑的完整手册（含 nginx http2 导致 WS 握手失败的坑）。
---

记录一次完整的排障与切换过程：从「IP 被墙」的判断、多网卡出口 IP 的确认，到 netplan 改默认路由、DNS 换向，以及回滚用的一键脚本。文中的 UUID、密码等凭据已做脱敏，实际使用时替换成自己的值。

<!-- more -->

> 用途：当服务器公网 IP 被墙后，快速切换到新 IP 并恢复服务。
> 最近更新：2026-11-10（首次搭建 + 从 `.138` 切到 `.207` 实战验证）

## 零、当前架构速览

```
客户端 --443/TLS/WS(/v2raypath)--> nginx --127.0.0.1:10086--> v2ray
                                      └─ 假网页 /var/www/moneyfreedom.cc.cd

旧通道（保留，未动）:
  Shadowsocks  0.0.0.0:8899
  VMess TCP+HTTP头 0.0.0.0:8443
```

| 项目 | 值 |
|---|---|
| 域名 | `moneyfreedom.cc.cd` |
| 当前公网 IP | `141.148.158.207` |
| 私网 IP（静态，对应 .207） | `10.0.0.38` |
| 私网 IP（DHCP，对应 .138 被墙） | `10.0.0.79` |
| VMess UUID | `<YOUR-UUID>` |
| WS 路径 | `/v2raypath` |
| 内部端口 | `10086`（仅 127.0.0.1） |
| 证书 | `/etc/ssl/v2ray/moneyfreedom.crt`（ZeroSSL，至 2027-02-07） |
| DNS 服务商 | dnshe.org（`a.ns.dnshe.org` / `b.ns.dnshe.org`） |

## 一、换 IP 时的判断步骤（先诊断，别瞎改）

### 1.1 确认是 IP 被墙，而不是配置坏了

```bash
# 服务端自测（应该通）
curl -sI --max-time 10 https://moneyfreedom.cc.cd

# WS 握手（应该返回 101）
curl -sv --max-time 10 \
  -H "Connection: Upgrade" -H "Upgrade: websocket" \
  -H "Sec-WebSocket-Version: 13" \
  -H "Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==" \
  "https://moneyfreedom.cc.cd/v2raypath"
```

**服务端全通 + 客户端超时 ⇒ 大概率被墙。**

### 1.2 用日志确认客户端到底到没到服务器

```bash
# 客户端真实 IP 是否出现在 nginx 日志
sudo grep "<客户端IP>" /var/log/nginx/access.log | tail

# v2ray 日志：能看到 reject/timeout 说明 TCP 到了但数据被掐
sudo grep "<客户端IP>" /var/log/v2ray/access.log | tail -20
```

现象对照：

| 现象 | 含义 |
|---|---|
| nginx 日志 0 条该 IP 的 443 请求 | **443 完全不通，IP 被封** |
| v2ray 日志出现 `failed to read ~ header ... i/o timeout` | TCP 通了但数据被干扰（QoS） |
| 有正常 `accepted tcp:...` | 链路健康 |

### 1.3 检查服务器有几个 IP（本机实战坑）

```bash
ip -4 addr show
ip route show default

# 逐个私网 IP 测出口公网地址（关键！映射可能和你想的相反）
curl -s --interface 10.0.0.38 https://api.ipify.org
curl -s --interface 10.0.0.79 https://api.ipify.org
```

> **2026-11-10 实战教训**：本机有两个私网 IP，映射是**反的**：
>
> - `10.0.0.38` → `141.148.158.207`（可用）
> - `10.0.0.79` → `161.153.24.138`（被墙）
>
> 而 DHCP 给的默认路由偏偏走 `.79`，导致出网全走被墙 IP。

## 二、换 IP 的两种场景

### 场景 A：公网 IP 换了，但服务器私网 IP 不变

一般只需改 DNS：

1. 域名后台把 `moneyfreedom.cc.cd` 的 A 记录改成新公网 IP；
2. 等生效：`dig +short @a.ns.dnshe.org moneyfreedom.cc.cd`；
3. 验证第四节。

**v2ray / nginx 都不用改**（都监听 0.0.0.0）。

### 场景 B：服务器有多个 IP，需要切换出网 IP（本次的情况）

#### B-1. 找出哪个私网 IP 对应可用的公网

```bash
curl -s --interface <私网IP> https://api.ipify.org
```

#### B-2. 修改 netplan，让静态 IP 的默认路由优先

文件：`/etc/netplan/60-cloud-init.yaml`

```yaml
network:
  version: 2
  ethernets:
    enp0s6:
      addresses:
        - 10.0.0.38/24
      routes:
        - to: default
          via: 10.0.0.1
          metric: 50
```

> `metric` 越小优先级越高。DHCP 路由默认 metric 100，写 50 就能压过它。
> **网关地址要换成你机器实际的**（`ip route show default` 里看）。

#### B-3. 应用并验证（先备份！）

```bash
sudo cp /etc/netplan/60-cloud-init.yaml /etc/netplan/60-cloud-init.yaml.bak-$(date +%Y%m%d-%H%M)
sudo netplan apply
ip route show default           # 应看到 proto static metric 50 在最前
curl -s https://api.ipify.org   # 应为新公网 IP
```

> ⚠️ **别用 `netplan try`**：它需要交互按回车，非交互环境会超时自动回滚。
> ⚠️ 改网络前确认 SSH 不会断（同网段操作，风险低，但要有心理准备）。

#### B-4. 改 DNS 指向新 IP

同场景 A 第 1-2 步。

## 三、从零重建整套服务（IP 彻底换了 / 新机器）

### 3.1 v2ray WS 入站

文件：`/etc/v2ray/conf/VMess-WS-10086.json`

```json
{
  "inbounds": [
    {
      "tag": "VMess-WS-10086.json",
      "port": 10086,
      "listen": "127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [{ "id": "<YOUR-UUID>" }]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": { "path": "/v2raypath" }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    }
  ]
}
```

校验并重启：

```bash
sudo /etc/v2ray/bin/v2ray test -c /etc/v2ray/config.json -d /etc/v2ray/conf
sudo systemctl restart v2ray
ss -tlnp | grep 10086
```

### 3.2 nginx 反代

文件：`/etc/nginx/conf.d/moneyfreedom.conf`

```nginx
server {
    listen 80;
    server_name moneyfreedom.cc.cd;

    location /.well-known/acme-challenge/ {
        root /var/www/moneyfreedom.cc.cd;
    }
    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name moneyfreedom.cc.cd;

    ssl_certificate     /etc/ssl/v2ray/moneyfreedom.crt;
    ssl_certificate_key /etc/ssl/v2ray/moneyfreedom.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        root /var/www/moneyfreedom.cc.cd;
        index index.html;
    }

    location /v2raypath {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:10086;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

应用：

```bash
sudo /usr/sbin/nginx -t && sudo systemctl reload nginx
```

### 3.3 假网页（防探测）

文件：`/var/www/moneyfreedom.cc.cd/index.html`，随便放个正常页面，`chown www-data:www-data`。

### 3.4 客户端节点

```
地址:   moneyfreedom.cc.cd
端口:   443
协议:   VMess
UUID:   <YOUR-UUID>
传输:   ws
路径:   /v2raypath
TLS:    开
SNI:    moneyfreedom.cc.cd
Host:   moneyfreedom.cc.cd
alterId: 0
加密:   auto
```

vmess 链接（用上面的字段生成即可，形如）：

```
vmess://<base64(JSON 节点信息)>
```

> 小提示：`{v, ps, add, port, id, aid, scy, net, type, host, path, tls, sni}` 拼接后 base64 编码就是分享链接，不同客户端字段略有差异。

## 四、验证清单（每次换完必跑）

```bash
# 1. 服务运行
systemctl is-active v2ray nginx

# 2. 端口
ss -tlnp | grep -E ":443 |:10086"

# 3. DNS（查权威，避开缓存）
dig +short @a.ns.dnshe.org moneyfreedom.cc.cd

# 4. HTTPS
curl -sI --max-time 10 https://moneyfreedom.cc.cd

# 5. WS 握手 —— 必须是 101
curl -sv --max-time 10 \
  -H "Connection: Upgrade" -H "Upgrade: websocket" \
  -H "Sec-WebSocket-Version: 13" \
  -H "Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==" \
  "https://moneyfreedom.cc.cd/v2raypath" 2>&1 | grep "< HTTP"

# 6. 真实代理测试（本机用临时 v2ray 客户端）
#    配一个 socks 入站 → 通过它访问 google，看 HTTP 200
```

**判定**：第 5 步 `101` + 第 6 步 `200` = 成功。

## 五、踩过的坑（重要！）

### 坑 1：nginx `http2` 会让 WebSocket 不通 ⭐⭐⭐

**nginx 的 HTTP/2 是「监听套接字」级别协商。** 只要同一 `IP:443` 上**任意一个** server 块写了 `listen 443 ssl http2`，整个 443 对所有 SNI 都优先 h2。而 **HTTP/2 不支持传统 WebSocket Upgrade**，v2ray WS 握手被拒 → 客户端超时。

**症状**：`curl` 得到 `HTTP/2 400` + `sec-websocket-version: 13`（看着像通了，其实不行）。

**解决**：**所有** 443 的 server 块都改成 `listen 443 ssl;`（去掉 `http2`）。

```bash
grep -rn "listen 443" /etc/nginx/conf.d/   # 必须一条 http2 都不能有
```

> nginx 1.24 不支持独立的 `http2 on;` 指令（那是 1.25.1+）。

**验证**：加 `--http1.1` 强制测试时应返回 `101`。

### 坑 2：配置文件名必须以 `.conf` 结尾 ⭐⭐

nginx 只 `include /etc/nginx/conf.d/*.conf`。写成 `moneyfreedom.conf.2` 或 `.bak` **完全不生效**，且 `nginx -t` 不会报错。

```bash
ls /etc/nginx/conf.d/*.conf   # 确认你的文件在这里
```

### 坑 3：多 IP 时映射关系可能和直觉相反 ⭐⭐

见 1.3。**先 `curl --interface` 实测再决定改哪个**，别猜。

### 坑 4：本机 DNS 缓存会骗你 ⭐

- Oracle 内部 DNS `169.254.169.254` 有自己的缓存，会缓存旧记录直到 TTL 过期（600s）。
- `moneyfreedom.cc.cd` 改完后本机 `dig` 仍返回旧 IP 是正常的，**不代表 DNS 没改对**。
- **判断方法**：查权威 NS（`@a.ns.dnshe.org`）+ 查公共 DNS（`@8.8.8.8`），都对了就是改好了。

### 坑 5：`netplan try` 在非交互环境会回滚 ⭐

它要按回车确认。脚本环境用 `netplan apply`。

### 坑 6：443 默认站点冲突

如果服务器上有别的站点（如 quantdinger）也占 443，注意 `default_server` 只有一处能不冲突。按 `server_name`/SNI 区分即可共存，但 **http2 必须全关**（见坑 1）。

### 坑 7：`v2ray test` 的参数

V2Fly 5.x 是 `v2ray test -c config.json -d confdir`，不是 `v2ray run -test`。

## 六、一键应急脚本（改完 DNS 后跑）

```bash
#!/usr/bin/env bash
set -u
DOMAIN="moneyfreedom.cc.cd"
NS="a.ns.dnshe.org"

echo "== DNS（权威）=="
dig +short @$NS "$DOMAIN"

echo "== 服务 =="
systemctl is-active v2ray nginx

echo "== 端口 =="
ss -tlnp | grep -E ":443 |:10086"

echo "== HTTPS =="
curl -sI --max-time 10 "https://$DOMAIN"

echo "== WS 握手（要 101）=="
curl -sv --max-time 10 --http1.1 \
  -H "Connection: Upgrade" -H "Upgrade: websocket" \
  -H "Sec-WebSocket-Version: 13" \
  -H "Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==" \
  "https://$DOMAIN/v2raypath" 2>&1 | grep "< HTTP"

echo "== nginx http2 检查（应为空）=="
grep -rn "listen 443.*http2" /etc/nginx/conf.d/ /etc/nginx/sites-enabled/ 2>/dev/null

echo "== 出口 IP =="
curl -s --max-time 8 https://api.ipify.org
```

## 七、备份位置一览

| 文件 | 备份 |
|---|---|
| `/etc/netplan/60-cloud-init.yaml` | `.bak-20261110-0240` |
| `/etc/netplan/50-cloud-init.yaml` | `.bak-20261110-0240` |
| `/etc/nginx/conf.d/moneyfreedom.conf` | `.bak-http2fix-0228`、`.2.bak-20261110-0224` |
| `/etc/nginx/conf.d/quantdinger.conf` | `.bak-http2fix-0228` |
| `/etc/systemd/resolved.conf` | `.bak-*` |
| `/etc/v2ray/conf/VMess-TCP-8443.json` | `.bak-20261110` |

## 八、待办 / 可优化

- [ ] `.cc.cd` 后缀 Cloudflare 无法接入（不在其支持 TLD 列表），不能走 CF 代理
- [ ] 旧通道 8443/8899 目前仅靠 IP 暴露，建议后续也迁到域名或关闭
- [ ] SS 密码偏弱，建议更换
- [ ] 若要「只让 v2ray 走某个 IP、其他服务走别的」，需做源地址策略路由（配合 `ip rule`）

---

## 九、容易被封，对策

### 前置

 新申请域名 + 新IP，新节点迁到 `17tizi.ccwu.cc` → `161.153.27.161`，只留 443 一个入口，双协议走不同高熵路径 + token 校验，裸端口全部关掉，出站也统一到新 IP。这些能显著延长 IP 存活时间，但不保证永不被墙。

### 实际改动（7 项，全部验证通过）

| 改动 | 防封作用 |
|---|---|
| 关掉 8899 裸 SS、8443 裸 VMess | ⭐ 最关键。裸端口是 GFW 主动探测最易命中的把柄，会连坐整个 IP。这是前几次换 IP 没几天又挂的主因 |
| 换掉 `/v2raypath` → `runtime.4af4ba363b4c65bfdb0c.js` | 高熵路径像静态 JS 资源，挡掉路径扫描 |
| 新增 SS 路径 → `2.e46bfb688c741047cdda.chunk.js` | 双协议同一 443、不同路径，协议有备选但不增加暴露面 |
 | nginx token 校验（map 检查 `X-Auth-Key` 头或 `?k=`） | 无 token 一律 404（不是 403/101），主动探测拿不到握手响应。token 写进 WS path，链接自包含 |
| 伪装站点 Northstar Edge 2.5KB 正常企业页 | 根路径 200，看起来是正常网站；顺手删掉老 moneyfreedom 的 `/v2raypath` 反代（它还会返回 101，等于给探测留门） |
 | TLS 指纹伪装 `fp=chrome` | 让 ClientHello 像真实浏览器；V2Fly 忽略该字段但不影响其他配置 |
 | 新证书 + 自动续签，出站切到 `10.0.0.135`、删 moneyfreedom 证书 | 入出站统一到新 IP；清掉续签失败告警 |
