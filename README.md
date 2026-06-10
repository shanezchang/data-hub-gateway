# data-hub-gateway

`data-hub` 后端的 **Vercel 对外网关**。把 `api.lumina-core.cn` 的所有请求,通过 Vercel
服务端 HTTPS 反代到阿里云后端 `https://www.lumina-core.cn/*`(ECS 上 Caddy 终结 TLS,
反代本机 :8000)。全链路 HTTPS [verified 2026-06-10]。

## 为什么需要它
后端跑在大陆阿里云服务器(SQLite + 爬虫 + API),但域名无 ICP 备案。
**阿里云的 ICP 拦截边界(2026-06-10 实测)**:
- 80 端口明文 HTTP:任何域名 Host 头都被拦截页劫持(裸 IP 访问不拦)
- 443 端口 TLS:域名 SNI **不拦截** —— 所以回源可以走 `https://www.lumina-core.cn`
- 推论:Let's Encrypt 发证只能用 **TLS-ALPN-01**(HTTP-01 的验证请求会被 80 拦截页吃掉)

对外则是 Vercel 的 HTTPS(海外节点),**免备案**。后端不动,对外干净。
⚠️ 若阿里云未来加 SNI 嗅探,此链路会断 —— 症状是 api 域 502/拦截页,届时回退方案:
destination 暂时改回 `http://120.55.183.188:8000/$1`(明文,仅救急)。

## 文件
```
vercel.json   把 /(.*) 全部反代到 https://www.lumina-core.cn/$1
```
⚠️ 不要在本仓库放任何静态文件(含 index.html):Vercel 静态文件优先于 rewrites,
会把对应路径从后端"抢走"。根路径 / 必须透传 —— 后端在 / 返回
{version, capabilities} 探针,llms.txt 的「缓存与同步」协议依赖它。

## 部署
1. Vercel → Add New Project → Import `shanezchang/data-hub-gateway`
2. Framework Preset = **Other**(无需构建)
3. Deploy
4. Settings → Domains → 添加 `api.lumina-core.cn`
5. 按 Vercel 提示,在阿里云云解析把 `api` 记录从 **A → 120.55.183.188** 改成
   **CNAME → Vercel 给的目标**(如 `cname.vercel-dns.com`)

完成后:`https://api.lumina-core.cn/v1/news`、`/docs`、`/portal/*` 全部 HTTPS 可用。

## 后端换地址时
改 `vercel.json` 里的 destination 即可(目前是 `https://www.lumina-core.cn`,
该域名 A 记录指向 ECS 120.55.183.188;TLS 配置见 data-hub 仓库 `deploy/Caddyfile`)。
