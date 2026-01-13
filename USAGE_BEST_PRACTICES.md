# 爬虫使用最佳实践

本文结合当前程序功能，给出在爬虫场景中的推荐用法与配置要点，帮助你减少成本、提升稳定性与速度。

## 1. 端口与固定 IP 策略

- 主代理端口（默认 `18080`）：每次请求可能使用不同出口 IP，适合需要轮转 IP 的场景。
- 隧道端口范围（默认 `18100-18199`）：**每个端口固定一个出口 IP**，适合稳定会话或需要固定 IP 的场景。
- 重要提示：**所有端口共享亿牛云爬虫代理的“新建链接数量”**（连接数/计费规则以亿牛云为准）。

参考（亿牛云说明）：
```
https://www.16yun.cn/help/ss_demo/#_1
```

实战建议：
- 将业务按端口分组（比如不同任务/账号/站点使用不同隧道端口）实现隔离。
- 控制并发与连接复用，降低“新建链接数量”。

## 2. 本地代理免认证（浏览器/软件更好配置）

本程序会自动为上游亿牛云代理添加认证信息，本地代理**无需认证**即可使用：

- 爬虫/脚本：直接设置本地 HTTP 代理即可。
- 浏览器/软件：只需要填入 `http://<host>:<port>`，不需要用户名密码。

示例：
```
# HTTP 代理
curl -x http://127.0.0.1:18080 https://httpbin.org/ip

# 隧道代理（固定出口 IP）
curl -x http://127.0.0.1:18100 https://httpbin.org/ip
```

## 3. 使用 Bypass 节省流量（推荐）

通过路由规则（Bypass），你可以：
- **DIRECT**：直连不走代理（节省流量/提升速度）
- **REJECT**：直接阻断（减少不必要请求）
- **PROXY**：继续走代理（保留匿名性/固定 IP）

### 3.1 典型场景

- **本地/内网直连**：避免请求走代理导致访问失败或浪费流量
- **静态资源/埋点/广告直连或阻断**：减少大流量资源消耗
- **仅对核心目标走代理**：其他请求全部直连

### 3.2 示例配置

```yaml
routing:
  defaultPolicy: "PROXY"
  rules:
    # 本地与内网直连
    - type: "DOMAIN-SUFFIX"
      value: "local"
      policy: "DIRECT"
    - type: "IP-CIDR"
      value: "127.0.0.0/8"
      policy: "DIRECT"
    - type: "IP-CIDR"
      value: "192.168.0.0/16"
      policy: "DIRECT"

    # 静态资源/埋点拦截
    - type: "DOMAIN-KEYWORD"
      value: "analytics"
      policy: "REJECT"
    - type: "DOMAIN-KEYWORD"
      value: "static"
      policy: "REJECT"

    # 国内站点直连
    - type: "DOMAIN-SUFFIX"
      value: "cn"
      policy: "DIRECT"

    # 仅目标站点走代理
    - type: "DOMAIN-SUFFIX"
      value: "google.com"
      policy: "PROXY"

dns:
  use_local_dns: true
  dns_server: ["114.114.114.114", "223.5.5.5"]
```

说明：
- 规则按顺序匹配，**首次命中即停止**。
- `REJECT` 会返回 **HTTP 450**。
- `IP-CIDR` 对主机名生效需开启 `dns.use_local_dns`。
- 规则命中统计在 Web 页面查看：`/routing`（内存保存，重启清零）。

## 4. 爬虫项目中的推荐做法

1. **优先使用隧道端口**
   - 同一任务/账号/站点固定使用某个隧道端口，避免频繁 IP 变化。

2. **合理拆分请求**
   - 主数据请求走 `PROXY`，静态资源走 `DIRECT` 或 `REJECT`。

3. **控制并发与连接复用**
   - 尽量复用连接，减少新建链接数量。

4. **观察统计数据**
   - 管理界面统计与 `/routing` 页面可以帮助定位流量浪费点。

## 5. 常见问题

- **为什么某些 IP-CIDR 规则不生效？**
  未开启 `dns.use_local_dns` 时，IP-CIDR 只对 IP 字面量生效。

- **REJECT 返回什么？**
  REJECT 直接返回 HTTP 450，不会向目标或上游发起请求。

