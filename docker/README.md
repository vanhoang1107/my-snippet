# error creating vxlan interface: file exists
> Reference: https://github.com/moby/libnetwork/issues/562#issuecomment-1155015141
My swarm service was down due to this issue.

I caused it by doing a yum update of docker while my docker services were still running.

I found a solution here which states that this occurs due to a race condition. dockerd is supposed to delete the vxlan device after a container is stopped.

```shell
$ docker service ls
ID             NAME                MODE         REPLICAS   IMAGE                                                                                  PORTS
ze71bkwjaqat   banana_server       replicated   0/1        blah.dkr.ecr.us-east-1.amazonaws.com/banana-api:banana-app-server-ad11cff               *:4030->3030/tcp

$ docker service ps banana_server --no-trunc
ID                          NAME                  IMAGE                             NODE           DESIRED STATE   CURRENT STATE             ERROR                                                                                                                       PORTS
                      
beg13kixclccj0utj59jym24m   banana_server.1       blah.dkr.ecr.us-east-1.amazo...   qa-server-01   Ready           Rejected 3 seconds ago    "network sandbox join failed: subnet sandbox join failed for "10.0.11.0/24": error creating vxlan interface: file exists"
yinps5swi9xf7jmdr27uk5s1u    \_ banana_server.1   blah.dkr.ecr.us-east-1.amazo...   qa-server-01   Shutdown        Rejected 8 seconds ago    "network sandbox join failed: subnet sandbox join failed for "10.0.11.0/24": error creating vxlan interface: file exists"
```

This was all I needed to do:

list the vxlan devices that are in a bad state
```shell
$ ip -d link show | grep vx
5659: vx-00100b-ripjm: <BROADCAST,MULTICAST> mtu 1450 qdisc noop state DOWN mode DEFAULT group default
5010: vx-00100c-o0la3: <BROADCAST,MULTICAST> mtu 1450 qdisc noop state DOWN mode DEFAULT group default
```
find which vxlan devices correspond to your docker services (the last 5 characters of the vx-*****-***** ID are a docker network ID)
```shell
$ docker network ls | grep -E "ripjm|o0la3"
ripjm8evcjro   banana-net-01                    overlay   swarm
o0la3crbs51r   banana_default            overlay   swarm
```
delete the vxlan devices
```shell
$ ip link delete vx-00100b-ripjm
$ ip link delete vx-00100c-o0la3
```
the docker swarm service should now come up clean
```shell
$ docker service ls
ID             NAME                 MODE         REPLICAS   IMAGE                                                                                  PORTS
...
ze71bkwjaqat   banana_server       replicated   1/1        blah.dkr.ecr.us-east-1.amazonaws.com/banana-api:banana-app-server-ad11cff               *:4030->3030/tcp
$
```