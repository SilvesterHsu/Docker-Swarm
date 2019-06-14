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

