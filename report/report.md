## AIT Lab 04 - Docker

**Author:** Müller Robin, Stéphane Teixeira Carvalho, Massaoudi Walid  
**Date:** 2020-12-02

### Introduction

In this laboratory, we will configure a load-balancer with dynamic scaling of servers. The goal of this lab is to become familiar with lightweight process supervision for Docker, understanding the concepts of dynamic scaling and make a decentralized management of web server instances.

In this report you will find the answers to the different questions and the difficulties encountered throughout the laboratory.

### Table of content
1. [Task 0 : Identify issues and install the tools](#Task-0-:-Identify-issues-and-install-the-tools)
1. [Task 1 : Add a process supervisor to run several processes](#Task-1-:-Add-a-process-supervisor-to-run-several-processes)


### Task 0 : Identify issues and install the tools
#### M1 : Do you think we can use the current solution for a production environment? What are the main problems when deploying it in a production environment?
The main problem will be that we will use the same amount of servers every time of the year. For instance, we could have something more intelligent that will create a new server when we have a lot of incoming requests and shut down some servers when there is not much work to do. Currently all the servers in the infrastructure are static and we have defined the amount of servers desired, two s1 and s2.

#### M2 : Describe what you need to do to add new webapp container to the infrastructure. Give the exact steps of what you have to do without modifiying the way the things are done.
To add a new container, first, we will have to go in the docker compose file and add the following lines :
```
webapp3:
       container_name: ${WEBAPP_3_NAME}
       build:
         context: ./webapp
         dockerfile: Dockerfile
       networks:
         heig:
           ipv4_address: ${WEBAPP_3_IP}
       ports:
         - "4002:3000"
       environment:
            - TAG=${WEBAPP_3_NAME}
            - SERVER_IP=${WEBAPP_3_IP}
haproxy:
       container_name: ha
       build:
         context: ./ha
         dockerfile: Dockerfile
       ports:
         - 80:80
         - 1936:1936
         - 9999:9999
       expose:
         - 80
         - 1936
         - 9999
       networks:
         heig:
           ipv4_address: ${HA_PROXY_IP}
       environment:
            - WEBAPP_1_IP=${WEBAPP_1_IP}
            - WEBAPP_2_IP=${WEBAPP_2_IP}
            - WEBAPP_3_IP=${WEBAPP_3_IP}
```

And then in the haproxy file :

```
# Define the list of nodes to be in the balancing mechanism
# http://cbonte.github.io/haproxy-dconv/2.2/configuration.html#4-server
server s1 ${WEBAPP_1_IP}:3000 check
server s2 ${WEBAPP_2_IP}:3000 check
server s3 ${WEBAPP_3_IP}:3000 check
```

#### M3 : Based on your previous answers, you have detected some issues in the current solution. Now propose a better approach at a high level.
As shown in the previous answer the configuration of a new node is relatively long. A better approach will be to use a script to create a new container when necessary and update the load-balancer to add the new server in the network. We could think of a that will automatically repeat the steps above or use a service that can do that for us.

#### M4 : You probably noticed that the list of web application nodes is hardcoded in the load balancer configuration. How can we manage the web app nodes in a more dynamic fashion?

We could think, as discussed in the previous question, of a application or service that will handle the new hosts and send their arriving or the fact that they are leaving to the load-balancer.

#### M5 : Do you think our current solution is able to run additional management processes beside the main web server / load balancer process in a container? If no, what is missing / required to reach the goal? If yes, how to proceed to run for example a log forwarding process?
No the current solution does not allow a container to handle multiple process. This is because Docker has define that a container is a process and so only one process can be run by a container.

To reach the goal of running multiple processes we can add to the container a process supervisor. This process will handle multiple inside a container and with that we can define which process could kill the container if he fail or the one's that does not kill the container. You will see later in the lab a process supervisor named `S6`.

#### M6 : What happens if we add more web server nodes? Do you think it is really dynamic? It's far away from being a dynamic configuration. Can you propose a solution to solve this?
We could think of a solution where we use a template for a node and use a script to change the configuration of the haproxy container directly.


#### 0.1
<img alt="Test 1" src="./screens/T0_s1.png">

On the screenshot shown above we can see the 2 nodes that were started s1 and s2

#### 0.2
Here is our URL :
https://github.com/Naludrag/Teaching-HEIGVD-AIT-2020-Labo-Docker

### Task 1 : Add a process supervisor to run several processes

#### 1.1
<img alt="Test 1" src="./screens/T1_s1.png">

We can see that this task has been done successfully. The 2 nodes are still visible and the haproxy accessible.

#### 1.2
In this task we installed a init system to be able to execute multiple process in a docker container. The basic idea of Docker is to run one process per container but with S6 we can manage and supervise multiple processes. With S6 we can choose which process to restart and we can never let our container die. We could also choose which process, if they die, shut down the container.
With this service we do not have "one process per container" but more "one thing per container".

Docker containers have to run a foreground process to be kept alive. And so, S6 will run as the foreground process and let the other processes run as background process. With that, if a process fail the container will not die.

In our laboratory, it will be interesting to have such a tool because we could restart the haproxy process to have the update the init script to add new servers.

### Task 2 : Add a tool to manage membership in the web server cluster

#### 2.1
All the logs for this point are in the folder logs but here is the link to the files :
- [HAProxy](../logs/task%202/ha.log)
- [S1](../logs/task%202/s1.log)
- [S2](../logs/task%202/s2.log)

#### 2.2
In the current solution we could have a problem because all the nodes have to be registered through the HAProxy. Because of that, if the HAProxy is not up new machines could not join the cluster and that is not what the Serf mind set is for.

The Serf mind set is to join a cluster through multiple machines and not only one.

#### 2.3
The Serf agent will try to join the cluster trough the load-balancer. If the cluster is not created it will create it and in reverse if it exists it will join it. But this step will only succeed if the ha container is accessible if not the startup of the Serf agent will fail.

If a new node has joined the cluster or if it leave the cluster, Serf will use the gossip protocol to tell other nodes that about this information. This protocol is based on SWIM protocol and serf sends custom messages types on top of this layer. Two messages in particular are often used :
- Join Intent :  This message is sent to tell the nodes that a new machine is now in the cluster.
- Leave Intent : This message is sent when a server gracefully exit the cluster. If a server fail no Leave Intent is sent and so with this message the Serf layer can know if a server has stopped gracefully or not.

This messages are always sent with a Lamport clock to maintain some notion of messages ordering.

If we want to find other solutions than Serf we could use for instance ZooKeeper or doozerd. But in some cases this solutions can be less interesting as for instance ZooKeeper. ZooKeeper cannot be used as a tool and a lot of the times developers have to use libraries to build features that the need.

### Task 3 : React to membership changes

#### 3.1
Here are the logs for this step :
- [HAProxy](../logs/task%203/ha.log)
- [S1](../logs/task%203/s1.log)
- [S2](../logs/task%203/s2.log)
- [HAProxy with nodes](../logs/task%203/ha_nodes.log)

#### 3.2
The logs of the serf.log file can be seen with the link below:
- [HAProxy](../logs/task%203/serf.log)

### Task 4: Use a template engine to easily generate configuration files

#### 4.1
- [Flattening docker images ](https://l10nn.medium.com/flattening-docker-images-bafb849912ff)
- [Squashing docker images](http://jasonwilder.com/blog/2014/08/19/squashing-docker-images/)
- [Slim down docker images ](https://blog.codacy.com/five-ways-to-slim-your-docker-images/)
- [Docker images layers](https://docs.docker.com/storage/storagedriver/)

#### 4.2
Use the shared  files and layers  between these images.
#### 4.3
The /tmp/haproxy.cfg generated file :
- [ha](../logs/task%204/haproxy_ha.cfg)
- [s1](../logs/task%204/haproxy_s1.cfg)
- [s2](../logs/task%204/haproxy_s2.cfg)

Docker ps :
- [Docker ps](../logs/task%204/Docker_ps.log)

Docker inspect:
- [ha](../logs/task%204/Docker_insp_ha.log)
- [s1](../logs/task%204/Docker_insp_s1.log)
- [s2](../logs/task%204/Docker_insp_s2.log)

#### 4.4
The containers use an overlay2 storage driver. When an existing file in a container is modified, the storage driver performs a copy-on-write operation. The specifics steps involved depend on the specific storage driver.
need more.

### Task 5 : Generate a new load balancer configuration when membership changes

#### 5.1
Here are the config files for the different steps :
- [No nodes](../logs/task%205/haproxy_nonodes.cfg)
- [Only s1](../logs/task%205/haproxy_onlys1.cfg)
- [All the nodes](../logs/task%205/haproxy_nodes.cfg)

Here are the files for the `docker inspect` :
- [HA inspect](../logs/task%205/docker_inspect_ha.log)
- [S1 inspect](../logs/task%205/docker_inspect_s1.log)
- [S2 inspect](../logs/task%205/docker_inspect_s2.log)

And here is the link to the `docker ps` command :
- [Docker ps](../logs/task%205/docker_ps.log)

#### 5.2
Here is the log file for the `/nodes` folder :
- [HA inspect](../logs/task%205/nodes_folder.log)

#### 5.3
Here is the log file for the `/nodes` folder :
- [HA inspect](../logs/task%205/nodes_folder_without_s1.log)

Here is the result of the configuration file :
- [Config file without s1](../logs/task%205/haproxy_without_s1.cfg)

Finally, here is the output of the `docker ps` command :
- [Config file without s1](../logs/task%205/docker_ps_without_s1.log)

### Difficulties
#### Task 1
In this task no difficulties were encountered.

#### Task 2
In this task no difficulties were encountered.

#### Task 3
In this task no difficulties were encountered.

#### Task 5
In this task no difficulties were encountered.

### Conclusion
To conclude, we found this laboratory interesting because we could practice the theory seen in the course. It was also interesting to see what can happen if a load-balancer has a slower server or if the session stickiness is not enabled. Seeing multiple balancing strategies and to choose between them is also an interesting point in the laboratory.

Finally, we are happy with the result that we have and we think that we completed the laboratory successfully.
