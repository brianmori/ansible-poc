# The Ansible POC
## Work smart and do not suffer pain from manual tasks anymore

I realized this proof of concept to understand Ansible in a real scenario and to present this solution in order to not suffer any more pain executing repetitive and manual tasks.
The PoC at the moment implements a Linux Apache MySQL PHP (LAMP) infrastructure, I do not exclude more features or components in the future ;-)

Topics covered:
* Provision of the infrastructure
* Setup a workstation VM with Ansible plus DNS Server installed
* Provision Vagrant testing and production environments
* Install and configure the web servers with Apache 2.4 and PHP 7.1
* Install and configure the database server MySQL 5.7
* Configure the virtual host, create the database and the db user
* Deploy the website and test it

The PoC can be run on any platform

Windows / Linux:
* Virtual Box
* Vagrant
* CentOS 7

Installation of the infrastructure:
 
 1. The default host-only virtual network is perfect (eg. 192.168.56.0/24)
 2. Create a Virtual Machine in Virtualbox with CentOS 7 
    1. Assign the static ip 192.168.56.100 on the host-only interface
    2. Execute as root or sudo "yum install ansible"
    3. Copy the id_rsa.txt in the $HOME/.ssh/id_rsa
    4. Copy the id_rsa.pub.txt in the $HOME/.ssh/id_rsa.pub
3. Change directory into the Vagrant
   1. vagrant up will start only the Testing environment, in case the computer has enough RAM, you can start already the Production environment by using:
   2. vagrant up ws_prod_1 
   3. vagrant up db_prod_1

### **! Ansible time !**

 The preparation of the infrastructure is finally complete, and we are ready to automate!
 Change the working directory to the *ansible/* directory of the cloned repository

##### Infrastructure facts

Let's give a look what kind of properties the infrastructure has:

    ansible all -m setup

This command will return all facts for the whole infrastructure. It is possible to limit the scope by inventory

    ansible -i inventories/production all -m setup

    ansible -i inventories/testing all -m setup

#####Bind DNS Setup
BIND DNS is going to be setup on the Virtual Machine Workstation to resolve the names.
A forward and reverse lookup zones are going to be setup.

    ansible-playbook -i inventories/workstation  workstation-pb.yml

You can now try to login with ssh to the target Vagrant VM
    ssh -l vagrant -i $HOME/.ssh/id_rsa webserver-test-1.mypoc.io

### Setup Testing Environment

This phase consists to configure the testing environment

##### Web  Server setup
    ansible-playbook -i inventories/testing/  webservers-pb.yml --vault-password-file vault_pass_testing.txt

It configures the repositories (REMI, Oracle MySQL, EPEL, ...) and it installs the Apache Httpd web server.

##### Database setup
    ansible-playbook -i inventories/testing/  dbservers-pb.yml --vault-password-file vault_pass_testing.txt

It configures the repositories (REMI, Oracle MySQL, EPEL, ...) and it installs the Oracle MySQL 5.7 db server.  

##### Web  Site deployment
    ansible-playbook -i inventories/testing/ www.website-pb.yml --vault-password-file vault_pass_testing.txt

It configures a technical user on the database and a virtual host in the httpd, it deploys the php page which open a database connection and it displays some information.

##### Verification time of Test environment !

Open a browser and navigate to these two links

    http://www.test.mypoc.io/server-status
    http://www.test.mypoc.io/test_db.php


### Setup Production Environment

This environment is pretty similar as testing environment, after all it is production. There are some differences though, the passwords are different but the playbook is the same, repetitive actions across environments.

In case you did not start the Vagrant production for memory limits, you can do it now, first by stopping the test instances
                
    vagrant halt 
    vagrant up ws_prod_1
    vagrant up db_prod_1

##### Web  Server setup
    ansible-playbook -i inventories/production/ webservers-pb.yml --vault-password-file vault_pass_production.txt

##### Database setup
    ansible-playbook -i inventories/production/ dbservers-pb.yml --vault-password-file vault_pass_production.txt

##### Web  Site deployment
    ansible-playbook -i inventories/production/ www.website-pb.yml --vault-password-file vault_pass_production.txt

##### Verification time of Prod environment !

Open a browser and navigate to these two links

    http://www.mypoc.io/server-status
    http://www.mypoc.io/test_db.php

In just few minutes, the testing and production environment are complete in a repeatable process.


### **! Ansible POC files !**

I have structured in two different ways:

 - Playbooks with roles for the configuration of the infrastructure
 - Plain playbook for the application deployment

I am personally very much against application deployments with tools like Ansible/Puppet, as Jenkins offers much more features. Nothing impedes though to use Jenkins with Ansible ;-)

#### Roles 

You can think roles as a category which is applied to infrastructure assets.
Roles may be shared to different asset categories.
The role *common* is shared between the db and web server of testing and production environment

