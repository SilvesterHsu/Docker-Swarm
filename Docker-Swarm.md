# Docker Swarm

有关集群的docker命令如下：

1. **docker swarm**：集群管理，子命令有init, join, join-token, leave, update
2. **docker node**：节点管理，子命令有demote, inspect,ls, promote, rm, ps, update
3. **docker service**：服务管理，子命令有create, inspect, ps, ls ,rm , scale, update
4. **docker stack**：试验特性，用于多应用部署

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

## 服务（`service`）

### 1. 在swarm集群上部署服务

#### 建立服务

在`manager`上运行如下命令：

```bash
docker service create --replicas 1 --name helloworld alpine ping docker.com
```

* docker service create 用来创建服务
* —name 表明服务名字是helloworld
* —replicas 表示期望1个服务实例
* alpine ping docker.com 表示运行镜像是alpine，命令是ping

> 可以通过以下方法在指定节点运行服务
>
> ```bash
> --constraint=node.HOSTNAME==<NODE-ID>
> ```

#### 查看服务

* 运行`docker service ls`来查看运行的服务：

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                        PORTS
rajn4sso6fd0        hello               replicated          1/1                 hello-world:latest
03niz8ebse57        portainer           replicated          1/1                 portainer/portainer:latest   *:9000->9000/tcp
```

* 运行`docker service ps <CONTAINER>`来查看任务：（在服务中运行的容器称为“任务”）

```bash
$ docker service ps jupyter
ID                  NAME                IMAGE                         NODE                        DESIRED STATE       CURRENT STATE          ERROR               PORTS
ms2zv5wsyvg2        jupyter.1           silvesterhsu/jupyter:latest   vmirdos6.hostflyte.com      Running             Running 2 hours ago
om3wofnemrph         \_ jupyter.1       silvesterhsu/jupyter:latest   p5hy0c1ez2eauvw1molg9nck2   Shutdown            Orphaned 2 hours ago
```

* 运行`docker service inspect <CONTAINER>`来查看任务的各种配置参数

### 2. 动态伸缩服务实例数

在swarm集群中部署一个服务后，你就可以使用命令行来改变服务的实例个数：

```bash
docker service scale <CONTAINER>=<REPLICAS>
```

其中`<CONTAINER>`是任务，`<REPLICAS>`是该任务的总数。

```bash
$ docker service scale jupyter=3
jupyter scaled to 3
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged
```

运行完成后，可通过`docker service ls`或是`docker service ps jupyter`查看任务，

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                         PORTS
yyjyeni1t0so        jupyter             replicated          3/3                 silvesterhsu/jupyter:latest   *:8888->8888/tcp
03niz8ebse57        portainer           replicated          1/1                 portainer/portainer:latest    *:9000->9000/tcp

$ docker service ps jupyter
ID                  NAME                IMAGE                         NODE                        DESIRED STATE       CURRENT STATE           ERROR               PORTS
ms2zv5wsyvg2        jupyter.1           silvesterhsu/jupyter:latest   vmirdos6.hostflyte.com      Running             Running 2 hours ago
om3wofnemrph         \_ jupyter.1       silvesterhsu/jupyter:latest   p5hy0c1ez2eauvw1molg9nck2   Shutdown            Orphaned 2 hours ago
zcn4ypjemzaj        jupyter.2           silvesterhsu/jupyter:latest   vmirdos6.hostflyte.com      Running             Running 5 minutes ago
ir5ixcsp2fr5        jupyter.3           silvesterhsu/jupyter:latest   vmirdos6.hostflyte.com      Running             Running 5 minutes ago
```

### 3. 删除服务

通过`docker service rm <CONTAINER>`来删除服务

```bash
docker service rm jupyter
```

> 尽管服务不存在了，任务容器还需要几秒钟来清理，你可以在节点上docker ps查看任务什么时候被移除。

### 4. 创建网络

创建`overlay`网络

```bash
$ docker network create --driver overlay jupyter-network
gdcu39950a8p4rswwpxem6ibi
```

创建服务

```bash
$ docker service create --network jupyter-network --publish published=8888,target=8888 --name jupyter -d silvesterhsu/jupyter jupyter notebook
t8iuapr4bbpwc1ojgs09nvt6w
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

# Portainer可视化

## 1. 下载镜像

```bash
docker pull portainer/portainer
```

## 2. 创建 volume

```bash
docker volume create portainer_data
```

查看volume创建结果：

```bash
$ docker volume ls
DRIVER              VOLUME NAME
local               portainer_data
```

## 3. 启动Portainer服务

在`manager`启动portainer服务：

```bash
docker service create \
--name portainer \
--publish 9000:9000 \
--constraint 'node.role == manager' \
--mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
--mount type=volume,src=portainer_data,dst=/data \
portainer/portainer \
-H unix:///var/run/docker.sock
```

查看启动的服务：

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                         PORTS
yyjyeni1t0so        jupyter             replicated          1/1                 silvesterhsu/jupyter:latest   *:8888->8888/tcp
03niz8ebse57        portainer           replicated          1/1                 portainer/portainer:latest    *:9000->9000/tcp
```

通过浏览器即可访问`<MANAGER-IP>:9000`portainer服务，随后创建账户，登陆即可看见`manager`。