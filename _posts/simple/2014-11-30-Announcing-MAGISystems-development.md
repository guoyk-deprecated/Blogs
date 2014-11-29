---
layout: post
title: 'MAGI.Systems 开发中'
---

网站地址 [https://magi.systems](https://magi.systems)
Github [https://github.com/yanke-guo/magi-systems](https://github.com/yanke-guo/magi-systems)

这个项目是用来做技术验证的，里面有 Devise, Faye, Sidekiq，还用到了 Capistrano 自动化部署。

预计会加上 Omnibus。

功能上比较简单，还在做扩展。目标是基于节点的主题实时聊天+发帖。

代码按照MIT协议公开，当前部署在 Linode，没有CDN。

当前发邮件是使用 sendmail，会比较坑爹，Production 数据库依旧是 Sqlite，所以未来某天肯定是会抹掉重来的。
