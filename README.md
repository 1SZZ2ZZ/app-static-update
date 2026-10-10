# 发布说明 · 这个仓库只放更新清单

给 StandbyCleaner 的客户端读的 `update.json` 挂在这里，由 GitHub Pages 直接
发出去。单独开一个仓库的理由只有一个：免费版 Pages 要求仓库公开，而主仓库
里有全部源码。把清单拆出去，源码仓库可以保持私有，客户端也照样能拿到更新。

## 一次性设置

1. 在 GitHub 上新建一个公开仓库，名字随意，例如 `StandbyCleaner-update`；
2. 把这个目录的内容推上去：

   ```bash
   cd publish
   git remote add origin https://github.com/<用户名>/StandbyCleaner-update.git
   git branch -M main
   git push -u origin main
   ```

3. 仓库 Settings -> Pages，Source 选 `Deploy from a branch`，分支 `main`，
   目录 `/ (root)`，保存。等一两分钟，清单地址就是：

   ```
   https://<用户名>.github.io/StandbyCleaner-update/update.json
   ```

4. 回到主项目，把这个地址写进客户端：

   ```bash
   python tools/feed_set.py https://<用户名>.github.io/StandbyCleaner-update/update.json
   ```

   主库那份 `update.json` 是这份文件的源。改完主库那一份，跑一次
   `python tools/feed_publish.py` 把它同步到这里再推，避免两处版本号不一致。

## 发新版本的顺序

顺序错了会出现「清单说 1.3.0 能下，点下去 404」，所以按这个来：

1. 在主项目改 `app/__init__.py` 的 `__version__` 为新版本号，重新打包；
2. 把 exe 传蓝奏云，拿解析后的直链。蓝奏云直链现在需要带 Referer 才不 403，
   把那个值填进下面的 `referer`；
3. 改主项目的 `update.json`：

   | 字段 | 必填 | 说明 |
   | --- | --- | --- |
   | `version` | 是 | 新版本号，必须**大于**用户已装版本 |
   | `downloadUrl` | 是 | 安装包直链 |
   | `notes` | 否 | 更新日志，字符串数组，每项一行，直接显示在弹窗里 |
   | `size` | 否 | 安装包字节数，填了会在下载后核对，拦「下到 HTML 错误页」 |
   | `minSupported` | 否 | 最低支持版本。本机低于它强制更新，适合跨兼容性断点 |
   | `force` | 否 | 对清单所有老版本强制更新 |
   | `publishedAt` | 否 | 发布日，显示在弹窗「更新内容」右侧 |
   | `referer` | 否 | 下载时附带的 Referer |

4. `python tools/feed_publish.py --push`（同步 + 提交 + 推送）；
5. 等 Pages 刷新（通常不到一分钟），用
   `python tools/feed_probe.py <清单地址>` 复核一遍再通知用户。

## 注意

清单文件不要改名。客户端请求的就是 `update.json` 这个名字，改了等于换地址，
已经发出去的包会一直等不到更新。

这个仓库里没有源码，也没有安装包（直链在网盘上），所以它公开不暴露任何东西。
