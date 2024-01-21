---
title: gcc 和 g++
date: 2023-10-23 11:46:55.241
updated: 2023-10-23 14:45:57.375
url: /archives/gcc-and-g
categories: 技术笔记
tags: 编程语言
---



> 用 gcc 直接编译 cpp，报错： undefined reference to std::cout

在网上搜了下，在 StackOverflow 中找到答案：建议用**g++** 来编译 cpp，原来是 `gcc 需要手动链接 C++ 的库`，而 `g++ 会自动链接 C++ 的库`。

- 使用 gcc 编译并且指定标准库（-lstdc++）：
```bash 
$ gcc -o main main.cpp -lstdc++
```

- 使用 g++ 编译：
```bash
$ g++ -o main main.cpp
```

g++ 等同于 `gcc -xc++ -lstdc++ -shared-libgcc`
- -x: 指定语言
- -l: 链接库 
- -shared-libgcc: 使用共享的 libgcc 


##  gcc 和 g++ 有何不同呢 ？

gcc 和 g++ 都是 GUN Complier Collection 的一部分，gcc 是 GNU Compiler Collection 的 C 语言编译器（主要），而 g++ 是 GNU Compiler Collection 的 C++ 语言编译器。

- gcc 和 g++ 都支持 `*.c` 和 `*.cpp` 文件的编译，g++ 和 gcc 绝大多数都相同。 
- g++ 编译更倾向于 `*.cpp` 文件，它会自动链接 `std c++` 库。gcc 编译倾向于 `*.c` 文件，如果需要编译 `*.cpp` 需要指定 `std c++` 库

## 学习参考 
- [Stackoverflow：gcc undefined reference to `std::ios_base::Init::Init()' ](https://stackoverflow.com/questions/18698059/gcc-undefined-reference-to-stdios-baseinitinit)
- [Stackoverflow: What is the difference between g++ and gcc?](https://stackoverflow.com/questions/172587/what-is-the-difference-between-g-and-gcc)
- [GCC-Online-Docs: 3.15 Options for Linking](https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html)