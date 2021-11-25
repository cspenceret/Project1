# Project1

## Overview


## Prerequsites

Prior to deploying the ELK Server the ANSIBLE Container must be installed on the Jump Box.  Once completed, perform the following:

- Run Git Bash as administrator
- SSH to Jump Box in GIT BASH
- Start the Ansible container  
- Connect to the Ansible Container
- Navigate to /etc/ansible
- Edit ansible.cfg and hosts files, as follows:

### Edit Hosts file
 - edit the hosts file and add the following lines:
 
```[webservers]```

10.1.0.8 ansible_python_interpreter=/usr/bin/python3
10.1.0.9 ansible_python_interpreter=/usr/bin/python3

```[elk]```

10.0.0.4 ansible_python_interpreter=/usr/bin/python3

```[filebeat]```

127.0.0.1

### Edit Ansible.cfg File


## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Network Diagram](Diagrams/AzureCloudDiagram2.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Ansible file may be used to install only certain pieces of it, such as Filebeat.




## ELK Server Ansible Playbook
  ```diff
  ---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: cspencer
  become: true
  tasks:
  # Use apt module
  - name: Install docker.io
    apt:
      update_cache: yes
      force_apt_get: yes
      name: docker.io
      state: present
   # Use apt module
  - name: Install python3-pip
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present
   # Use pip module (It will default to pip3)
  - name: Install Docker module
    pip:
      name: docker
      state: present
   # Use command module
  - name: Increase virtual memory
    command: sysctl -w vm.max_map_count=262144
   # Use sysctl module
  - name: Use more memory
    sysctl:
     name: vm.max_map_count
     value: 262144
     state: present
     reload: yes
   # Use docker_container module
  - name: download and launch a docker elk container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
    # Please list the ports that ELK runs on
      published_ports:
      - 5601:5601
      - 9200:9200
      - 5044:5044
   # Use systemd module
  - name: Enable service docker on boot
    systemd:
      name: docker
      enabled: yes
```
This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly ```Available```, in addition to restricting ```Access``` to the network.
- _TODO: What aspect of security do load balancers protect? What is the advantage of a jump box?_

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.
- ```Filebeat monitors system logs```
- ```Metricbeat monitors and reports on system metrics```

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name    | Function   | IP Address | Operating System |
|---------|------------|------------|------------------|
| Jumpbox | Jump Host  | 10.1.0.4   | Linux            |
| web-1   | Web Server | 10.1.0.8   | Linux            |
| web-2   | Web Server | 10.1.0.9   | Linux            |
| ELK     | ELK Server | 10.0.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet.  This is achieved by ensuring the internal network is on its own private subnet and also the Network Security Group denys ALL except for SSH and one addressable workstaion with http allow. 

Only the _____ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _TODO: Add whitelisted IP addresses_

Machines within the network can only be accessed by _____.
- _TODO: Which machine did you allow to access your ELK VM? What was its IP address?_

A summary of the access policies in place can be found in the table below.

| Name |  Publicly Accessible   | Allowed IP Addresses | Control |
|---|---|---|---|
| Jump Box  internal | No | 10.1.0.4 | Security Group Policy and SSH |
| Jump Box Public | Yes | 1.123.42.152 | Security Group Policy and SSH |
| HTTP | Yes | 1.123.42.152 | Security Group Policy |
| ELK | No | 10.0.0.0/24 | Security Group Policy and SSH |
| ELK Public - Kabana | Yes | 1.123.42.152 | Security Group Policy and Port 5601 allow |






### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _TODO: What is the main advantage of automating configuration with Ansible?_

The playbook implements the following tasks:
- _TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- Install Docker.io
- Install Python PIP-3
- Install
- Increase Memory
- Download and install ELK container sebp/elk:761
- Publish elk ports
      - 5601:5601
      - 9200:9200
      - 5044:5044
   # Use systemd module
  - name: Enable service docker on boot
    systemd:
      name: docker
      enabled: yes

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
```
- web-1 10.1.0.8
- web-2 10.1.0.9
```
We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the _____ file to _____.
- Update the _____ file to include...
- Run the playbook, and navigate to ____ to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
