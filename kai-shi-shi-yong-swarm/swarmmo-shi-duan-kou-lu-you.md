# Swarm模式端口路由网

Docker Engine在Swarm模式下，使service可以轻松的将端口暴露给外部资源使用。所有的节点都在一个入口路由网络中。路由网络使每个节点都可以在接受任何一个service暴露的端口连接，即使节点上并没有运行任何task。这是因为路由网会将端口上连接的请求指向到暴露该端口的一个可用节点上，并由其中一个暴露该端口的container来处理。

为了在Swarm中使用这个入口网络，需要在开启Swarm模式之前，保证节点之间在下面两个端口之间可以互相访问：

* 7946 TCP/UDP 用来实现container发现服务的端口

* 4789 UDP container入口网络端口

同时需要开放Swarm节点与外部资源的通信端口。例如，与外部负载均衡设备的通信端口。

我们也可以通过设置绕过路由网络。

## 暴露service端口

在创建service时，通过使用参数`--publish`参数来暴露端口。`target`用来指定container内部的端口号；`published`用来指定向外暴露的端口号，这个端口号将被绑定到路由网上。如果不设置`published`将会为service的task指定一个随机的端口号，需要查看task的信息才能确定这个端口号是多少。

```
$ docker service create \
  --name <SERVICE-NAME> \
  --publish published=<PUBLISHED-PORT>,target=<CONTAINER-PORT> \
  <IMAGE>
```

> **注意：**之前版本的语法使用冒号分隔。冒号前表示container内部端口号，冒号后表示对外暴露的端口号，例如`-p 8080:80`。新的语法的好处是更容易理解和更具灵活性。

Service通过Swarm向外暴露`<PUBLISHED-PORT>`才能变成可用的service。如果我们省略这个参数，service会被随机的绑定一个端口。`<CONTAINER-PORT>`端口是container监听的端口。这个参数是必须设置的。

举个例子，下面的命令将创建一个Nginx service，使container的80端口向外部暴露成8080端口，即在swarm的每一个节点上都会向外暴露8080端口。

```
$ docker service create \
  --name my-web \
  --publish published=8080,target=80 \
  --replicas 2 \
  nginx
```

当我们访问任何一个节点的8080端口时，Docker会将请求路由到一个可用的container上。在swarm的其他节点上8080端口可能并不是真正的被绑定，但是路由网知道如何进行路由通信，并且阻止端口冲突。





