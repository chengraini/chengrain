# 节点删除与查看

> 删除节点
```java
[zk: localhost:2181(CONNECTED) 26] get -w /sanguo
```

> 递归删除节点
```java
[zk: localhost:2181(CONNECTED) 1] set /sanguo “xisi”
```

> 查看节点状态
```java
[zk: localhost:2181(CONNECTED) 17] stat /sanguo
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000011
mtime = Wed Aug 29 00:21:23 CST 2018
pZxid = 0x100000014
cversion = 9
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 1

```
