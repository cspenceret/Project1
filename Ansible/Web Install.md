# Project1 - Web Install

## Overview

This procedure is to use Ansible playbook that installs Docker and configures web VM's with the DVWA web app.

## Prerequistes

The procedure to install Ansible shall be completed and working as per the following document: 

[Ansible Install Document](../Ansible/Ansible_Install.md)

Create two web virtual machines and ensure they are allocated to an Availability Set eg WEB-Avail-Set

Key to this is that the Ansible hosts files contains the IP addresses of the web servers are configured under the tag [webservers].

## Procedure

Using gitBash SSH to your jump box, and connect to the Ansible container.  Follow this procedure:

1. sudo docker container list -a to identify the container name.

```
CONTAINER ID   IMAGE                          COMMAND                  CREATED       STATUS                        PORTS     NAMES
35a08e09566c   cyberxsecurity/ansible         "/bin/sh -c /bin/bas…"   12 days ago   Exited (127) 50 seconds ago             [Container_Name]
```
2. sudo docker start [Container_Name]

3. sudo docker attach [Container_Name] to obtain a Docker shell.

You are now connected to the Ansible Container shell.

### Web Ansible YAML Playbook 
Create a YAML playbook configuration file in the ansible container. For this project the file is located in /etc/ansible/ and named web_playbook.yml

Note that the inbound and outbound port 80:80 should be set in the DVWA Ansible configuration.
  
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
ansible-playbook /etc/ansible/web_playbook.yml

```
PLAY [Config Web VM with Docker] ***************************************************************

TASK [Gathering Facts] *************************************************************************

ok: [web1 ip]

ok: [web2 ip]

TASK [docker.io] *******************************************************************************
[WARNING]: Updating cache and auto-installing missing dependency: python-apt

changed: [web1 ip]
changed: [web2 ip]

TASK [Install pip3] *****************************************************************************
changed: [web1 ip]
changed: [web2 ip]

TASK [Install Docker python module] ************************************************************
changed: [web1 ip]
changed: [web2 ip]

TASK [download and launch a docker web container] **********************************************
changed: [web1 ip]
changed: [web2 ip]

PLAY RECAP *************************************************************************************
web1 ip                   : ok=5 
web2 ip                   : ok=5
  changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```


To test that DVWA is running on the new VM, SSH to the new VM from your Ansible container.

SSH to your container:

ssh [username]@10.0.0.8
```
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 5.0.0-1027-azure x86_64)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

  System information as of Mon Jan  6 20:01:03 UTC 2020

  System load:  0.01              Processes:              122
  Usage of /:   9.9% of 28.90GB   Users logged in:        0
  Memory usage: 58%               IP address for eth0:    10.0.0.8
  Swap usage:   0%                


18 packages can be updated.
0 updates are security updates.


Last login: [Date]


### Test using curl

Run curl localhost/setup.php to test the connection. If everything is working, you should get back some HTML from the DVWA container.

- curl localhost/setup.php
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">

  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

    <title>Setup :: Damn Vulnerable Web Application (DVWA) v1.10 *Development*</title>

    <link rel="stylesheet" type="text/css" href="dvwa/css/main.css" />

    <link rel="icon" type="\image/ico" href="favicon.ico" />

    <script type="text/javascript" src="dvwa/js/dvwaPage.js"></script>

  </head>
```

### Load Balancer

Install a load balancer in front of the web VM's to distribute the traffic among more than one VM.

Create a new load balancer in your red team resource group and give it a name.

![Load balancer](/Diagrams/Loadbalancer.png)


Add a frontend IP address.

![Front End Adress](/Diagrams/LB_FE_Address.png)

Create a new public IP address.

Add a backend pool and add the web VM's.

![Front End Adress](/Diagrams/LB_BE_Pool.png)

### filebeat
``` 
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

    - name: download filebeat deb
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
      
    - name: install filebeat deb
      command: sudo dpkg -i filebeat-7.6.1-amd64.deb

    - name: Copy yml config to elk server
      copy:
        src: /etc/ansible/files/filebeat-config.yml
        dest: /etc/filebeat/filebeat.yml

    - name: enable and configure system module
      command: filebeat modules enable system
      
    - name: setup filebeat
      command: filebeat setup

    - name: start filebeat service
      command:  service filebeat start

    - name: enable service filebeat on boot
      systemd:
        name: filebeat
        enabled: yes
```


### Metricbeat

- Config file - Add the ELK server internal IP Address to the Kibana endpoint configuration and the Elasticsearch ouput.
```
# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:
  host: "10.0.0.4:5601"

  #-------------------------- Elasticsearch output ------------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["10.0.0.4:9200"]
  username: "elastic"
  password: "changeme"
```
- Ansible File

```
---
- name: installing and launching metricbeat
  hosts: webservers
  become: yes
  tasks:

    - name: download metricbeat deb
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
      
    - name: install metricbeat deb
      command: sudo dpkg -i metricbeat-7.4.0-amd64.deb

    - name: Copy yml config
      copy:
        src: /etc/ansible/roles/metricbeat-config.yml
        dest: /etc/metricbeat/metricbeat.yml

    - name: setup metricbeat
      command: metricbeat setup

    - name: enable and configure system module
      command: metricbeat modules enable docker
      #systemd: filebeatservice
      #become: true

    - name: setup metricbeat
      command: metricbeat setup

    - name: start metricbeat service
      command: systemctl start metricbeat

    - name: enable service metricbeat on boot
      systemd:
        name: metricbeat
        enabled: yes

```