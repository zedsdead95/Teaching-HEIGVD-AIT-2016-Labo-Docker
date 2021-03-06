# Administration IT
## Laboratoire n°4 :

> Auteurs : Loic Frueh - Koubaa Walid	
> Date : 07.01.2019  



### Pedagogical objectives
Build your own Docker images

Become familiar with lightweight process supervision for Docker

Understand core concepts for dynamic scaling of an application in production

Put into practice decentralized management of web server instances

Instructions for the lab report
This lab builds on a previous lab on load balancing.

In this lab you will perform a number of tasks and document your progress in a lab report. Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

We expect you to have in your repository (you will get the instructions later for that) a folder called report and a folder called logs. Ideally, your report should be in Markdown format directly in the repository.

###Task 0: Identify issues and install the tools

First we install all the tools we need as we did on the previous laboratory.

![alt](img/1.png)

![alt](img/2.png)

![alt](img/3.png)


**Deliverables**

Here is the screenshot of our backend nodes on HAProxy:

![alt](img/4.png)


Here is the url of our repository:

https://github.com/zedsdead95/Teaching-HEIGVD-AIT-2016-Labo-Docker


Here is our answers for the questions

**M1: Do you think we can use the current solution for a production environment? What are the main problems when deploying it in a production environment?**

No we can't use this solution for a production environement. The main problems are that every time a node is added or removed for some reason, we have to edit the config files manually which is not good for a production environment. Moreover, we need to rebuild the images after every modification which cause a lack of disponibility.

**[M2] Describe what you need to do to add new webapp container to the infrastructure. Give the exact steps of what you have to do without modifiying the way the things are done. Hint: You probably have to modify some configuration and script files in a Docker image.**

- First we need to modify the **haproxy.cfg** and add a new node in the load balancer:

In the "Define the list of nodes" part add ```server s3 <s3>:3000 check```

- Then add the new container to the **run.sh** script:

```sed -i 's/<s3>/$S3_PORT_3000_TCP_ADDR/g' /usr/local/etc/haproxy/haproxy.cfg```

- Next modify the **provision.sh**:

add these lines

```docker rm -f s3 2>/dev/null || true docker ```

```run -d --name s3 softengheigvd/webapp```

modify the last line like this (add --link s3)

```docker run -d -p 80:80 -p 1936:1936 -p 9999:9999 --link s1 --link s2 --link s3 --name ha softengheigvd/ha```

- Finally, just type ```vagrant provision``` in your favorite command line terminal so that the changes become effective.




**[M3] Based on your previous answers, you have detected some issues in the current solution. Now propose a better approach at a high level.**

A better aproach would be to have a daemon software always running in background that detects every change in the infrastructure (node's addition or deletion) and edit the config files as in the M2 point.

**[M4] You probably noticed that the list of web application nodes is hardcoded in the load balancer configuration. How can we manage the web app nodes in a more dynamic fashion?**

We need to use a tool that runs on all nodes and exchange message periodically in order to know which node is currently running and then update the config files dynamically. This tool, for this lab, is named **SERF**. This tool allows us to update the load balancer configuration dynamically.

**[M5] In the physical or virtual machines of a typical infrastructure we tend to have not only one main process (like the web server or the load balancer) running, but a few additional processes on the side to perform management tasks. For example to monitor the distributed system as a whole it is common to collect in one centralized place all the logs produced by the different machines. Therefore we need a process running on each machine that will forward the logs to the central place. (We could also imagine a central tool that reaches out to each machine to gather the logs. That's a push vs. pull problem.) It is quite common to see a push mechanism used for this kind of task. Do you think our current solution is able to run additional management processes beside the main web server / load balancer process in a container? If no, what is missing / required to reach the goal? If yes, how to proceed to run for example a log forwarding process?**

Current solution only run one process per container. For this case we need to run multiple process in the same docker container.
In order to do this, we need to add a new abstract layer such as a process supervisor in each container. This process supervisor will be always running in background and monitoring other processes.

**[M6] In our current solution, although the load balancer configuration is changing dynamically, it doesn't follow dynamically the configuration of our distributed system when web servers are added or removed. If we take a closer look at the run.sh script, we see two calls to sed which will replace two lines in the haproxy.cfg configuration file just before we start haproxy. You clearly see that the configuration file has two lines and the script will replace these two lines. What happens if we add more web server nodes? Do you think it is really dynamic? It's far away from being a dynamic configuration. Can you propose a solution to solve this?**

If we add some nodes we also have to add them in the load balancer config file. Currently this has to be done manually which is bad. A solution, in this lab, would be the usage of a template engine. The template will use the datas provided by the **SERF agents** to generate the config file.



## Task 1


We followed all the instructions, build the new images and killed the previous containers and rebuild them after modifyig the Dockerfiles.

We executed the scripts furnished.

Here are the containers created 

![alt](img/5.png)

Then we need to configure S6 as our main process.
Once again we followed all the instructions, build the new images and killed the previous containers and rebuild them after modifyig the Dockerfiles.

We execute 	

		mkdir -p /vagrant/ha/services/ha /vagrant/webapp/services/node

We have then the appropriate structure.

Once copied, we replaced the hashbang instruction in both files.

The start scripts are ready but now we must copy them to the right place in the Docker image. In both ha and webapp Docker files, you need to add a COPY instruction to setup the service correctly.

We did that, and rebuild images and restarted the containers.


![alt](img/6.png)

1. Here is the screenshot of the stats page of HAProxy at http://192.168.42.42:1936.

	![alt](img/7.png)

2. We did not found difficulties doing these tasks since scripts were furnished.

	We need to install a process supervisor to be able to run more than one service per container. That is because we have discovered that we can install a serf agent as a service in each container next to it's main service in order to modify the load balancer's configuration when we add or remove backend nodes.
	
	
	
## Task 2

After installing serf dependances we execute

mkdir /vagrant/ha/services/serf /vagrant/webapp/services/serf

Then we copied the run file from ha/services/serf to webapp folder.

We succeeded in applying modifications on Dockerfiles and we tested the logs for the containers. Here is a part of the log for **ha container**

![alt](img/8.png)

Then we created the bridge and killed all containers.

![alt](img/9.png)

1. the logs are in the logs folder

2. The problem with the current solution is that we need to be careful on the order of containers starting. Indeed, if we start the web apps nodes before the haproxy, then the web apps will try to join the cluster but won't be able to do this because the cluster is started by the haproxy container which is not yet started himself. In addition, if one web app fail to join the cluster, its **SERF agent** starting will also fail.

3. **SERF** is a cluster managment tool based on the **GOSSIP** Protocol. Each nodes run a **SERF agent** that communicate with other and share its status. Thanks to this, the cluster (and all its nodes) is always aware of the addition of a new node or the deletion of an existing node in the cluster. The **GOSSIP** Protocol is built on UDP to broadcast messages between the cluster's nodes. 

The other solution may be: ZooKeeper or doozerd.


## Task 3
We copied the scripts as asked.

We modified the dockerfile.

We then build and run our docker images/containers.

1. The logs are in the logs folder.

2. For the in-container logs, we copies both logs from vagrant for ha container and the bash log in the same file :)


## Task 4

First we install nodejs.

Then we modify the scripts as specified.


1. Each **RUN** command result in the creation of a layer. Therefore, it is more light to do only one **RUN** instruction instead of three. The less layer we have the less memory space it takes, it's a gain of performance and time. However, when it comes to rebuild an image with the addition of an extra library for instance, it's more advised to have multiple layer because all the layer goes in cache and only the layer impacted by the new modifications are rebuild the other are simply taken from the cache memory. If there was only one **RUN** command, all the image would had to be rebuilt.

The **squashing** is the reduction of the layers number in a docker image without losing the benefit of the cache memory.

The **flattening** goal is to reduce the size of a docker container by deleting its history.


2. Our solution is that there should be the least RUN instructions possible to avoid having a lot of "layers" when building a docker image. This implementation requires a repetition of several instructions with the && operator. But the disadvantage using this technique, is that a run command would probably grow very large and instructions could be repeated.
This is why using a RUN for a unique operation is more proper, each opeation on a different line. These are two "schools" but we prefer the second one :D !

3. We got the log save on logs folder

	The file for docker logs is named docker_log

4. Based on the last three logs (inspect), we can see that we are the output is manually generated in the cfg file. What we should do is to find a mechanism that will do this automatically.



## Task 5

1. haproxy files are provides such as other log files inside the logs folder

2. Here is the content of the folder

	![alt](img/10.png)


3. The docker ps log file is in the logs folder

## Task 6

We replaced the  ha/services/ha/run script by the following script part.

We then modified the two scripts member-join and member-leave as told.

Then we rebuild re-run all the containers/images

Finally we can confortly see that we can easily start our nodes on  http://192.168.42.42 

![alt](img/11.png)

1. here are the screenshot we made:

	![alt](img/12.png)

 	log file for docker ps log is in the logs folder

2. Personnaly we liked this solution eventhought it was quite long to implement.


## Conclusion

This lab was quite simple to realize thanks to the long but detailed steps and explanations. But we found that this lab was very long to finish nevertherless. Thanks very much for these useful steps for our future job or future tasks we may have to do on our professionnal life.
