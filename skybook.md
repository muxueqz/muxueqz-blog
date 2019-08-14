Title: 一个用Nim语言实现的轻量级书签管理器
Date: 2019-08-13 18:20
Tags: nim
Slug: skybook
Author: muxueqz
Summary: 轻量级的美味书签替代品

[https://github.com/muxueqz/skybook](https://github.com/muxueqz/skybook)

## 特点
* 搜索功能：借助浏览器的自定义搜索引擎功能
* 存储为json文件，可以支持git版本化管理和备份到Github上
* 本地运行，不依赖网络服务，也不用担心服务器被关

## 快速开始
* 安装好Nim编译器，如果你用Debian GNU/Linux，`apt install nim`即可
* 安装
```bash
nimble install skybook
skybook &> /dev/shm/skybook.log &
```
**书签数据默认会保存在当前目录的`bookmarks.db`里，大家可以针对这个文件备份**

## 截图
![](https://raw.githubusercontent.com/muxueqz/skybook/master/screenshot.jpg)
