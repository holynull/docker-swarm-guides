# 查看service的状态

在swarm中部署了service之后，我们就可以通过Docker CLI来查看正在运行的service的状态。

1. 通过ssh登录到manager节点上。

2. 运行命令`docker service inspect --pretty <SERVICE-ID>`查看service的详细信息。

 我们来看一下`helloworld`服务的详细信息：

 ```
 [manager1]$ docker service inspect --pretty helloworld

 ID:		 9uk4639qpg7npwf3fn2aasksr
 Name:		helloworld
 Service Mode:	REPLICATED
  Replicas:		1
 Placement:
 UpdateConfig:
  Parallelism:	1
 ContainerSpec:
  Image:		alpine
  Args:	ping docker.com
 Resources:
 Endpoint Mode:  vip
 ```
 
 > 提示：如果需要返回json格式的数据，去掉参数`--pretty`就可以了。
 
    ```
     [manager1]$ docker service inspect helloworld
    [{
        "ID": "9uk4639qpg7npwf3fn2aasksr",
        "Version": {
            "Index": 418
        },
        "CreatedAt": "2016-06-16T21:57:11.622222327Z",
        "UpdatedAt": "2016-06-16T21:57:11.622222327Z",
        "Spec": {
            "Name": "helloworld",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "alpine",
                    "Args": [
                        "ping",
                        "docker.com"
                    ]
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "MaxAttempts": 0
                },
                "Placement": {}
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 1
                }
            },
            "UpdateConfig": {
                "Parallelism": 1
            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {}
        }
    }]
    ```
 
3. 通过运行命令``来查看service在哪些节点上运行。

    ```
    [manager1]$ docker service ps helloworld

    NAME                                    IMAGE   NODE     DESIRED STATE  LAST STATE
    helloworld.1.8p1vev3fq5zm0mi8g0as41w35  alpine  worker2  Running        Running 3 minutes
    ```

