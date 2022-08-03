# 创建带序号的节点（永久节点 + 带序号）

> 先创建一个普通的根节点/sanguo/weiguo
```java
[zk: localhost:2181(CONNECTED) 1] create /sanguo/weiguo
“caocao”
Created /sanguo/weiguo
```

> 创建带序号的节点
```java
[zk: localhost:2181(CONNECTED) 2] create -s
/sanguo/weiguo/zhangliao “zhangliao”
Created /sanguo/weiguo/zhangliao0000000000
```

```java
[zk: localhost:2181(CONNECTED) 3] create -s
/sanguo/weiguo/zhangliao “zhangliao”
Created /sanguo/weiguo/zhangliao0000000001
```

```java
[zk: localhost:2181(CONNECTED) 4] create -s
/sanguo/weiguo/xuchu “xuchu”
Created /sanguo/weiguo/xuchu0000000002
```

```
如果原来没有序号节点，序号从 0 开始依次递增。如果原节点下已有 2 个节点，则再排
序时从 2 开始，以此类推。
```