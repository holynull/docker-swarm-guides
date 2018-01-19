# 强行移除节点

有些情况下，需要在移除节点之前关闭节点。如果节点变成不可达，或者没有响应。我们可以像如下方法使用`--force`参数强行移除节点。

```
$ docker node rm node9

Error response from daemon: rpc error: code = 9 desc = node node9 is not down and can't be removed

$ docker node rm --force node9

Node node9 removed from swarm
```

强行移除节点之前，必须将节点先降级成为worker节点。降级Manager节点时，请确保剩下的manager个数为一个奇数。

