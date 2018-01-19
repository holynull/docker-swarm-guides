# Manager节点配置静态IP地址

当初始化Swarm是，我们指定参数`--advertise-addr`来广播当前节点的IP地址给其他Manager节点。由于manager节点作为一个稳定的基础组件，我们应该给它分配一个固定的静态IP地址，以防止服务器重启造成IP地址改变。

如果Swarm重启，并且所有的manager节点都因为重启而获得一个新的IP地址，其他节点就无法链接到manager节点。Swarm就会出现故障被挂起。

Worker节点可以使用动态IP地址。

