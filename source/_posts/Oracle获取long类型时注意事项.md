---
title: Oracle Jdbc获取long类型的数据时注意事项
tags: oracle long
categories: jdbc
---


通过Jdbc获取long类型的数据后，就无法再次获取查询结果数据，因为long类型获取之后就会导致结果流被关闭，因此一般将long类型的结果在最后才进行获取，否则就无法获取到其他类型的数据了。



值得大家注意！

