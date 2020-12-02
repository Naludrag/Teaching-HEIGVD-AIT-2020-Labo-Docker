## AIT Lab 04 - Docker

**Author:** Müller Robin, Stéphane Teixeira Carvalho, Massaoudi Walid  
**Date:** 2020-12-02

### Introduction

In this laboratory, we will configure a load-balancer with different configuration like sticky sessions or drain mode. We will also look at the results with the different modifications. We will also look at what happens when a server is lower than another. At the end, we will look at two new algorithms to choose a node and compare them by showing results and choose the most suitable for the laboratory.


### Task 0 : Identify issues and install the tools
#### M1 : Do you think we can use the current solution for a production environment? What are the main problems when deploying it in a production environment?

#### M2 : Describe what you need to do to add new webapp container to the infrastructure. Give the exact steps of what you have to do without modifiying the way the things are done.

#### M3 : Based on your previous answers, you have detected some issues in the current solution. Now propose a better approach at a high level.

#### M4 : You probably noticed that the list of web application nodes is hardcoded in the load balancer configuration. How can we manage the web app nodes in a more dynamic fashion?

#### M5 : Do you think our current solution is able to run additional management processes beside the main web server / load balancer process in a container? If no, what is missing / required to reach the goal? If yes, how to proceed to run for example a log forwarding process?

#### M6 : What happens if we add more web server nodes? Do you think it is really dynamic? It's far away from being a dynamic configuration. Can you propose a solution to solve this?


#### 0.1
<img alt="Test 1" src="./screens/T0_s1.png">

#### 0.2
Here is our URL :
https://github.com/Naludrag/Teaching-HEIGVD-AIT-2020-Labo-Docker

### Task 1 : Identify issues and install the tools

### Conclusion
To conclude, we found this laboratory interesting because we could practice the theory seen in the course. It was also interesting to see what can happen if a load-balancer has a slower server or if the session stickiness is not enabled. Seeing multiple balancing strategies and to choose between them is also an interesting point in the laboratory.

Finally, we are happy with the result that we have and we think that we completed the laboratory successfully.
