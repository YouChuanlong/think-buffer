---
title: Mac 上用 clang 编译找不到标准库
date: 2024-01-16 20:46:40.719
updated: 2024-01-16 21:07:53.699
url: /archives/mac-clang-stdioh-error
categories: 技术笔记
tags: 工具使用
---

# clang error: 'stdio.h' file not found 

前一段时间碰到这问题是在 GoLand 编译 Go 语言项目时报的找不到标准库。在终端用 clang 来编译也如此。


> 我的系统版本
```bash
$ sw_vers

ProductName:	macOS
ProductVersion:	12.7.2
```

> 编译 go 项目时 cgo 报的错误：
```bash
# runtime/cgo
_cgo_export.c:3:10: fatal error: 'stdlib.h' file not found
```

> 在终端使用 clang 编译 c 文件：
```c
#include<stdio.h>
int main() {
    printf("Hello\n");
    return 0;
}
// file: main.c 
```


```shell
cc main.c  -o main
main.c:1:9: fatal error: 'stdio.h' file not found
#include<stdio.h>
        ^~~~~~~~~
1 error generated.
```



网上大部分的方法是去重新让 xcode 重装命令工具，对我来说没有解决此问题。

最后找到解决方法是在添加一个 SDKROOT 环境变量：[帖子地址（Stackoverflow: Cannot find stdio.h）](https://stackoverflow.com/questions/51761599/cannot-find-stdio-h)


### 方法一：添加环境变量
先试下是否可行，能解决再添加到 `~/.bash_profile`
```bash 
$ export SDKROOT=$(xcrun --sdk macosx --show-sdk-path)
```


#### 分析下变量的作用

```bash
$ xcrun --sdk macosx --show-sdk-path

/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX12.3.sdk
```
关于 `xcrun` 的介绍： 
>  xcrun - Run or locate development tools and properties


关于 `$SDKROOT` ChatGPT 的介绍： 
>   在 macOS 系统中，SDKROOT 是一个环境变量，用于指定当前正在使用的软件开发工具包（SDK）的根目录路径。
> 通常，SDKROOT 的值是 macOS 系统上安装的 Xcode 工具包的路径。Xcode 是苹果提供的开发工具套件，包括编译器、调试器、接口构建工具等，用于开发 macOS 和 iOS 应用程序。      


大致的原因是： `/usr/` 目录下没有 `include/` 文件。上次我有试过去将 `/usr/local/include` 链接到 `/usr/` 目录下，由于 macOS 对此目录为只读权限，所以无法链接。



### 方法二：xcode 重新安装一遍命令行工具

该方法对我来说并没有解决，不过可以试下：


```bash
# 删除之前的命令行工具
$ sudo rm -rf /Library/Developer/CommandLineTools
```


```bash
# 安装 
$ xcode-select --install
```


## 学习参考
- [Stackoverflow: Cannot find stdio.h](https://stackoverflow.com/questions/51761599/cannot-find-stdio-h)
- [Github: runtime/cgo: fails to build after updating to Mojave](https://github.com/golang/go/issues/27921)
- [Stackoverflow: SDKROOT path for latest SDK ](https://stackoverflow.com/questions/13964742/sdkroot-path-for-latest-sdk)

