Title: KOReader Plugins：给电子书阅读器加上稍后阅读、云端同步和中文字体
Date: 2026-09-11 18:20
Tags: KOReader, Kindle, EPUB, rclone, Supabase, Lua
Category: 阅读
Slug: koreader-plugins
Author: muxueqz
Summary: 为 KOReader 增加稍后阅读、书库同步和 Kindle 中文字体支持

# KOReader 也可以自己加功能

KOReader 本身已经是一个非常完整的电子书阅读器，不过我的阅读流程里还有一些小问题：看到网页文章时想先收藏，回到 Kindle 后又希望能直接读；设备里的书需要和远程书库同步；Kindle 自带的部分中文字体，却被 KOReader 默认列入了黑名单。

于是有了这个 [KOReader Extras](https://github.com/muxueqz/koreader-extras) 仓库。它不是 KOReader 官方项目，而是一组面向个人使用的插件和补丁，主要解决三个问题：把待读文章变成 EPUB、同步远程书库，以及恢复 Kindle 的部分中日韩字体。

## 这组插件解决什么问题？

目前仓库 README 中列出的内容有三个。

### readitlater：把网页文章带到阅读器里

`readitlater.koplugin` 使用 Supabase 保存待读列表，在 KOReader 中同步这些链接。设备同步到待阅读条目后，插件会完成下面的工作：

1. 从文章 URL 下载网页。
2. 读取网页标题。
3. 将网页内容转换成本地 EPUB。
4. 把文章和状态保存到设备上的 SQLite 数据库。
5. 在 KOReader 的菜单中提供待读列表，直接打开 EPUB 阅读。

读完后可以在阅读器里归档，之后再把本地状态同步回 Supabase。设备临时没有网络时，已经下载好的文章仍然可以从本地列表打开。

### rclone-sync：同步远程书库

`rclone-sync.koplugin` 给 KOReader 主菜单增加一个同步入口，实际调用设备上已经安装的 rclone，将：

```text
remote:/book/
```

同步到：

```text
/mnt/us/boxbook/
```

同步过程中使用锁文件避免重复运行，并排除 `*.sdr/` 目录。这样可以把远程存储中的书籍集中同步到 Kindle 的本地书库，同时不把阅读器产生的 SDR 目录当成普通书籍同步。

### Kindle CJK 字体补丁：解锁内置中文字体

Kindle 系统里有一些中日韩字体，但 KOReader 默认会屏蔽其中一部分。`patches/1-enable-kindle-cjk-fonts.lua` 只放开下面几种字体：

```text
STSongMedium.ttf
STHeitiBold.ttf
STHeitiMedium.ttf
STKaiMedium.ttf
```

补丁不会清空整个字体黑名单，其他字体仍然保持 KOReader 原来的处理方式。它只适用于 Kindle，前面的两个组件不限定 Kindle。

## 安装

先确认设备上已经安装并能正常运行 KOReader。然后把插件目录复制到 KOReader 的插件目录：

```text
koreader/plugins/readitlater.koplugin/
koreader/plugins/rclone-sync.koplugin/
```

如果需要 Kindle 字体补丁，把文件复制到 patches 目录：

```text
koreader/patches/1-enable-kindle-cjk-fonts.lua
```

复制完成后重启 KOReader。插件会在主菜单中注册自己的入口，字体补丁则会在启动时生效。

## 配置 rclone 同步

设备上需要有与 CPU 架构匹配的 rclone。这个仓库不打包 rclone，默认位置是：

```text
/mnt/us/rclone
```

接着创建配置文件：

```text
koreader/plugins/rclone-sync.koplugin/rclone.conf
```

配置中的 remote 名称必须叫 `remote`，因为插件固定同步 `remote:/book/`。例如，配置完成后可以先在命令行确认远程端能被访问，再从 KOReader 主菜单进入 `RcloneSync`。

`rclone.conf` 通常包含 OAuth token 或其他访问凭据，不要把它提交到 Git 仓库，也不要直接复制别人的配置文件。

## 配置稍后阅读

`readitlater` 需要一个 Supabase 项目，并在插件目录中创建私有的 `config.lua`：

```lua
return {
    supabase_url = "https://YOUR_PROJECT.supabase.co",
    supabase_key = "YOUR_PRIVATE_KEY",
    table_name = "links",
}
```

这里的 URL、密钥和表名需要替换成自己的值。插件通过 Supabase REST API 查询 `pending` 和 `complete` 状态的条目，并根据条目的 `id`、`url` 和状态处理本地 EPUB。

如果某些网站必须登录才能访问，可以创建私有的 `cookies.lua`，按域名提供 Cookie：

```lua
return {
    ["example.com"] = "session=YOUR_PRIVATE_COOKIE",
}
```

插件会优先匹配完整域名，也会尝试匹配更高层级的域名。Cookie 只应放在确实需要登录的网站配置下，不要把浏览器导出的全部 Cookie 都放进去。

## 一次完整的阅读流程

配置好 Supabase 后，可以把它当成一个简单的跨设备稍后阅读箱：

1. 在其他设备上把文章 URL 写入 Supabase 的待读表。
2. 在 KOReader 的 `Read It Later` 菜单中执行 `Sync`。
3. 插件下载网页并生成 EPUB，保存到 KOReader 的数据目录。
4. 通过 `Show List` 打开待读文章。
5. 阅读完成后，在当前文档的菜单中选择 `Archive`。
6. 下次同步时，已完成的状态会回写到 Supabase，本地对应的 EPUB 也会被清理。

如果某篇文章下载失败，后续同步会重试；在待读列表中长按条目，也可以单独重新下载。网页内容是否能被正确转换，取决于目标网站的 HTML 结构，因此复杂页面、登录页面和强依赖 JavaScript 的页面可能需要额外测试。

## 隐私和安全

这组插件很适合个人使用，但配置时仍然要注意几件事：

- `readitlater` 会把文章 URL、条目状态和时间戳发送到你配置的 Supabase 服务。
- 待读文章会从对应 URL 下载，并保存成设备上的本地 EPUB。
- 配置的 Cookie 会在匹配域名的下载请求中发送。
- `rclone-sync` 能访问哪些文件，取决于 `rclone.conf` 中配置的 remote 权限。
- `config.lua`、`cookies.lua`、`rclone.conf`、OAuth token 和服务端密钥都不应该提交或上传。

当前版本的网页下载还会通过 shell 命令传递远程 URL。使用不可信的远程数据前，应先审查并修复 URL 转义问题；带 Cookie 的下载和 HTTP 重定向也应该在自己的环境中谨慎测试。

## 写在最后

这不是一个试图覆盖所有阅读场景的大型框架，而是几个很实用的小工具：一个负责把网页变成适合阅读器的 EPUB，一个负责把书库同步到设备，另一个负责把 Kindle 已经拥有的中文字体重新利用起来。

KOReader 的插件接口让这类个人化改造并不复杂。对我来说，最重要的结果是把“看到文章”“同步到设备”和“坐下来阅读”这几个原本分散的动作串成了一条简单的流程。

