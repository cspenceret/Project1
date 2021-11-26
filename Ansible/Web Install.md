# Project1 - Web Install

## Overview

This procedure is to use Ansible playbook that installs Docker and configures web VM's with the DVWA web app.

## Prerequistes

The procedure to install Ansible shall be completed and working as per the following document: 

[Ansible Install Document](Ansible/Ansible_Install.md)

Key to this is that the Ansible hosts files contains the IP addresses of the web servers unter the tag [webservers].

## Procedure

Using gitBash SSH to your jump box, and connect to the Ansible container.

1. sudo docker container list -a to identify the container name.

```
CONTAINER ID   IMAGE                          COMMAND                  CREATED       STATUS                        PORTS     NAMES
35a08e09566c   cyberxsecurity/ansible         "/bin/sh -c /bin/bas…"   12 days ago   Exited (127) 50 seconds ago             [Container_Name]
```
2. sudo docker start [Container_Name]

3. sudo docker attach [Container_Name] to obtain a Docker shell

### Web Ansible YAML Playbook 
Create a YAML playbook configuration file in the ansible container. For this project the file is located in /etc/ansible/ and named web_playbook.yml
  
  ```diff
---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:

  - name: Uninstall apache2
    apt:
      name: apache2
      state: absent

  - name: docker.io
    apt:
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      name: python3-pip
      state: present

  - name: Install Python Docker Module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      restart_policy: always
      published_ports: 80:80

```
Note: Use the Ansible apt module to install docker.io and python3-pip.  Note that update_cache must be used here, or docker.io will not install. 

Use the Ansible docker-container module to install the cyberxsecurity/dvwa container.

Make sure you publish port 80 on the container to port 80 on the host.
```

Running your playbook should produce an output similar to the following:
root@1f08425a2967:~# ansible-playbook /etc/ansible/pentest.yml

```
PLAY [Config Web VM with Docker] ***************************************************************

TASK [Gathering Facts] *************************************************************************
ok: [10.0.0.6]

TASK [docker.io] *******************************************************************************
[WARNING]: Updating cache and auto-installing missing dependency: python-apt

changed: [10.0.0.6]

TASK [Install pip3] *****************************************************************************
changed: [10.0.0.6]

TASK [Install Docker python module] ************************************************************
changed: [10.0.0.6]

TASK [download and launch a docker web container] **********************************************
changed: [10.0.0.6]

PLAY RECAP *************************************************************************************
10.0.0.6                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```


To test that DVWA is running on the new VM, SSH to the new VM from your Ansible container.

SSH to your container:

root@1f08425a2967:~# ssh sysadmin@10.0.0.6
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 5.0.0-1027-azure x86_64)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

  System information as of Mon Jan  6 20:01:03 UTC 2020

  System load:  0.01              Processes:              122
  Usage of /:   9.9% of 28.90GB   Users logged in:        0
  Memory usage: 58%               IP address for eth0:    10.0.0.6
  Swap usage:   0%                IP address for docker0: 172.17.0.1


18 packages can be updated.
0 updates are security updates.


Last login: Mon Jan  6 19:33:51 2020 from 10.0.0.4

Run curl localhost/setup.php to test the connection. If everything is working, you should get back some HTML from the DVWA container.

ansible@Pentest-1:~$ curl localhost/setup.php

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">

  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

    <title>Setup :: Damn Vulnerable Web Application (DVWA) v1.10 *Development*</title>

    <link rel="stylesheet" type="text/css" href="dvwa/css/main.css" />

    <link rel="icon" type="\image/ico" href="favicon.ico" />

    <script type="text/javascript" src="dvwa/js/dvwaPage.js"></script>

  </head>


### Load Balancer

Solution Guide: Load Balancing
To complete this activity, you had to install a load balancer in front of the VM to distribute the traffic among more than one VM.

Create a new load balancer and assign it a static IP address.


Start from the homepage and search for "load balancer."



Create a new load balancer in your red team resource group and give it a name.



Add a frontend IP address.

![Front End Adress](../Diagrams/LB_FE_Address.png)



Give the IP address a unique address name. This name will be used to create a URL that maps to the IP address of the load balancer.


Create a new public IP address.





Add a backend pool.

![Front End Adress](../Diagrams/LB_BE_Pool.png)


Add your Web VMs to the backend pool.





Do not add any inbound or outbound rules.