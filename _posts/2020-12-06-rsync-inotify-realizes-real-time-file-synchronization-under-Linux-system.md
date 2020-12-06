---
layout: post
title: 	使用rsync+inotify实现Linux系统下文件实时同步
categories: FileSync
description: 
keywords: rsync,inotify,Linux,文件同步,文件实时同步
---

> 
>

### 0x00  准备工作

```flow
st=>start: Start:>https://www.zybuluo.com
io=>inputoutput: verification
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
```
