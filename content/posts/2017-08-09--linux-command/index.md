---
title: linux命令随记
subTitle: linux
category: Dialy
cover: 5.png
---

- 检索日志 : grep  -A 100 'xxxxxx' servelet.log

- 检索当前线程: ps -ef | grep java 

- 查看当前线程:ps -aux

- 切割日志:grep "$(date+"%Y-%m-%d")" servelet.log >today.log

