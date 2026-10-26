---
title: ubuntu服务器迁移
date: 2026-10-26
categories: 运维
tags:
    - ubuntu
    - 服务器迁移
    - 部署
#description: 记录 Ubuntu 服务器迁移全过程：3x-ui 代理面板、ftt 前端项目、bbgo 源码编译、QuantDinger 容器化部署及 Binance 代理中转。
---

本文记录一次完整的 Ubuntu 服务器迁移流程，按组件拆分为四个部分：代理面板 3x-ui、前端项目 ftt、量化框架 bbgo、以及容器化项目 QuantDinger。

<!-- more -->

## 一、安装 3x-ui

1、提前解析好域名到服务器。

2、ufw 放行 80、443（申请证书时需要）：

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

3、安装 3x-ui：

```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

4、具体安装步骤参照 https://www.youtube.com/watch?v=wES8BEcYczI

5、登录 3x-ui 网站，新建入站、客户端。

6、放行入站和客户端使用的端口（以及 `2053`、`2096`）：

```bash
sudo ufw allow 2053/tcp
sudo ufw allow 2096/tcp
# 根据实际入站/客户端端口继续放行
```

## 二、安装 ftt

> 前置：提前解析好域名到服务器。

```bash
# 安装 Node 16.x
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

# 验证
node -v   # v16.x

# 全局装 pm2（若用 NodeSource 方式）
sudo npm install -g pm2

# 克隆项目
git clone https://github.com/17it/fortune

# 回到项目目录
cd fortune
npm install
npm run build
pm2 start ecosystem.config.js
```

安装 Nginx 并迁移原配置与证书：

```bash
# 安装 nginx
sudo apt update
sudo apt install -y nginx

# 拷贝 nginx 配置到新服务器
scp /etc/nginx/conf.d/* root@207.148.89.101:/etc/nginx/conf.d
# 拷贝证书到新服务器
scp /etc/nginx/ssl/* root@207.148.89.101:/etc/nginx/ssl

sudo systemctl start nginx      # 启动
sudo systemctl enable nginx     # 开机自启
sudo systemctl status nginx     # 查状态
```

> 有可能 80 被其他进程占用，导致访问不到项目：

```bash
sudo pkill -f 'nginx: master'
sudo systemctl start nginx
```

## 三、安装 bbgo（从源码编译安装）

```bash
# 安装 go
wget https://go.dev/dl/go1.27.2.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.27.2.linux-amd64.tar.gz

# 将 Go 的二进制目录添加到 PATH
export PATH=$PATH:/usr/local/go/bin
source ~/.profile

# 检查
go version

# 安装 bbgo
CGO_ENABLED=1 go install -x github.com/c9s/bbgo/cmd/bbgo@main
export PATH=~/go/bin:$PATH

# 验证
bbgo version
```

安装 Redis 并迁移配置：

```bash
# 安装 redis
apt update && apt install -y redis-server
systemctl enable --now redis-server
redis-cli ping   # 返回 PONG 即正常

# 拷贝配置到新服务器
scp bbgo.yaml bbgo.yaml.bak bbgo1.sqlite3 .env.local root@207.148.89.101:/root
```

运行（使用 tmux 后台常驻）：

```bash
# 后台运行
apt install -y tmux

# 新建会话并运行
tmux new -s bbgo
# 进入 tmux 后执行启动命令（--no-sync 不同步之前订单）
/root/go/bin/bbgo run --no-sync

# 按 Ctrl+B 然后按 D 脱离会话（程序继续后台跑）
# 回来查看
tmux attach
```

## 四、安装 QuantDinger（预构建镜像）

执行安装脚本：

```bash
sudo bash https://github.com/17it/fortune.git/quantdinger_install.sh
```

修改配置：

```bash
cd ~/QuantDinger
vim .env

# 修改为
FRONTEND_HOST=0.0.0.0
MOBILE_HOST=0.0.0.0
```

重启并验证：

```bash
docker compose up -d

# 验证：应显示 0.0.0.0:8888 和 0.0.0.0:8889
ss -tlnp | grep -E "8888|8889"

# 防火墙放行端口
sudo ufw allow 8888/tcp
sudo ufw allow 8889/tcp
```

### 代理（oracle 在 usa，被 binance 禁止连不上，用 jap 服务器代理中转）

**一、在 3x-ui 面板添加 HTTP 入站**

- 备注：`quantdinger-proxy`
- 协议：http（推荐，ccxt 原生支持，免装 PySocks；也可选 socks）
- 监听 IP：留空或 `0.0.0.0`
- 端口：`3128`（随意，别和面板/其他节点冲突）
- 用户名/密码：务必设置，如 `qd` / 强密码
- 修改后重启 Xray

**二、jap 服务器验证 + 放行防火墙**

```bash
# 本机先验证代理是否生效（换成你的用户名密码）
curl -s -x http://qd:你的密码@127.0.0.1:3128 -o /dev/null -w '%{http_code}\n' https://api.binance.com/api/v3/ping
# 应返回 200

# 只允许美国服务器访问这个端口
sudo ufw allow from 161.153.24.138 to any port 3128
```

**三、oracle 服务器设置代理**

```bash
cd ~/QuantDinger
sudo vim docker-compose.yml
```

在 `backend` 服务 `environment:` 段追加代理：

```yaml
- HTTP_PROXY=http://qd:你的密码@JP_IP:3128
- HTTPS_PROXY=http://qd:你的密码@JP_IP:3128
- NO_PROXY=localhost,127.0.0.1,postgres,redis,db
```

重启并验证：

```bash
docker compose up -d

docker compose exec backend sh -c "curl -s -o /dev/null -w '%{http_code}\n' https://api.binance.com/api/v3/ping"
```
