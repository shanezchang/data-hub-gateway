# data-hub-gateway

`data-hub` 后端的 **Vercel 对外网关**。把 `api.lumina-core.cn` 的所有请求,通过 Vercel
服务端 HTTPS 反代到阿里云后端 `http://120.55.183.188:8000/*`。

## 为什么需要它
后端跑在大陆阿里云服务器(SQLite + 爬虫 + API,IP 直连完全正常),但
**未备案域名直连大陆 IP 会被阿里云 ICP 拦截**(连 :8000 也拦)。

Vercel 反代时,发往源站的 Host 是 **IP**(不是域名),所以**绕过 ICP 拦截**;
对外则是 Vercel 的 HTTPS(海外节点),**免备案**。后端不动,对外干净。

## 文件
```
vercel.json   把 /(.*) 全部反代到 http://120.55.183.188:8000/$1
index.html    根路径 / 的极简首页(其余路径都被反代)
```

## 部署
1. Vercel → Add New Project → Import `shanezchang/data-hub-gateway`
2. Framework Preset = **Other**(无需构建)
3. Deploy
4. Settings → Domains → 添加 `api.lumina-core.cn`
5. 按 Vercel 提示,在阿里云云解析把 `api` 记录从 **A → 120.55.183.188** 改成
   **CNAME → Vercel 给的目标**(如 `cname.vercel-dns.com`)

完成后:`https://api.lumina-core.cn/v1/news`、`/docs`、`/portal/*` 全部 HTTPS 可用。

## 后端换地址时
改 `vercel.json` 里的 destination 即可(目前是 `120.55.183.188:8000`)。
