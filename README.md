# LocalDrop 手机视频下载器

这是一个移动优先的单页应用，参考 SnapWC 的输入、解析、结果和 FAQ 流程设计。前端可以直接部署到 GitHub Pages；默认依次尝试公开的 VxTwitter 和 FxTwitter 接口解析 X/Twitter 帖子，不要求你注册 Cloudflare 或 X Developer 账号。

## 使用方式

1. 直接在浏览器打开 `index.html`，或将整个文件夹部署到 GitHub Pages。
2. 复制 MP4、WebM、MOV、JPG、PNG 或 GIF 的直链。
3. 粘贴后点击“获取下载链接”，预览并保存文件。
4. X/Twitter 帖子页直接点击“获取下载链接”即可尝试解析；默认使用公开第三方接口。
5. 在支持 PWA 的浏览器中，可通过“添加到主屏幕”安装成手机应用。

## 能力边界

默认解析请求依次发送到 `https://api.vxtwitter.com/X/status/{tweetId}` 和 `https://api.fxtwitter.com/2/status/{tweetId}`。这些接口不是 X 官方接口，也不是你控制的服务器；它们免注册、免 Token，但可能限流、停机或改变返回格式。

如果不部署 Worker，应用仍然支持直接媒体直链（通常包含 `video.twimg.com`、`pbs.twimg.com` 或以 `.mp4` 等扩展名结尾）。

直链模式不会经过本应用服务器；最近使用的链接仅保存在当前设备的浏览器 localStorage 中。解析 X/Twitter 帖子时，帖子 ID 会发送给默认的 FxTwitter 公开接口，这是使用第三方解析服务的必要步骤。

## 可选本地预览

如果浏览器对 `file://` 下的媒体预览有限制，可以在包含该文件夹的电脑上临时运行一个静态文件服务器，再用手机访问同一局域网地址。应用本身不需要后端接口。

## 可选：改用自己的解析接口

如果将来你愿意注册 X Developer 和 Cloudflare，可以改用项目内的 Worker。Worker 的源码在 `worker/src/index.js`，示例配置在 `worker/wrangler.toml.example`；页面里的“自定义解析接口（可选）”可覆盖默认公开接口。

1. 在 X Developer Platform 创建应用并获取可用于 X API v2 的 Bearer Token。X API 的套餐、读取额度和收费可能变化；部署前先检查你的开发者账号可用额度与价格。
2. 安装 Node.js，然后在 `worker` 目录复制 `wrangler.toml.example` 为 `wrangler.toml`，把 `ALLOWED_ORIGIN` 改成你的 GitHub Pages 源，例如 `https://your-name.github.io`（项目站点则包含仓库路径的 origin 不变，origin 本身不含路径）。
3. 创建 KV 命名空间：`npx wrangler kv namespace create localdrop-rate-limits`。把输出的 namespace ID 填到 `wrangler.toml` 的 `id`。
4. 在 `worker` 目录运行 `npx wrangler login`，再运行 `npx wrangler secret put X_BEARER_TOKEN`，按提示粘贴 Token。不要把 Token 写入前端、GitHub 仓库或普通配置变量。
5. 运行 `npx wrangler deploy`。部署成功后会得到 `https://...workers.dev` 地址。
6. 在手机打开 GitHub Pages 站点，在“配置我的解析接口”中填入 Worker 地址并保存；再粘贴 X/Twitter 帖子 URL 点击解析。

Worker 限制每个 IP 每小时最多解析 20 次，并且只接受配置的 GitHub Pages Origin。这个限制可减少误用，但不能替代 Cloudflare 账号侧的用量告警；X API 仍可能按你的套餐计费。Worker 仅支持公开帖子，私密、受保护或需要登录权限的媒体不会解析。

## GitHub Pages 部署

仓库内包含 `.github/workflows/deploy-pages.yml`。把本文件夹内容推送到 GitHub 仓库默认分支后，在仓库 Settings → Pages 中将 Build and deployment source 设为 GitHub Actions；之后每次推送都会自动发布静态页面。Worker 需要单独部署到 Cloudflare，不会随 GitHub Pages 一起运行。
