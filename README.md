# app-static-update · StandbyCleaner 的更新清单仓库

客户端从这里读 `update.json` 决定要不要提示更新。单独开一个仓库的理由只有
一条：免费版 GitHub Pages 要求仓库公开，而主项目里有全部源码。把清单拆出去，
源码仓库可以保持私有，客户端照样能拿到更新。

| 项 | 值 |
| --- | --- |
| 仓库 | https://github.com/1SZZ2ZZ/app-static-update |
| 分支 | `main`（根目录） |
| 客户端读的地址 | `https://1szz2zz.github.io/app-static-update/update.json` |
| 声明式备份地址 | `https://raw.githubusercontent.com/1SZZ2ZZ/app-static-update/main/update.json` |

`index.html` 是清单页，自己 fetch 同目录的 `update.json` 把版本与更新内容渲染
出来。它的用处不只是好看——浏览器能打开它，就说明 Pages 通了、清单能取到；
打不开就说明是托管的问题而不是客户端的问题。

## 一次性设置（只做一次）

1. 仓库 Settings -> Pages，Source 选 `Deploy from a branch`，分支 `main`，
   目录 `/ (root)`，保存。等一两分钟生效。
2. 本机推一次内容（这个目录已经 `git init` 并提交过了）：

   ```bash
   cd publish
   git remote add origin https://github.com/1SZZ2ZZ/app-static-update.git
   git branch -M main
   git push -u origin main
   ```

3. 客户端地址已写好（`app/update.py` 的 `DEFAULT_FEED_URL`），不用再改。
   换源时才需要：

   ```bash
   python tools/feed_set.py https://<新地址>/update.json
   ```

## 发新版本的顺序

顺序错了会出现「清单说 1.3.0 能下，点下去 404」，按这个来：

1. 在主项目改 `app/__init__.py` 的 `__version__` 为新版本号，跑 `python build.py` 重新打包；
2. 把 exe 传蓝奏云，拿解析后的直链。蓝奏云直链需要带 Referer 才不 403，
   把那个值填进下面的 `referer`；
3. 改**主项目根目录**的 `update.json`（那是唯一源，本地这份是它的副本）：

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
5. 等 Pages 刷新（通常不到一分钟），用 `python tools/feed_probe.py` 复核一遍
   再通知用户。

## 注意

- **清单文件不要改名。** 客户端请求的就是 `update.json`，改名等于换地址，
  已经发出去的包会一直等不到更新。
- 这个仓库里没有源码、也没有安装包（直链在网盘上），所以它公开不暴露任何东西。
- 顺带一提：仓库里曾经传上来一个 `settings.json`，那是客户端自己的运行配置，
  不是清单，已经删掉。要改本机配置请改主项目里的那份，别往这里放。
