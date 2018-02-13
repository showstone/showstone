---
layout: post
title: "记一次redis报错JedisDataException: ERR Proxy fail to forward command"
description: ""
category: JAVA
tags: [redis, JedisDataException, Proxy, forward, command]
---

前几天老大要求优化redis,减少redis的读取次数,用mget来替代循环多次读取redis.线下测试没有问题,等到线上却发现大量报:redis.clients.jedis.exceptions.JedisDataException: ERR Proxy fail to forward command.

google后没有发现什么有用的资料.挂上vpn连上生产环境的配置,发现程序也是正常work的.证明不是环境的问题.

以Jedis的庞大用户量,应该不会是Jedis的问题,继续进行测试,最后发现当key的个数为0时,能重现问题.

    redis.clients.jedis.exceptions.JedisDataException: ERR Proxy fail to forward command   
      at redis.clients.jedis.Protocol.processError(Protocol.java:113)   
      at redis.clients.jedis.Protocol.process(Protocol.java:138)
      at redis.clients.jedis.Protocol.read(Protocol.java:192)   
      at redis.clients.jedis.Connection.readProtocolWithCheckingBroken(Connection.java:282) 
      at redis.clients.jedis.Connection.getBinaryMultiBulkReply(Connection.java:218)
      at redis.clients.jedis.Connection.getMultiBulkReply(Connection.java:211)  

通过命令行直接操作:

  > 127.0.0.1:6379> mget   
  > (error) ERR Proxy fail to forward command

坑爹的是redis的官方文档没有关于这些内容的任何介绍:

    MGET key [key ...]

    Available since 1.0.0.
    Time complexity: O(N) where N is the number of keys to retrieve.
    Returns the values of all specified keys. For every key that does not hold a string value or does not exist, the special value nil is returned. Because of this, the operation never fails.

    Return value
    Array reply: list of values at the specified keys.
    Examples
    redis> SET key1 "Hello"
    OK
    redis> SET key2 "World"
    OK
    redis> MGET key1 key2 nonexisting
    1) "Hello"
    2) "World"
    3) (nil)
    redis> 



