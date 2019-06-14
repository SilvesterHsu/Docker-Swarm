## 集群（`swarm`、`node`）

### 1. 创建一个swarm集群

在`manager`上初始化新的swarm集群：

```bash
docker swarm init --advertise-addr <MANAGER1-IP>
```

其中`<MANAGER1-IP>`是`manager`的ip地址。

### 2. 加入swarm集群

通过`manager`上展示的命令加入swarm集群。

查询`manager`的加入方法：

```bash
docker swarm join-token manager
```

查询`worker`的加入方法：

```bash
docker swarm join-token worker
```

会得到类似的结果：

```bash
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-39cl9rb3w70jht6qmvwzl5vteupvltcc7qksb2xpjuvyq2x8cf-a12rvi6y7ff39yj7lydexse70 172.245.135.174:2377
```

在`worker`节点上输入相应命令即可加入swarm集群。

> `worker`加入集群后，我们不用再管它，所有命令操作均在`manager`上运行。

### 3. 集群检查

在`worker`加入集群后，可通过`docker node ls`来在`manager`上查看节点信息：

```bash
$ docker node ls
ID                            HOSTNAME                 STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
cy4b158l15cdw8qcenb08upww *   vmirdos6.hostflyte.com   Ready               Active              Leader              18.09.6
p5hy0c1ez2eauvw1molg9nck2     vultr.guest              Down                Active                                  18.09.5
```

### 4. 上、下线节点

管理节点会把任务分配给ACTIVE的节点，所有ACTIVE的节点都能接到任务。下线节点使节点不会接受新任务，管理节点会停止该节点上的任务，分配到别的ACTIVE的节点上。

> **注意：**下线一个节点不移除节点中的独立容器，如`docker run`，`docker-compose up`或`docker API`启动的容器都不会删除。节点的状态仅影响集群服务的负载是否分到该节点。

#### 下线

运行`docker node update --availability drain <NODE-ID>`来下线一个节点：

先查看`<NODE-ID>`：

```bash
$ docker node ls
ID                            HOSTNAME                 STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
cy4b158l15cdw8qcenb08upww *   vmirdos6.hostflyte.com   Ready               Active              Leader              18.09.6
```

下线节点：

```bash
$ docker node update --availability drain cy4b158l15cdw8qcenb08upww
cy4b158l15cdw8qcenb08upww
```

查看状态：

```bash
$ docker node ls
ID                            HOSTNAME                 STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
cy4b158l15cdw8qcenb08upww *   vmirdos6.hostflyte.com   Ready               Drain               Leader              18.09.6
```

#### 上线

运行`docker node update --availability active <NODE-ID>`来上线一个节点：

上线节点：

```bash
$ docker node update --availability active cy4b158l15cdw8qcenb08upww
cy4b158l15cdw8qcenb08upww
```

查看状态：

```bash
$ docker node ls
ID                            HOSTNAME                 STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
cy4b158l15cdw8qcenb08upww *   vmirdos6.hostflyte.com   Ready               Active              Leader              18.09.6
```

