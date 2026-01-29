+++
date = '2026-01-27T17:34:19+08:00'
draft = false
title = 'Week07'
+++
# Nginx
## Nginx是什么
Nginx(engine x)是一个轻量级、高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP代理服务器。

Nginx在高链接并发的情况下有不错的性能。

Nginx使用C编写，无论是系统资源开销还是CPU使用效率都不错。

Nginx安装简单，配置文件简洁（还支持perl语法），运行稳定。

## 什么是反向代理？
反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网路上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

## Nginx常用命令
常用到的命令如下：
```bash
nginx -s stop       快速关闭Nginx，可能不保存相关信息，并迅速终止Web服务
nginx -s quit       平稳关闭Nginx，保存相关信息，有安排的结束Web服务
nginx -s reload     因改变了Nginx相关配置，需要从新加载配置而重载
nginx -s reopen     重新打开日志文件
nginx -c </path/to/config>   为Nginx指定一个配置文件，来代替缺省的
nginx -t            不运行，仅仅测试配置文件，Nginx将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件
nginx -v            显示Nginx的版本
nginx -V            显示Nginx的版本，编译器版本和配置参数
```

## Nginx配置
### Nginx配置文件结构
Nginx的命令很少，只负责启停服务器等简单功能，Nginx的逻辑主要通过配置体现。

Nginx配置的基本结构：
```nginx
...         # 全局块

events {    # events块
    ...
}

http {      # http块
    ...     # http全局块
    server {    # server块
        ...     # server全局块
        location [PATTERN] {    # location块
            ...
        }
        location [PATTERN] {
            ...
        }
    }
    server {
        ...
    }
    ...
}
```

更详细内容参考Nginx中文维基[^1]。

[^1]: Nginx中文维基：[https://tool.oschina.net/apidocs/apidoc?api=nginx-zh](https://tool.oschina.net/apidocs/apidoc?api=nginx-zh)