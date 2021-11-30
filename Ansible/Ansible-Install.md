# Project1 - Ansible Install
## Overview

This procedure installs Docker and Ansible on the Jump Box.

## Procedure

- sudo apt update 
- sudo apt install docker.io
- sudo systemctl status docker to check docker is running  If not run sudo systemctl start docker.

- sudo docker pull cyberxsecurity/ansible.

- docker run -it cyberxsecurity/ansible /bin/bash

- Run exit to quit.

- Add a SSH allow from Jump box to virtual Network

![SecGpSSH](/Diagrams/SecGpSSH.png)


## Provisioners

Connect to your Ansible container. Once you're connected, create a new SSH key and copy the public key.

- sudo docker start [container_name]
- sudo docker attach [container_name]
- Now connected to ansible container shell.
- Run ssh-keygen to create an SSH key.
- Run cat .ssh/id_rsa.pub to display your public key.
- Copy public key string.

Return to the Azure portal and locate one of your web-vm's details page.
 - Reset your Vm's password and use your container's new public key for the SSH user.  See Diagram below.

 ![Web SSH Reset Password](/Diagrams/WebSSH.png)

- Note the internal IP for your new VM from the Details page and test SSH connection to each web server.

```
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 5.0.0-1027-azure x86_64)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

System information as of [date]

System load:  0.01              Processes:           108
Usage of /:   4.1% of 28.90GB   Users logged in:     0
Memory usage: 36%               IP address for eth0: 10.1.0.8
Swap usage:   0%


0 packages can be updated.
0 updates are security updates.
```
- Exit this SSH session by typing exit.

## Ansible Configuration Files

## Hosts 

- goto /etc/ansible
- nano hosts
- Uncomment the [webservers] header line.
- Add webserver internal IP Address and python line , for example 10.1.0.8 ansible_python_interpreter=/usr/bin/python3

```
    # This is the default ansible 'hosts' file.
    #
    # It should live in /etc/ansible/hosts
    #
    #   - Comments begin with the '#' character
    #   - Blank lines are ignored
    #   - Groups of hosts are delimited by [header] elements
    #   - You can enter hostnames or ip addresses
    #   - A hostname/ip can be a member of multiple groups
    # Ex 1: Ungrouped hosts, specify before any group headers.

    ## green.example.com
    ## blue.example.com
    ## 192.168.100.1
    ## 192.168.100.10

    # Ex 2: A collection of hosts belonging to the 'webservers' group

    [webservers]
    ## alpha.example.org
    ## beta.example.org
    ## 192.168.1.100
    ## 192.168.1.110
    10.1.0.8 ansible_python_interpreter=/usr/bin/python3
	10.1.0.9 ansible_python_interpreter=/usr/bin/python3

```
### ansible.cfg

Change the Ansible configuration file to use your administrator account for SSH connections.

- nano /etc/ansible/ansible.cfg and scroll down to the remote_user option.
- Uncomment the remote_user line and replace root with your admin username using this format:
- remote_user = <user-name-for-web-VMs>
```

Example:
# What flags to pass to sudo
# WARNING: leaving out the defaults might create unexpected behaviours
#sudo_flags = -H -S -n

# SSH timeout
#timeout = 10

# default user to use for playbooks if user is not specified
# (/usr/bin/ansible will use current user as default)
remote_user = [sysadmin]

# logging is off by default unless this path is defined
# if so defined, consider logrotate
#log_path = /var/log/ansible.log

# default module name for /usr/bin/ansible
#module_name = command

```
- save file
- ansible all -m ping

```
Your output should look like:
10.1.0.8 | SUCCESS => {
"changed": false, 
"ping": "pong"
}
10.0.0.6 | SUCCESS => {
		"changed": false, 
		"ping": "pong"
}
```


