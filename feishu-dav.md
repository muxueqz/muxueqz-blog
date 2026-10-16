Title: 把飞书云盘挂载到 Linux：feishu-dav
Date: 2026-10-16 10:00
Tags: Python, Linux, 工具
Slug: feishu-dav
Author: muxueqz
Summary: 用一个单文件 Python 脚本把飞书云盘变成 WebDAV 服务，再配合 rclone mount 挂载到 Linux 本地目录

# 介绍

飞书（Lark）在国内团队协作里用得不少，云盘里堆满了共享的文档、表格和归档文件。但飞书官方并没有提供 Linux 下的同步盘客户端，想在 Linux 上方便地访问云盘文件，往往只能网页版手动下载，或者调用开放平台 API 自己写脚本——都很别扭。

[feishu-dav](https://github.com/muxueqz/feishu-dav) 就是为了解决这个问题：它把飞书云盘通过 **WebDAV 协议**暴露成本地 HTTP 服务，再用 `rclone mount` 挂载成一个本地目录。之后你就可以像操作普通文件夹一样浏览、读取、上传飞书云盘里的文件了。

整个项目只有一个 Python 文件 `feishu_dav.py`，依赖也很少：

```bash
pip install wsgidav cheroot requests
```

# 设计思路

feishu-dav 的核心是两层转换：

* **飞书开放平台 API → 文件系统语义**：用 OAuth `user_access_token` 直连飞书开放平台（不依赖 lark-cli），把云盘的文件夹列表、文件下载、上传接口翻译成文件系统操作
* **文件系统语义 → WebDAV**：基于 [wsgidav](https://github.com/mar10/wsgidav) 实现 WebDAV 服务端，于是任何支持 WebDAV 的客户端（rclone、Finder、Windows 资源管理器……）都能直接连

选择 WebDAV 而不是自己发明一个 FUSE 文件系统，好处是 rclone 已经把挂载、缓存、断点续传这些脏活都做好了，feishu-dav 只需要把协议翻译对即可，实现因此可以保持在一千行上下的单文件里。

# 快速上手

安装依赖后，一条命令自动引导：

```bash
python feishu_dav.py --init
```

引导流程参考了 larkfs init 的风格：

1. 检查应用凭证：依次尝试命令行参数 → 环境变量 `FEISHU_APP_ID/FEISHU_APP_SECRET` → 最后交互式输入（会给出在[开发者后台](https://open.feishu.cn/app)创建应用的指引）
2. 自动打开浏览器完成 OAuth 授权（本地回调 `127.0.0.1:8577`）
3. 验证云盘访问权限并保存配置到 `~/.feishu-dav/config.json`（权限 600，含 token，注意不要外传）

授权采用 **OAuth Device Flow（RFC 8628）**，这是个很贴心的设计：不需要在开放平台配置重定向 URL。SSH 到服务器上操作时加 `--no-browser`，只打印授权链接，复制到本机浏览器或手机飞书扫码打开即可。

非交互场景（CI / 脚本）也支持：

```bash
python feishu_dav.py --init \
  --app-id cli_xxx --app-secret xxx \
  --mount Shared:xxxx --read-only
```

初始化完成后启动服务：

```bash
python feishu_dav.py              # 默认 127.0.0.1:8765
python feishu_dav.py --doctor     # 自检配置/凭证/挂载点，给出修复建议
```

# 用 rclone 挂载

```bash
rclone config create feishu webdav url=http://127.0.0.1:8765 vendor=other
mkdir -p ~/feishu
rclone mount feishu: ~/feishu --vfs-cache-mode writes --daemon
```

之后 `ls ~/feishu` 就能看到云盘内容了。不挂载也可以直接用 `rclone lsf feishu:` 浏览，或者任何 WebDAV 客户端连接。

# 一些实用细节

* **在线文档**：docx/sheet/bitable/slides 这类飞书在线文档没有实体文件，feishu-dav 把它们映射为只读虚拟文件，目录里显示为 `文档名.docx/xlsx/pdf`（内容是导出产物），导出结果缓存在 `~/.feishu-dav/export_cache/` 避免重复导出。飞书导出 API 不支持 markdown，所以没有 md 格式可选
* **别人共享的文件夹**：在向导里把该文件夹 URL 中的 folder_token 加为挂载点即可
* **安全控制**：删除操作默认禁止（需 `--allow-delete` 开启）；`--read-only` 模式下一切写操作返回 403。放在团队共享场景下建议只读挂载
* **性能**：目录列表有 60s TTL 缓存，写操作后主动失效；在线文档采用 lazy 导出——列目录零导出、秒级返回（有缓存的报真实大小，未缓存的不报大小），首次 GET 下载时才真正导出并写缓存（有效期 24h），之后列目录和挂载打开都正常。代价是 rclone 对未缓存文档按空文件处理，挂载场景首次访问前需要 curl 预热一次；普通文件的大小由后台线程异步探测，下次刷新生效
* **同名文件**：飞书云盘允许同名，WebDAV 路径不允许，所以同名文件会显示为 `name~token前8位`

# 踩过的坑

* **文件大小显示为 0**：飞书 `drive/v1/files` 列表接口不返回 size 字段，PROPFIND 只能报 0。不影响实际下载——rclone 是流式传输按实际内容计的，上传后的文件大小也正常
* **部分文件下载 403**：自动创建的 PersonalAgent 应用即使授权页显示包含 `drive:file:download`，媒体下载接口仍可能返回 403。解决方法是到开发者后台确认 drive 相关权限已开通，并且**创建版本并发布**——只开通不发布是不生效的，然后重新 `--init` 授权
* **授权链接超时**：链接有效期约 4 分钟，超时重跑 `--init` 即可

# 小结

feishu-dav 是一个典型的「用胶水解决实际问题」的小工具：WebDAV + rclone 这套组合是现成的，飞书开放平台 API 是现成的，缺的只是中间那一层协议翻译。单文件实现、`--init` 一条命令完成引导、`--doctor` 自检，上手成本很低。如果你在 Linux 上工作而又离不开飞书云盘，值得一试。
