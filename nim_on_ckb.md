Title: 在CKB上运行nim程序
Date: 2020-06-09 22:20
Tags: nim
Category: nim
Slug: nim-on-ckb
Author: muxueqz

## 前言
众所周知，Nervos 底层公链 CKB 的虚拟机（[CKB-VM](https://github.com/nervosnetwork/ckb-vm)）是基于 RISC-V 指令集打造的区块链虚拟机，
其中一个特性就是"任何可以编译成 RISC-V 指令集的语言均可以直接用来为 CKB 开发智能合约"，

理论上只要支持CKB VM里的syscall就可以作为合约的开发语言，又看到 https://hookrace.net/blog/nim-binary-size/ 里提到如何精简nim编译出来的二进制文件与systemcall，
因此，我就在想，是不是可以用nim编译出可以在CKB VM上运行的程序呢？
想到就去做。

## 准备工作
```bash
# Debug工具
docker pull muxueqz/ckb-standalone-debugger-plus:latest
# Nim工具链
docker pull muxueqz/ckb-nim-builder:latest
# nim ckb syscall
git clone https://github.com/muxueqz/ckb-nim-builder.git --recurse-submodules
```

## 编写测试合约

用Nim编写合约的代码如下：
```nim
import ckb_syscall


proc main: clong {.exportc: "_start".} =
   ckb_debug("hello world")
   ckb_exit 0
```

这份代码在`ckb-nim-builder`已经有了：`example.nim`

## 接下来我们把它编译成链上可运行的二进制：
```bash
cd ckb-nim-builder
docker run -w /app -v $PWD/:/app -it --rm muxueqz/ckb-nim-builder bash -x /app/build.sh example.nim
```

## 测试编译出来的二进制能不能运行
```bash
docker run -w /data/ -v $PWD:/data  -it --rm muxueqz/ckb-standalone-debugger-plus:latest /data/example 0x10000000
```

如果得到的结果如下，那就说明可以正常运行了
```
Executing: /opt/debugger/ckb-debugger --tx-file tx.json --script-group-type type -i 0 -e input
DEBUG:<unknown>: script group: Byte32(0xe858b26d459d418fa5f78a8dd6c639cb7714942816caacb9749830e5203f31ac) DEBUG OUTPUT: hello world
Run result: Ok(0)
Total cycles consumed: 1091
Transfer cycles: 56, running cycles: 1035
```


## 参考
* https://github.com/nervosnetwork/ckb-vm/blob/416bb211abea1ef79ffa5ce3b14706044e65cd2a/src/machine/mod.rs
* https://nim-lang.org/docs/nimc.html#nim-for-embedded-systems
* https://github.com/jjyr/ckb-std/blob/master/test/contract/src/main.rs
* https://zhuanlan.zhihu.com/p/95982790
* https://medium.com/nervosnetwork/intro-to-ckb-script-programming-2-script-basics-85a593720d8f
* https://livebook.manning.com/book/nim-in-action/chapter-8/35
* https://xuejie.space/2020_03_03_introduction_to_ckb_script_programming_performant_wasm/
