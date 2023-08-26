# Project-Base-Learning-12

ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

## PROJECT TASK

- Deploy and configure UAT Web Servers using Ansible imports and roles

## BACKGROUND KNOWLEDGE

In this project we will continue working with **ansible-config-mgt** repository and make some improvements of the code. Now there is need to learn how to...

- Refactor the Ansible code 
- Create Static assignments
- Import Playbook functionality

**Code Refactoring** ...this is the process of restructuring existing computer code. Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. For this project, you will move things around a little bit in the code, but the overall state of the infrastructure remains the same.

**Imports** allows effective re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.

**Ansible** is a popular open-source configuration management and automation tool that is used for various IT tasks such as *configuration management*, *application deployment*, and *orchestration*. One of the key features of Ansible is its ability to use variables to store and manipulate data. These variables can be defined in various ways and can be used to perform *dynamic assignments* in ansible playbooks.

**Dynamic assignments** in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments

**Static Assignments** here, the values of variables are defined and fixed at the time of writing the playbook. 

### move to project 13
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

In our previous projects 9 & 11, Jenkins was configured to create a directory for every change in the code, this consumes space in the Jenkins server. To fix this we will be making changes using **copy artifact** plugin.

In our "Jenkins-Ansible server", We create a new directory called **ansible-config-artifact** where we will store all artifacts after each build, Change permissions to this directory, so Jenkins can save files in the ansible-config-artifact.

First start up all your previous Instances from project 11, from VS CODE, connect to your JENKINS-ANSIBLE via the path PS C:\Users\EZEPC\Desktop\DevOps> where your .PEM kkey is stored.

```
ssh -A ubuntu@54.81.72.41
sudo mkdir ansible-config-artifact
sudo chmod -R 0777 ansible-config-artifact
```

![12_2](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/a8f80b7a-b300-40d4-8581-62fa329a307c)


Go to Jenkins web console click on **Manage Jenkins** then click on **Manage Plugins**, On available tab search for **Copy Artifact** and install this plugin without restarting Jenkins.

![12_3](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/b56e047f-a1f9-4f60-a4c8-1dae7b1fab62)


 Create a new Freestyle project and name it **save_artifacts**

 ![12_4](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/374201d4-dcd6-45fb-b319-5881036bd6e9)


This project will be triggered by completion of your existing ANSIBLE project. Configure it accordingly:

Go to configurations, make the following changes.

We can configure the number of builds to keep in order to save space on the server. In this project we want to keep the last 2 builds, We update the Source **Code Management** section.

![12_5](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/75ca017b-0894-4d4e-b8ff-8e334a8adf28)

The main idea of **save_artifacts** project is to save artifacts into **/home/ubuntu/ansible-config-artifact** directory. To achieve this, we will create a Build step and choose Copy artifacts from our previous ansible project. We will specify ansible as a source project and **/home/ubuntu/ansible-config-artifact** as a target directory.

![12_6](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/e7212f1d-716c-42b3-a346-7ff225d7dd69)

Then click on apply and save to see the Expect Result as seen in Step 16 above.

To test this setup, we will be making some changes in the README.MD file inside the **ansible-config-mgt** repository.

If both Jenkins jobs have completed one after another – you shall see your files inside **/home/ubuntu/ansible-config-artifact** directory and it will be updated with every commit to your main branch. Remember to update your GITHUB  **ansible-config-mgt** repository with the current Ansible URL..

![12_7](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/85ddef8f-a3a3-4b38-baf2-9c5559d3f5e3)

## STEP 2 - REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML

Before starting to refactor the codes, ensure that you have pulled down the latest code from **main** branch, and created a new branch, name it **refactor** . DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used.




In Project-11, all the tasks are written in a single playbook common.yml. These are pretty simple set of instructions for only 2 types of OS. In a case where we have many more tasks and we need to apply this playbook to other servers with different requirements, We will have to read through the whole playbook to check if all tasks written there are applicable and to check if there is anything that needs to be added for certain server/OS families. This will become a tedious exercise and the playbook will become messy which will make it difficult for your colleagues to use your playbook.

Breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let see code re-use in action by importing other playbooks.

- 2_A : Within the **playbooks** directory, create a new file and name it **site.yml**. This file will be an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, **site.yml** will become a parent to all other playbooks that will be developed. Including *common.yml* that was created previously. 

- 2_B : Create a new directory withing the **ansible-config-mgt** directory and name it **static-assignment**. The static-assignment directory is where all other children playbooks will be stored.This is merely for easy organization of your work.

- 2_C :  Move **common.yml** file into the newly created **static-assignment** directory.

### 12_7 pix showing 1 & 2 above

- 2_D : Inside **playbooks/site.yml** file, import **common.yml** playbook.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
Your folder structure should look like this;

### 12_8 PIX OF YOUR NEW OLDER STRUCTURE

- 2_E : Run **ansible-playbook** command against the **inventory/dev.yml** environment

Since you need to apply some tasks to the development servers and wireshark is already installed in project-11. We will create another playbook in the static-assignment and name it **common-del.yml**

We will configure this playbook to delete wireshark utility.  In the **static-assignment/common-del.yml** we paste the snippet

```
---
- name: update web, nfs and db servers
  hosts: wbsrvrs, nfs, dbsrvr
  remote_user: ec2-user
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

### 12_9 pix showing update of above on common-del.yml

Update **playbooks/site.yml** with the following snippet replacing **common.yml** with **common-del.yml** and run it against dev servers:

```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml
```

To push the codes to github and merge to the main branch

We run the commands

```
git status
git add .
git commit -m "<commit-message>"
git push origin refactor
```

### 12_10 pix showing OK execution of above

We then go to the github and create a pull request, Merge to the main branch

We can see the Jenkins build as soon as the code is merged to the main branch

### 12_11 pix showing OK execution of above


The artifact is saved in the ansible-config-artifact directory in the jenkins server.

We then run the playbook

`ansible-playbook -i ansible-config-artifact/inventory/dev.yml ansible-config-artifact/playbooks/site.yaml`

### 12_11 pix showing OK execution of above

To check that wireshark is deleted on all the servers, we run the comand on the various target servers.

`wireshark --version`

### 12_12 pix showing OK execution of above

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.

## STEP 3 - CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’

We have our nice and clean **dev** environment, so let us put it aside and configure 2 new Web Servers as **UAT**. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated **ROLE** to make our configuration reusable.

**Roles** let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users.

- 3_A : Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our UAT servers, so give them names accordingly – **Web1-UAT** and **Web2-UAT**

**TAKE NOTE**: power off all other instances not in use to avoid extra cost. We are just going to be using the Jenkins server and the UAT webservers for this section.

- 3_B : To create a role, you must create a directory called **roles/** , relative to the playbook file or in **/etc/ansible/** directory.

There are two ways how you can create this folder structure

  1 - Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)

    ```
    mkdir roles
    cd roles
    ansible-galaxy init webserver
    sudo apt install tree -y
    tree webserver
    ```
  2 - Create the directory/files structure manually

Option 1 was choosen, - 

### 12_13 pix showing OK execution of above CMD

The entire directory structure should look like below after removing unnecessary directories and files

### 12_14 pix showing THE new tree

- 3_C : Update your inventory **ansible-config-mgt/inventory/uat.yml** file with IP addresses of your 2 UAT Web servers

### 12_15 pix showing OK update

- 3_D : Update the jenkins /etc/hosts directory with the UAT webservers.

### 12_16 pix showing OK update

- 3_E : Check for connectivity to the uat-webservers

### 12_16 pix showing OK ping

- 3_F : Go to /etc/ansible/ansible.cfg and update

### 12_17 pix showing OK UPDATE

- 3_G : Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

*DEVELOP YOUR OWN SCRIPT*
Install and configure Apache (httpd service)
Clone Tooling website from GitHub **https://github.com/dybran/tooling.git**
Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started.

Your **main.yml** should consist of following tasks:

```
---
- name: install apache
  yum:
    name: "httpd"
    state: present

- name: install git
  yum:
    name: "git"
    state: present

- name: clone a repo
  git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  file:
    path: /var/www/html/html
    state: absent
```

## STEP 4 - REFERENCE WEBSERVER ROLE

Within the **static-assignment** directory, create a new assignment for **uat-webservers uat-webservers.yml**. This is where we will reference the role.

### 12_18 pix showing OK of above

Update the uat-webservers.yml file

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

### 12_18 pix showing OK of above

Remember that the entry point to our ansible configuration is the **site.yml** file. Therefore, you need to refer your **uat-webservers.yml** role inside **site.yml**

So, we should have this in site.yml

```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml

- hosts: all
- import_playbook: ../static-assignments/uat-webservers.yml
```

### 12_19 pix showing OK of above


## STEP 5 - COMMIT AND TEST

Commit your changes, create a Pull Request and merge them to main branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into **/home/ubuntu/ansible-config-mgt/** directory.

Push to the github repository refactor branch

```
git status
git add .
git commit -m "<commit-message>"
git push origin refactor
```


### 12_20 pix showing OK of above

Create a pull request, then merge to main branch, The Jenkins builds automatically. Then saves the artifact in the ansible-config-artifact Jenkins server.

### 12_21 pix showing OK of above

Now we run the playbook

`ansible-playbook -i /ansible-config-artifact/inventory/uat.yml /ansible-config-artifact/playbooks/site.yml`

### 12_22 pix showing OK of above

We should be able to see both of the UAT Webservers configured and can reach them from the browser:

http://<Web01-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

### 12_23 pix showing OK of above

# CONGRATULATIONS Eze ! You have just deployed and configured UAT Web Servers using Ansible imports and roles.
