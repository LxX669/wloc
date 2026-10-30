# LxX669 WLOC 自用部署指南

本分支由 LxX669 维护，基于 [Yu9191/wloc](https://github.com/Yu9191/wloc)，继续使用 AGPL-3.0。自有部署包括仓库订阅、脚本、选点页面和地图链接解析；地图瓦片、地名搜索仍由地图服务提供。

## 1. 使用自己的模块

Quantumult X 订阅地址：

```text
https://raw.githubusercontent.com/LxX669/wloc/refs/heads/main/modules/wloc.conf
```

其他客户端的地址见 [README](../README.md#订阅地址)。所有模块的脚本与图标均引用本仓库。MITM 主机名保留 `gs-loc.apple.com` 和 `gs-loc-cn.apple.com`，它们是需要拦截的 Apple 服务，不能改为自己的域名。

## 2. 部署自己的选点页面和解析服务

推荐从本仓库的 [一键部署入口](https://deploy.workers.cloudflare.com/?url=https://github.com/LxX669/wloc/tree/main/worker) 开始，在自己的 Cloudflare 账号完成部署。也可以在电脑执行：

```bash
git clone https://github.com/LxX669/wloc.git
cd wloc/worker
npm ci
npx wrangler login
npm run deploy
```

默认 Worker 名称为 `lxx669-wloc`。部署完成后，以 Cloudflare 实际返回的域名为准，不要直接使用示例域名。使用 Pages 时执行 `npm run pages:deploy`。

无需 KV、数据库或地图 API 密钥。页面请求自己的同域 `/api/parse`，不会调用原作者的解析服务。坐标写入代理工具的设备存储；浏览器收藏仅保存在该站点的 localStorage 中。换域名后，原站点收藏不会自动迁移。

## 3. 在 iPhone 使用

1. 启用模块、开启 MITM，并安装和信任证书。
2. Safari 打开刚部署的真实域名，建议添加到主屏幕。
3. 点选地图或粘贴地图分享链接，点击「储存到设备」。
4. 用地图 App 验证；受 GPS、系统缓存影响，结果以设备实测为准。

部署完成后，可将模块的说明及 Loon 的 `#!openUrl` 改为自己的页面域名。目前这些入口指向本指南，不会误跳到上游公共站点。

## 4. 快捷指令使用自己的域名

现有快捷指令：复制到自己的快捷指令 App，找到解析服务地址，将域名改为自己的部署域名，保留 `/api/parse` 路径及其查询参数。

从零创建「设置位置」快捷指令：

1. 接收共享菜单中的 URL 或文本。
2. 创建 URL：`https://<你的部署域名>/api/parse`。
3. 使用「获取 URL 内容」，方法 GET，查询参数 `u` 为共享内容、`format` 为 `json`；通过参数字段传值并正确 URL 编码，避免把地图链接中的 `&` 当作外层参数。
4. 读取返回字典的 `lat` 和 `lon`；若返回 `error`，显示错误并停止。
5. 请求 `https://gs-loc.apple.com/wloc-settings/save`，GET 查询参数 `lat` 和 `lon` 填入上一步结果。检查返回的 `success` 是否为 true。

「清除位置」快捷指令直接请求：

```text
https://gs-loc.apple.com/wloc-settings/save?action=clear
```

Apple 储存请求必须由设备上的代理模块拦截；解析服务则使用自己的 Cloudflare 域名。这两类地址作用不同。

## 维护和验证

```bash
cd worker
npm test
```

测试覆盖地图链接解析、坐标换算，以及 Stash 响应格式。仅更改署名不会改变代码来源；保留 LICENSE、上游致谢及贡献者信息。
