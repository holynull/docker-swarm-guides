# 安全(PKI)

Docker内置了PKI，使保障发布容器化的业务流程系统的安全性变得很简单。Swarm节点之间采用TLS来鉴权、授权和加密通信。

当我们运行`docker swarm init`命令时，Docker指定当前节点为一个manager节点。默认情况下，manager节点会生成一个新的根CA证书以及一对密钥。CA证书和密钥将会被用在节点之间的通信上。也可以通过参数`--external-ca`来指定其他CA证书。详情请参阅[Docker swarm init](https://docs.docker.com/engine/reference/commandline/swarm_init/)

Manager节点会生成两个token，一个用来添加worker节点，一个用来添加manager节点。每一个token包含了CA证书的digest和一个随机密钥。当节点加入到Swarm中时，加入的节点将digest发送给远端的manager节点，由manager节点来验证申请加入节点的根CA证书是否合法。远端manager节点通过随机密钥来验证申请加入的节点是否被核准加入。

每当节点加入到Swarm中后，manager都会给节点发送一个证书。这个证书中包含了一个随机生成的节点ID，用来标识这个证书通用名称下的节点，以及组织单元下的角色。在节点的当前Swarm生命周期中，节点ID是节点的安全加密的身份标识。

下图说明了manager节点和worker节点如何使用TSL 1.2进行通信加密的。

![](/assets/tls.png)

下面的例子展示了一个worker节点获得证书内容：

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            3b:1c:06:91:73:fb:16:ff:69:c3:f7:a2:fe:96:c1:73:e2:80:97:3b
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN=swarm-ca
        Validity
            Not Before: Aug 30 02:39:00 2016 GMT
            Not After : Nov 28 03:39:00 2016 GMT
        Subject: O=ec2adilxf4ngv7ev8fwsi61i7, OU=swarm-worker, CN=dw02poa4vqvzxi5c10gm4pq2g
...snip...
```

默认情况下，每个节点的证书3个月轮换一次。我们可以通过配置修改的这个间隔：`
docker swarm update --cert-expiry <TIME PERIOD>`。最小的证书轮换时间为1小时。详情请参考[docker swarm update](https://docs.docker.com/engine/reference/commandline/swarm_update/)

## CA证书轮换

如果Swarm的CA密钥或Manager节点受到危害，您可以轮换Swarm的根CA证书，以使所有节点都不再信任旧的根CA签名的证书。

运行命令`docker swarm ca --ratate`来生成新的CA证书和密钥。也可以通过参数`--ca-cert`和`--external-ca`来指定外部的根CA证书。

当运行了`docker swarm ca --rotate`命令后，会按顺序发生下面的事情：

1. Docker会生成一个交叉签名（cross-signed）证书。即新证书是由旧的证书签署的。这个交叉签名证书将作为一个过渡性的证书。这是为了确保节点仍然能够信任旧的证书，也能使用新的CA证书验证签名。

2. 在Docker 17.06或者更高版本中，Docker会通知所有节点立即更新TLS证书。根据Swarm中节点的数量多少，这个过程可能会花费几分钟时间。

> **注意：如果Swarm中的节点上Docker的版本不一致，将会发生下面的情况：**
- 只有manager节点运行了Docker 17.06或者更高版本，并且作为leader时，才能通知到其他节点更新TLS证书。
- 只有Docker 17.06或者更高版本才会支持这个指令。
最好确保所有的Swarm节点上运行Docker 17.06或者更高版本。

3. 在所有的节点都更新了新CA证书签署的TLS证书后，Docker将不在信任旧的证书，并通知所有节点仅信任新的CA证书。

    加入Swarm使用的token将发生变化，旧的token将不可用。
    
从这时起，所有新的由新根CA证书签署的节点证书将颁发给节点，并且完全不存在过度内容。













