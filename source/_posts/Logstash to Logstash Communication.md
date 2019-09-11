---
title: Logstash to Logstash Communication
date: 2019-09-11 17:17
tags:
- ELK
---

# Logstash to Logstash Communication

> 实现logstash与logstash之间的通信需要通过lumberjack插件实现。



### 1.生成证书

执行openssl req -x509 -days 3560 -batch -nodes -newkey rsa:2048 -keyout lumberjack.key -out lumberjack.cert -subj /CN=localhost

生成一个lumberjack.cert 和一个lumberjack.key

> 使用x509方式创建证书默认有效时间是30天，使用days设置过期时间



### 2.保存证书

将lumberjack.cert保存在upstream logstash所在服务器（docker服务的话就保存在容器中），将lumberjack.key保存在downstream logstash所在服务器（docker服务的话就保存在容器中）。

以docker容器的方式启动logstash：

把私钥和证书保存在同一路径下{local_lumberjack_path}

```
docker-compose.yml中
volumes:
    {local_lumberjack_path}: /usr/share/logstash/lumberjack 将私钥、证书放入容器内部
```

### 3.logstash 配置

建立链接时，upstream logstash使用lumberjack作为output，downstream logstash使用beats作为input，开启ssl认证。

```
upstream logstash:
output{
    lumberjack {
        port => 5001
        host => "**********"
        ssl => true
        ssl_certificate => "/usr/share/logstash/lumberjack/lumberjack.cert"
    }
}
```

```
downstream logstash:
input{
        beats {
            port => 5001
            codec => json
            ssl => true
            ssl_certificate => "/usr/share/logstash/lumberjack/lumberjack.cert"
            ssl_key => "/usr/share/logstash/lumberjack/lumberjack.key"
        }
}
```

> 在本项目中，upstream logstash位于aws服务器上，upstream logstash位于广东跳板机上，两台机器通过ssh_tunnel进行通信，端口为5001

> upstream logstash的数据由logspout（采集服务的日志）提供，logspout部署在aws swarm上，向upstream开启的监听端口5001发送数据

> 数据流向：switch -> logspout -> upstream logstash ---ssh_tunnel+lumberjack---> downstream logstash
