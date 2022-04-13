---
title: 解决图像检测demo的超时问题
date: 2021-05-19 10:19:21
tags:
---

## 前言
图像检测的demo在测试时出现了在并发上去后出现`504 Gateway Time-out. The gateway did not receive a timely response from the upstream server or application.`的问题，导致识别失败，记录排查过程。

## 过程

在部署了一个测试服务器后，安装相同版本的环境时出现新的问题。python执行命令时出现`ImportError: DLL load failed`找不到指定模块，刚开始以为是缺少dll文件，想查询缺少的文件进行补充，后面发现没办法补足。进一步查找方案时发现是pip下载包时需要在管理员模式下进行。

环境解决以后问题复现不出来，查询Apache的错误日志，有查到一条`AH00326: Server ran out of threads to serve requests. Consider raising the ThreadsPerChild setting`的记录，于是在`httpd-mpm.conf`中提升了子线程的数量，但依然有超时问题。

```
<IfModule mpm_winnt_module>
ThreadsPerChild 512 
MaxRequestsPerChild 0
</IfModule>
```

查询了Apache的访问日志，发现返回的状态码都是正常的，应该是请求被拦截了。Remote Address也不是服务器的地址。
![headers](/images/202105/20210519105659.png)

应该是阿里云cdn回源请求那边超时时间的设置问题

![回源请求超时时间](/images/202105/20210519112320.png)

最后通过ip地址访问接口，绕开域名绑定的cdn回源方式解决这个问题

## 补充

如果是服务器问题的一些配置。

nginx访问出现504 Gateway Time-out，一般是由于程序执行时间过长导致响应超时，例如程序需要执行90秒，而nginx最大响应等待时间为30秒，这样就会出现超时。通常有以下几种情况导致

1.程序在处理大量数据，导致等待超时。
2.程序中调用外部请求，而外部请求响应超时。
3.连接数据库失败而没有停止，死循环重新连。

解决方法参考这篇[文章](https://blog.csdn.net/haibo0668/article/details/103869582/)
