---
title: 使用pm2维护Laravel队列
date: 2021-05-11 14:58:04
tags: pm2 PHP Laravel
---

## 前言

当前维护着的CTC效能平台项目需要升级，把几个子程序规整到一个平台上进行管理。以`web管理平台->PHP后端->C++数据层->子程序`形式组成，php后端与C++数据层中的通信才用socket协议。php的短生命周期，目前使用Laravel框架且不使用swoole的情况下采用框架内队列（queue）。队列进程是长生命周期的进程，socket获取C++提供的最新数据。

## 问题

但出现几种问题需要解决

- artisan命令是放在bat批处理文件内执行的，出错中止无法维护
- php的socket_read()每次执行占用了较大的内存并且没有释放，导致内存溢出

## 解决方案

Laravel文档内的方案是使用Supervisor进行进程守护，但由于服务器是window系统，所以决定用pm2。

先全局安装pm2
```bash
npm i pm2 -g 
```

安装pm2的windows服务，之后就可以在windows服务内多出一个pm2服务
```bash
npm i pm2-windows-service -g
pm2-service-install -n pm2
```


接下来需要配置需要执行的命令，参考这篇[文章](https://subashbasnet.com.np/how-to-run-and-monitor-laravel-queue-using-pm2/)

```yaml
#laravel-queue-worker.yml
apps:
  - name: laravel-queue-worker
    script: artisan
    exec_mode: fork
    interpreter: php
    instances: 1
    args:
      - queue:work
      - --tries=5
      - --sleep=1
      - --delay=3

```

内存泄漏的问题原计划是用设置jobs执行数量上限，退出后由pm2重启。但需要Laravel8才能支持，Laravel8还需要php7.3以上的版本支持，升级成本较高。
