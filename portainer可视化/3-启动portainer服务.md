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