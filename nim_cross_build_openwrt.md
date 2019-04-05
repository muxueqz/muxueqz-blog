Title: 在Linux上跨平台编译OpenWRT路由器上的Nim程序
Date: 2019-04-05 13:20
Tags: openwrt, nim
Category: nim
Slug: nim_on_openwrt
Author: muxueqz
Summary: Nim是个很好玩儿的语言，我们来看看怎么在路由器上运行Nim编写的程序

[Nim](http://nim-lang.org/) 是个很好玩儿的语言，我们来看看怎么在路由器上运行Nim编写的程序

## 准备工作
* 安装好Nim编译器，如果你用Debian GNU/Linux，`apt install nim`即可
* 一台mipsel的OpenWRT路由器

## 写个Nim程序
来个Hello World：
```nim
echo "hello world"
```

## 准备OpenWRT的编译工具链
```bash
aria2c 'http://archive.openwrt.org/chaos_calmer/15.05.1/ramips/mt7621/OpenWrt-SDK-15.05.1-ramips-mt7621_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64.tar.bz2'
tar xvf OpenWrt-SDK-15.05.1-ramips-mt7621_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64.tar.bz2
```

## 配置PATH环境变量，让Nim编译器找到编译工具链
```bash
export PATH=/dev/shm/OpenWrt-SDK-15.05.1-ramips-mt7621_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64/staging_dir/toolchain-mipsel_1004kc+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin/:$PATH
```

## 开始编译吧！
```bash
nim -d:release --opt=size -d:ssl --cpu:mipsel --os:linux --gcc.exe:mipsel-openwrt-linux-gcc --gcc.linkerexe:mipsel-openwrt-linux-gcc  c  helloworld.nim
```

在我的机器上编译之后187KB，对比Golang编译出来的十几MB，真是轻量太多了，

如果还想再小，可以用`upx`神器压缩一下：
```bash
upx -9 --best --lzma helloworld
```

这样就只要70KB了，对于只有几MB ROM的路由器，真是福音
