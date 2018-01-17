# Task的状态

Service是应用运行的理想状态的描述，task在这个理想状态下完成工作。工作按照下面的流程在Swarm节点之间被调度：

1. 使用CLI运行命令`docker service create`，或者使用UCP web界面。

2. 请求传递给manager节点。

3. manager节点在特定的节点调度service的运行。

4. 每一个service可以由多个task来执行。

5. 每一个task都有一个生命周期，生命周期的状态包括：`NEW`，`PENDING`和`COMPLETE`等等。

Task是一次执行单元。当task停止，就不会再被执行，除非一个新的task会取代它。

在task执行完成或者失败之前，task会通过一系列的状态变化。task由`NEW`状态初始化。task的状态变化过程是不可逆的。例如，一个task是永远不会从`COMPLETE`状态变回`RUNNING`状态的。

Task的状态如下表：

|状态|描述|
|---|---|
|`NEW`|初始化状态|
|`PENDING`|资源分配了任务时的状态|
|`ASSIGNED`|task被分配到节点后的状态|
|`ACCEPTED`|task被worker节点接受后的状态。|
|`PREPARING`|Docker正在准备task|
|`STARTING`|Docker启动task|
|`RUNNING`|正在运行中的状态|
|`COMPLETE`|task已经存在，并且没有错误码|
|`FAILED`|task已经存在，但是有错误码出现|
|`SHUTDOWN`|Docker被请求关闭task|
|`REJECTED`|worker节点拒绝接受task|
|`ORPHANED`|节点离线时间超长|

## 查看状态

运行命令`docker service ps <service-name>`来获得task的状态。`CURRENT STATE`表示task的状态：

```
$ docker service ps webserver
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR                              PORTS
owsz0yp6z375        webserver.1         nginx               UbuntuVM            Running             Running 44 seconds ago                                      
j91iahr8s74p         \_ webserver.1     nginx               UbuntuVM            Shutdown            Failed 50 seconds ago    "No such container: webserver.…"   
7dyaszg13mw2         \_ webserver.1     nginx               UbuntuVM            Shutdown            Failed 5 hours ago       "No such container: webserver.…"  
```


