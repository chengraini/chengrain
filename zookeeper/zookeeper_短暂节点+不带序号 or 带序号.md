# 创建短暂节点（短暂节点 + 不带序号 or 带序号）

> 创建短暂的不带序号的节点
```java
[zk: localhost:2181(CONNECTED) 7] create -e /sanguo/wuguo
“zhouyu”
Created /sanguo/wuguo
```

> 创建短暂的带序号的节点
```java
[zk: localhost:2181(CONNECTED) 2] create -e -s /sanguo/wuguo
“zhouyu”
Created /sanguo/wuguo0000000001
```

>在当前客户端是能查看到的
```
zk: localhost:2181(CONNECTED) 3] ls /sanguo
[wuguo, wuguo0000000001, shuguo]
```

>退出当前客户端然后再重启客户端
```
[zk: localhost:2181(CONNECTED) 12] quit
[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkCli.sh
```

>再次查看根目录下短暂节点已经删除
```
[zk: localhost:2181(CONNECTED) 0] ls /sanguo
[shuguo]
```

>修改节点数据值
```
[zk: localhost:2181(CONNECTED) 6] set /sanguo/weiguo “simayi”
```