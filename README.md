# Project-Base-Learning-12

ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

## PROJECT TASK

- Deploy and configure UAT Web Servers using Ansible imports and roles

## BACKGROUND KNOWLEDGE

In this project we will continue working with **ansible-config-mgt** repository and make some improvements of the code. Now there is need to learn how to...

- Refactor the Ansible code 
- Create Static assignments
- Import Playbook functionality

**Code Refactoring** ...this is the process of restructuring existing computer code. Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. For this project, you will move things around a little bit in the code, but the overal state of the infrastructure remains the same.

**Imports** allows effective re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.

**Ansible** is a popular open-source configuration management and automation tool that is used for various IT tasks such as *configuration management*, *application deployment*, and *orchestration*. One of the key features of Ansible is its ability to use variables to store and manipulate data. These variables can be defined in various ways and can be used to perform *dynamic assignments* in ansible playbooks.

**Dynamic assignments** in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments

**Static Assignments** here, the values of variables are defined and fixed at the time of writing the playbook. 

Dynamic assignments are useful in scenarios where the values of variables need to be determined based on certain conditions or inputs.

There are several ways to perform dynamic assignments in ansible. Some of the common methods are:

- **Using conditionals** : Conditionals in ansible allow you to specify a block of tasks that should be executed only if a certain condition is met. For example, you can use the when clause to specify a condition under which a task should be executed.

- **Using loops** : Loops in ansible allow you to repeat a set of tasks for a given number of times or for each item in a list. You can use loops to perform dynamic assignments by looping over a list of items and assigning a value to a variable for each iteration.

- **Using set_fact module** : The set_fact module in ansible allows you to set the value of a variable based on the output of a command or expression. You can use this module to perform dynamic assignments by specifying a command or expression that returns the value you want to assign to the variable.

- **Using register module** : The register module in ansible allows you to store the output of a task in a variable. You can use this module to perform dynamic assignments by storing the output of a task in a variable and using it later in the playbook.

In summary, dynamic assignments in ansible allow you to use variables whose values are determined at runtime. This is a useful feature that enables you to write more flexible and dynamic playbooks that can adapt to changing conditions or inputs.

## Setup and technologies used in Project 12

- Two RHEL8 Web Servers
- One RHEL8 DB Server
- One RHEL8 NFS server
- One LB server(based on Ubuntu 20.04 LTS)
- One **Jenkins/Ansible Server** server(based on Ubuntu 20.04 LTS)
- Two 'UAT' Web Server

- Your target architecture will look like this:

![12_1](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/300339c9-aace-47a4-9497-4fe1f1184c74)

## STEP 1 - JENKINS JOB ENHANCEMENT  

In our previous projects 9 & 11, Jenkins was configured to create a directory for every change in the code which consumes space in the Jenkins server. To fix this we will be making changes using **copy artifact** plugin.

In our "Jenkins-Ansible server", We create a new directory called **ansible-config-artifact** where we will store all artifacts after each build.

`sudo mkdir ansible-config-artifact`

Change permissions to this directory, so Jenkins can save files in the ansible-config-artifact.

`sudo chmod -R 0777 ansible-config-artifact`

### 12_2

Go to Jenkins web console click on **Manage Jenkins** then click on **Manage Plugins**, On available tab search for **Copy Artifact** and install this plugin without restarting Jenkins. Create a new Freestyle project and name it **save_artifacts**

### 12_3 showing all steps above

This project will be triggered by completion of your existing ansible project. Configure it accordingly:

Go to configurations, make the following changes.

We can configure the number of builds to keep in order to save space on the server. In this project we want to keep the last 2 builds, We update the Source **Code Management** section.

### 12_4

The main idea of **save_artifacts** project is to save artifacts into **/home/ubuntu/ansible-config-artifact** directory. To achieve this, we will create a Build step and choose Copy artifacts from our previous ansible project. We will specify ansible as a source project and **/home/ubuntu/ansible-config-artifact** as a target directory.

### 12_5 showing above steps

Then click on apply and save.

To test this setup, we will be making some changes in the README.MD file inside the **ansible-config-mgt** repository.

If both Jenkins jobs have completed one after another – you shall see your files inside **/home/ubuntu/ansible-config-artifact** directory and it will be updated with every commit to your main branch.

### Pix showing success execution of above step

## STEP 2 - REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML

