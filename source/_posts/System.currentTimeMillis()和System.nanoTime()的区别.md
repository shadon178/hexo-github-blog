---
title: System.currentTimeMillis()和System.nanoTime()的区别
tags: currentTimeMillis nanoTime
categories: java
---

# System.currentTimeMillis()
返回的是1970 年 1 月 1 日午夜到目前的毫秒数，建议主要用于获取时间，而不要用于计时，因为在计时的过程中，时间可能受NTP（时间服务器）的影响产生计时误差。

# System.nanoTime()
返回的是一个会不断自增的、精确计时的纳秒数，不受NTP的影响，因此不能用于获取时间，但是非常适合用于精确的计时场景。

# 总结
1、System.currentTimeMillis()适合获取时间的场景；
2、System.nanoTime()适合计时的场景；