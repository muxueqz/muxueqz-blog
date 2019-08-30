Title: 轻量级书签管理器Skybook 1.0发布
Date: 2019-08-30 18:20
Tags: nim
Slug: skybook_1.0
Author: muxueqz
Summary: 更新啦！

[https://github.com/muxueqz/skybook](https://github.com/muxueqz/skybook)

## 更新内容
### 支持删除书签功能
对，这个功能原本我是没想到的，因为是JSON文本，所以在编辑器中批量删除可能更方便，直到@Tback1提醒我才发现。

由于在目前没用到`javascript`，暂时就用新的表单页面来提供删除功能了，以后再看看要不要调整一下架构。

### 换了个CSS框架
从`Semantic`换到`Milligram`，比原来更轻，不再请求Google Fonts，加速访问

### 配置了项目的CI和自动发布
现在Linux和MacOS用户可以在[https://github.com/muxueqz/skybook/releases](https://github.com/muxueqz/skybook/releases)上直接下载编译好的程序使用啦！

#### Linux安装
```bash
wget https://github.com/muxueqz/skybook/releases/download/1.0.3/skybook-linux -O skybook
skybook &> /dev/shm/skybook.log &
```

#### Mac安装
```bash
wget https://github.com/muxueqz/skybook/releases/download/1.0.3/skybook-osx -O skybook
skybook &> /dev/shm/skybook.log &
```

**书签数据默认会保存在当前目录的`bookmarks.db`里，大家可以针对这个文件备份**
