---
title: Docker中的java程序开启JMX监控
tags: jmx
categories: docker
---
```
docker run -d \
--name spring-app \
-e "JAVA_OPTIONS=
        -Dcom.sun.management.jmxremote.port=1090 
		-Dcom.sun.management.jmxremote.rmi.port=1090   
		-Dcom.sun.management.jmxremote.ssl=false 
		-Dcom.sun.management.jmxremote.authenticate=false  
		-Djava.rmi.server.hostname=172.16.18.16
	"
--net dadi-bridge \
--init \
--restart=always spring-app
```

查看其中的JAVA_OPTIONS参数即可