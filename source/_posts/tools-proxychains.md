---
title: 工具：ProxyChains 如何在终端中使用
date: 2023-09-12 17:00:21.864
updated: 2023-09-19 12:00:58.473
url: /archives/usage-proxy-chains
categories: 技术笔记
tags: 工具使用
---


最近在阿里云上的一个服务器（CentOS）需要走梯子，需要用到两个工具：
- V2ray-Core： 这个不必多讲。和往常一样，走端口去访问外面，例如：socks5 1080
- ProxyChains：一个开源的代理工具，[ProxyChains Github Repository ](https://github.com/haad/proxychains) 
> ProxyChains is a UNIX program, that hooks network-related libc functions in dynamically linked programs via a preloaded DLL and redirects the connections through SOCKS4a/5 or HTTP proxies.


### 安装 ProxyChains
```bash
# Ubuntu
$ apt-get install proxychains 
# CentOS 
$ yum install proxychains-ng
# Mac 
$ brew install proxychains-ng
```

### 拉源码编译 ProxyChains (另一种安装方法)
```bash
$ git clone https://github.com/haad/proxychains.git
$ cd proxychains
$ ./configure
$ make
$ sudo make install
```

### 配置 Proxy 端口 

配置文件，如果没有，就创建下：`/etc/proxychains.conf`
也可以用： `proxychains -f xxx.conf 命令` 来指定。
```shell
strict_chain
proxy_dns
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000
quiet_mode
[ProxyList]
# 添加以下行，使用本地V2Ray代理，根据 v2ray 的 config 
socks5  127.0.0.1 1080
```

### 如何使用
```bash
$ proxychains4 需要走代理的命名 
# 例如： 
$ proxychains4 curl https://en.wikipedia.org/
```

### 验证下是否可以访问维基 
```bash 
[root@AliyunServer proxychains]# proxychains4  curl https://en.wikipedia.org/
[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/local/lib/libproxychains4.so
[proxychains] DLL init
[proxychains] Dynamic chain  ...  127.0.0.1:1080  ...  en.wikipedia.org:443  ...  OK
```

### 设置别名
每次都要输入 proxychains4 太麻烦，到 `~/.bash_profile` 给它声明个别名。


```bash
# proxychains4 的别名 pxy，终端输入 pxy 就是 proxychains4
$ alias pxy='proxychains4'
```

声明完成后，记得让它生效。

```bash 
$ source ~/.bash_profile

# 验证别名是否 OK 
$ pxy --help 
```


## Mac 上如何使用 
由于 MacOS 有 SIP（简单来说就是系统保护）,需要先关闭，否则无法正常使用。

> System Integrity Protection (SIP) in macOS protects the entire system by preventing the execution of unauthorized code. 

1. 在终端中查看 SIP 状态
```bash  
$ csrutil status 
```

2. 重启 Mac 进入 Recovery mode 
```bash 
$ sudo reboot
# 开机时按住： command + r ，进入 Recovery mode 
# 进入之后，上面的顶部菜单栏找到终端，输入命令即可
# csrutil --help 

# 关闭 sip 
$ csrutil disable
# 开启 sip 
# csrutil enable 
# 然后在重启 Mac 就回到正常模式了
$ reboot 
```

3. 关闭 SIP 后，就可以正常使用了

```bash 
# 查看 SIP 是否关闭  
$ csrutil status
# System Integrity Protection status: disabled.
```

```bash 
# 查看当前的 IP 归属地 
$ proxychains4 curl cip.cc 
[proxychains] config file found: /usr/local/etc/proxychains.conf
[proxychains] preloading /usr/local/Cellar/proxychains-ng/4.16/lib/libproxychains4.dylib
IP	: ***.***.***,**
地址	: 美国  加利福尼亚州  洛杉矶
运营商	: *******

数据二	: 美国 | 加利福尼亚州洛杉矶CNServers数据中心

数据三	: 美国加利福尼亚 | 层峰网络

URL	: http://www.cip.cc/***.***.***,**
```


## 学习参考 
- [Github: ProxyChains ](https://github.com/haad/proxychains)
- [About System Integrity Protection on your Mac](https://support.apple.com/en-us/102149)
- [Disabling and Enabling System Integrity Protection](https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection)

