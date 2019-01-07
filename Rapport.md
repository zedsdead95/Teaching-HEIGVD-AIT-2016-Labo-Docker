# Administration IT
## Laboratoire n°4 :

> Auteurs : Loic Frueh - Koubaa Walid	
> Date : 07.01.2019  



###Pedagogical objectives
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



**[M3] Based on your previous answers, you have detected some issues in the current solution. Now propose a better approach at a high level.**

A better aproach would be to have a daemon software always running in background that detects every change in the infrastructure (node's addition or deletion) and edit the config files as in the M2 point.

**[M4] You probably noticed that the list of web application nodes is hardcoded in the load balancer configuration. How can we manage the web app nodes in a more dynamic fashion?**



**[M5] In the physical or virtual machines of a typical infrastructure we tend to have not only one main process (like the web server or the load balancer) running, but a few additional processes on the side to perform management tasks.**







