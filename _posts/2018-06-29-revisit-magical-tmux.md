---
layout: post
title: "再遇神奇的Tmux"
tagline: ""
description: "时隔1年多, 再次遇到`Tmux`, 倍感亲切!"
date: '2018-06-29 13:53:45 +0800'
category: linux
tags: tmux iTerm2 linux
---
> {{ page.description }}

![](http://p9fggfk3y.bkt.clouddn.com/20180629060349_tmux.png){:width="100%"}
先来张效果图, 最下面的一行为 `Tmux` 的会话

# iTerm2
`mac` 自带的 `Terminal` 比较弱, 通常都是会安装 `iTerm2` 的

`iTerm2` 对应管理多台服务器也是比较方便的, 通常情况下就已经够用了; 经常是会打开多个 `Tab` 窗口; 对 `mac` 非常友好, 使用方便

# Tmux
`tmux` 是一款终端复用命令行工具，一般用于 Terminal 的窗口管理

恢复上次会话是 `tmux` 的拿手绝活; 走的时候是什么样, 回来的时候就是什么样! 分屏功能也很溜的

终端就不行了, 假如终端开了4个Tab窗口看日志, 一不小心终端退出了, 那么就得挨个Tab打开, 然后挨个打开日志查看

最近家里网络极其不稳定, 终端连远程主机几分钟就掉线, 有时甚至不到1分钟就掉了... 心态爆炸... 

再次使用 `tmux` 之后, 至少很快可以恢复到之前的工作状态 (2步搞定:1.链接终端 2.恢复上次的tmux会话)

# 安装 Tmux
步骤请看文末最后2篇文章

# Tmux 使用/快捷键
首先要感谢人家的整理 :)
{% gist ryerh/14b7c24dfd623ef8edc7 %}

---
参考：
- [首遇:神奇的Tmux](https://xu3352.github.io/linux/2017/02/14/magical-tmux)
- [十分钟学会 tmux](https://www.cnblogs.com/kaiye/p/6275207.html)
- [Linux终端复用神器-Tmux使用梳理](https://www.cnblogs.com/kevingrace/p/6496899.html)
