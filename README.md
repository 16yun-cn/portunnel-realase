# 亿牛云代理转发工具

本程序是一个本地 HTTP/HTTPS 代理转发服务，用于对接 **亿牛云（16yun）爬虫代理**。它会在本机监听固定端口，自动为上游请求添加认证信息，并支持“主代理端口（随机IP）”与“隧道端口（固定IP）”两种使用方式，同时提供 Web 管理界面查看统计数据。

## 适用场景

- 将亿牛云代理统一封装为本地代理端口，供脚本/浏览器/软件直接使用
- 通过不同隧道端口固定出口 IP，方便分业务隔离
- 通过 Web 面板查看请求成功率与域名访问统计

## 快速开始

1) 准备配置文件 `config.yaml`（可从示例复制并修改）：

```
cp config.example.yaml config.yaml
```

2) 运行程序：

```
./proxy-tunnel-<os-arch>
```

3) 访问 Web 管理界面：

```
http://localhost:19090
```

4) 使用代理：

```
# 主代理端口：每次请求可能获得不同 IP
curl -x http://localhost:18080 https://httpbin.org/ip

# 隧道端口：固定出口 IP
curl -x http://localhost:18100 https://httpbin.org/ip
```

## 配置说明

**注意：** `remoteProxy.host` 必须是 `16yun.cn` 主域名或其子域名，否则程序会报错并退出。

### server
- `server.proxyPort`：主代理端口（无隧道 ID，每次请求可能获得不同 IP）
- `server.webPort`：Web 管理界面端口
- `server.tunnelStart`：隧道代理起始端口
- `server.tunnelCount`：隧道数量（端口范围：`tunnelStart` ~ `tunnelStart+tunnelCount-1`）

### remoteProxy
- `remoteProxy.host`：亿牛云代理地址（必须为 `16yun.cn` 主域名或子域名，例如 `proxy.16yun.cn`）
- `remoteProxy.port`：亿牛云代理端口（默认 3100）
- `remoteProxy.username`：亿牛云账号用户名
- `remoteProxy.password`：亿牛云账号密码

### admin
- `admin.username`：Web 管理员用户名
- `admin.passwordHash`：管理员密码哈希（bcrypt），可用 `htpasswd -bnBC 10 "" <password> | tr -d ':\n'` 生成
- `admin.jwtSecret`：JWT 签名密钥（生产环境务必修改）
- `admin.tokenExpiry`：Token 有效期（秒）

### stats
- `stats.maxHours`：保留的小时统计数量（默认 168=7 天）
- `stats.maxDomains`：保留的域名统计数量（默认 1000）

### retry
- `retry.maxAttempts`：最大重试次数（包含首次请求）
- `retry.baseDelayMs`：退避基准延迟（毫秒）
- `retry.maxDelayMs`：退避最大延迟（毫秒）
- `retry.jitter`：抖动比例（0~1）
- `retry.statusCodes`：可重试的上游状态码列表

### logging
- `logging.debug`：Debug 日志开关（输出更详细的错误与 URL）

## 运行端口说明

- 代理端口：`server.proxyPort`（默认 18080）
- 隧道端口范围：`server.tunnelStart` ~ `server.tunnelStart+tunnelCount-1`（默认 18100-18199）
- Web 端口：`server.webPort`（默认 19090）

## 常见问题

- **启动报错：remoteProxy.host must be a 16yun.cn domain**
  请确认 `remoteProxy.host` 填写的是 `16yun.cn` 主域名或子域名，如 `t.16yun.cn`。
