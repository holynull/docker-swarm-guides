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

路由网在分配给节点的IP地址上监听向外暴露的端口。在宿主机的外部路由IP地址上，这些端口是可用的。其他的IP地址访问只在宿主机内部可用。

![Ingress network](/assets/ingress-routing-mesh.png)


通过下面的命令，我们可以为已经存在的service开放对外端口：

```
$ docker service update \
  --publish-add published=<PUBLISHED-PORT>,target=<CONTAINER-PORT> \
  <SERVICE>
```

可以通过`docker service inspect`来查看service对外暴露的端口。例如：

```
$ docker service inspect --format="{{json .Endpoint.Spec.Ports}}" my-web

[{"Protocol":"tcp","TargetPort":80,"PublishedPort":8080}]

```

在输出结果中`TargetPort`表示`<CONTAINER-PORT>`，`PublishedPort`表示`<PUBLISHED-PORT>`，service将在`PublishedPort`端口上对请求进行监听。

## 仅暴露TCP端口或者仅暴露UDP端口

在默认情况下，暴露的端口为TCP端口。你可以指定暴露的端口为UDP端口。当暴露端口时，省略协议，则默认暴露的端口为TCP协议端口。如果使用长格式语法（建议Docker 1.13+），设置参数`protocol`为`tcp`或者`udp`即可。

### TCP

#### 长格式语法

```
$ docker service create --name dns-cache \
  --publish published=53,target=53 \
  dns-cache
```

#### 短格式语法

```
$ docker service create --name dns-cache \
  -p 53:53 \
  dns-cache
```

### TCP/UDP

#### 长格式语法

```
$ docker service create --name dns-cache \
  --publish published=53,target=53 \
  --publish published=53,target=53,protocol=udp \
  dns-cache
```

#### 短格式语法

```
$ docker service create --name dns-cache \
  -p 53:53 \
  -p 53:53/udp \
  dns-cache
```

### UDP

#### 长格式语法

```
$ docker service create --name dns-cache \
  --publish published=53,target=53,protocol=udp \
  dns-cache
```

#### 短格式语法

```
$ docker service create --name dns-cache \
  -p 53:53/udp \
  dns-cache
```

## 绕过路由网

当需要每次通过端口访问，都指定访问一个固定节点上service的实例时，我们可以通过设置绕过路由网。这就涉及到一个概念，称之为`host`模式。这种情况下，我们需要记住一下几点：

- 如果我们访问到一个没有运行service实例的节点时，可能会链接失败或者访问到一个完全不同的service实例上。

- 如果期望在每个节点上运行同一个service的多个task，就不能指定一个静态的`target`端口。也不能允许Docker随机分配暴露的端口。通过创建全局service而不是副本模式的service，或者使用`placement constraints`约束，来确保在指定的节点上运行一个单独的service实例。

要绕过路由网，必须使用`--publish`，并且设置`mode`参数为`host`。如果省略`mode`，或者设置`mode`为`ingress`，则不会绕过路由网。下面的命令创建了一个`host`模式的全局service，并绕过路由网：

```
$ docker service create --name dns-cache \
  --publish published=53,target=53,protocol=udp,mode=host \
  --mode global \
  dns-cache
```

## 使用外部负载均衡







