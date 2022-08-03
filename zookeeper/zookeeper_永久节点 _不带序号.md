# 节点类型（持久/短暂/有序号/无序号）

>分别创建2个普通节点（永久节点 + 不带序号）

```
[zk: localhost:2181(CONNECTED) 3] create /sanguo “diaochan”
Created /sanguo
[zk: localhost:2181(CONNECTED) 4] create /sanguo/shuguo
“liubei”
Created /sanguo/shuguo

注意：创建节点时，要赋值
```

>获得节点的值
```
[zk: localhost:2181(CONNECTED) 5] get -s /sanguo
diaochan
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000003
mtime = Wed Aug 29 00:03:23 CST 2018
pZxid = 0x100000004
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1

```

```
[zk: localhost:2181(CONNECTED) 6] get -s /sanguo/shuguo
liubei
cZxid = 0x100000004
ctime = Wed Aug 29 00:04:35 CST 2018
mZxid = 0x100000004
mtime = Wed Aug 29 00:04:35 CST 2018
pZxid = 0x100000004
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0

```

