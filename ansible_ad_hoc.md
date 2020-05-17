# Ansible AD HOC Commands

## Prerequisites
* Ansible must be installed - only in control machine. Ansible is agentless
* Some remote Virtual machines to test, you can use vagrant, AWS/Azure cloud instance or local VMs
* Make sure that the control machine and VM has SSH connectivity. Best practice is to enable SSH key-based authentication for efficiency.

## What is Ansible ad hoc commands
**ad hoc** is latin words refers to something done for a very precise and particular purpose. As the word suggests, ansible ad hoc commands are **written for a very particular task**. Note that if the task is frequent in used then it is best to have a playbook created for the same so that typing again and again of the ad-hoc command is avoided

## Ansible ad_hoc commands Syntax
![](Image/ad_hoc_syntax.png)

To run an ad hoc command, the command must have the following syntax.
```
ansible <host-pattern> [options]
```

for example. the command should be written as follows.
```
ansible serverhostgroup -m <modulename> -a <arguments to the module>
```
**NOTE: serverhostgroup can be all or IP or groupname**

A single ansible ad hoc command can have multiple options. -m and -a are one amongst them and widely used.

## Examples: ad hoc commands

**1:  Validate the connection between the control machine and host, using ansible ad hoc commands.**

In this example, we are going to test the remote nodes or hosts and make sure they respond back using Ansible’s default SSH channel

We presume that you have set up SSH key based auth between the control machine and the hosts. If yes then there is no need to enter the credentials and command would be simple. Also you need to add your remote host details in the /etc/ansible/hosts file else you have to use **-i** flag for the custom inventory path

```
$ ansible myserverhost -m ping 
host01 | SUCCESS => {
"changed": false,
"ping": "pong"
}

host02 | SUCCESS => {
"changed": false,
"ping": "pong"
}

host03 | SUCCESS => {
"changed": false,
"ping": "pong"
}
```

**2:  Get the uptime of remote hosts using ansible ad hoc command**

In this example, we are going to know the uptime of the hosts. Ansible provides two major modules to run the command over the remote server.

Which one to pick is not a big confusion if you know what are they and their capabilities

Here are the commands you can use to get the uptime. All three commands would yield you the same results.
```
ansible myserverhost -m command -a uptime

ansible myserverhost -m shell -a uptime

ansible myserverhost -a uptime
```

as you could have already figured out **-m** is the module and **-a** should contain the command it should run which goes as an argument to command and shell.


**NOTE:** The difference between **command** and **shell** command is that variables like **$HOME** and operations like **<, >, |, ;** and **&** will not work with the **command** module. If you want to run any command say you using **<, >, |,** etc you actually want the **shell** module instead.

**3:  How to check the free memory or memory usage of  hosts using ansible ad hoc command**

The following ansible ad hoc command would help you get the free memory of all the hosts in the host group named **myserverhost**

Here, we are running the **free** -m command on the remote hosts and collecting the information
```
$ ansible myserverhost -a "free -m"
host01 | SUCCESS | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:           1839         108        1570           8         160        1563
Swap:          1023           0        1023

host02 | SUCCESS | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:           1839         100        1581           8         157        1573
Swap:          1023           0        1023
```

**4:  ansible ad hoc command to get physical memory allocated to the host**

In this example we are going to use two commands together so we must opt to shell module.
```
$ ansible myserverhost -m shell -a "cat /proc/meminfo | head -2" 
host01 | SUCCESS | rc=0 >>
MemTotal:        1883428 kB
MemFree:         1616460 kB

host02 | SUCCESS | rc=0 >>
MemTotal:        1883428 kB
MemFree:         1607372 kB
```
 
**5:  ansible ad hoc command to execute a command as root user (sudo) on host**

In this example, we are going to access one of the privileged configuration files. We are going to check if the user exists by searching the /etc/passwd file
```
$ ansible myserverhost -m shell -a "cat /etc/passwd | grep -i someuser" -b -K
host01 | SUCCESS | rc=0 >>
someuser:x:1000:1000:someuser:/home/someuser:/bin/bash

host02 | SUCCESS | rc=0 >>
someuser:x:1000:1000:someuser:/home/someuser:/bin/bash

here,
-b is the option for become and by default it will become root user

–K is to tell ansible to ask for SUDO password
```

**6:  ansible ad hoc command to Execute a command as a different user  (sudo su)**

In this example, we are going to create a new file inside a directory /opt/dir_ac which is owned by ac user

In the following ad-hoc command snapshot you can see we have given the username we want to switch to  using --become-user=weblogic option
```
$ ansible myserverhost  -m file -a "path=/opt/dir_ac/testDIR state=directory mode=0755" --become-user=ac
host01 | SUCCESS => {
    "changed": true,
    "gid": 1001,
    "group": "ac",
    "mode": "0755",
    "owner": "ac",
    "path": "/opt/dir_ac/testDIR",
    "secontext": "unconfined_u:object_r:usr_t:s0",
    "size": 6,
    "state": "directory",
    "uid": 1001
}
host02 | SUCCESS => {
    "changed": true,
    "gid": 1001,
    "group": "ac",
    "mode": "0755",
    "owner": "ac",
    "path": "/opt/dir_ac/testDIR",
    "secontext": "unconfined_u:object_r:usr_t:s0",
    "size": 6,
    "state": "directory",
    "uid": 1001
}
```
 
**7: Create a unix user group with ansible ad hoc command**

In this example, we will use the ad hoc command to manage presence of groups on a host. With below ad-hoc command we ensure group "somegroup" exists
```
$ ansible myserverhost -m group -a "name=somegroup state=present" 
host01 | SUCCESS => {
    "changed": true,
    "gid": 1001,
    "name": "somegroup",
    "state": "present",
    "system": false
}
host02 | SUCCESS => {
    "changed": true,
    "gid": 1001,
    "name": "somegroup",
    "state": "present",
    "system": false
}
```
**8: Create a unix user with ansible ad hoc command**

In this example, we will use the ad hoc command to manage user accounts and user attributes. With below ad-hoc command we add create the user 'john' with a primary group of 'admin' and create its home directory
```
$ ansible app -m user -a "name=john group=admin createhome=yes" -b
host01 | SUCCESS => {
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1001,
    "home": "/home/john",
    "name": "john",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1001
}
host02 | SUCCESS => {
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1001,
    "home": "/home/john",
    "name": "john",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1001
}
``` 

**9: Create a Directory with 755 permission using ansible ad hoc command**

In this example, we will use the ad hoc command to createa directory with required permission
```
$ ansible myserverhost -m file -a "path=/opt/dir_ac state=directory mode=0755" -b
host01 | SUCCESS => {
    "changed": true,
    "gid": 0,
    "group": "root",
    "mode": "0755",
    "owner": "root",
    "path": "/opt/dir_ac",
    "secontext": "unconfined_u:object_r:usr_t:s0",
    "size": 6,
    "state": "directory",
    "uid": 0
}
host02 | SUCCESS => {
    "changed": true,
    "gid": 0,
    "group": "root",
    "mode": "0755",
    "owner": "root",
    "path": "/opt/dir_ac",
    "secontext": "unconfined_u:object_r:usr_t:s0",
    "size": 6,
    "state": "directory",
    "uid": 0
}
```

**Example 10: Create a file with 755 permission using ansible ad hoc commands**

In this example, we will use the ad hoc command to createa file with required permission
```
$ ansible app -m file -a "path=/tmp/testfile state=touch mode=0755"
host02 | SUCCESS => {
    "changed": true,
    "dest": "/tmp/testfile",
    "gid": 1000,
    "group": "admansible",
    "mode": "0755",
    "owner": "admansible",
    "secontext": "unconfined_u:object_r:user_tmp_t:s0",
    "size": 0,
    "state": "file",
    "uid": 1000
}
host01 | SUCCESS => {
    "changed": true,
    "dest": "/tmp/testfile",
    "gid": 1000,
    "group": "admansible",
    "mode": "0755",
    "owner": "admansible",
    "secontext": "unconfined_u:object_r:user_tmp_t:s0",
    "size": 0,
    "state": "file",
    "uid": 1000
}
```

**12: How to check free disk space of hosts using ansible ad hoc commands**
```
ansible myserverhost -m shell -a "df -h"
```

**13: ad hoc command to Install a package using yum command**

In this example we use the ad hoc command to manage package with the yum package manager. Below command install the latest version of httpd.
```
ansible myserverhost -m yum -a "name=httpd state=latest"
```

**14: ad hoc command to Start or stop the service**

In this example we use the ad hoc command to manage package with the yum package manager to start and stop the service.

***To Start***
```
ansible multi -s -m service -a "name=httpd state=started enabled=yes"
```
***To Stop***
```
ansible multi -s -m service -a "name=httpd state=stop enabled=yes"
```
 
**15: Install and configure python Django application server with ansible ad hoc commands**

These are set of commands you have to execute to install the Django application server and Mysql libraries. Here we are using easy_install which is an ansible module it helps to find the easy installation option from ansible galaxy
```
$ ansible myserverhost -s -m yum -a "name=MySQL-python state=present"
$ ansible myserverhost -s -m yum -a "name=python-setuptools state=present"
$ ansible myserverhost -s -m easy_install -a "name=django"
```

**16: Managing Cron Job and Scheduling with Ansible ad hoc**

In this example we see how do we manage cron.d and crontab entries.

*Run the job every 15 minutes to execute the script*
```
$ ansible myserverhost -m cron -a "name='daily-cron-all-servers' minute=*/15 job='/path/to/minute-script.sh'"
```

*Run the job every four hours to execute the hour-script*
```
$ ansible myserverhost -s -m cron -a "name='daily-cron-all-servers' hour=4 job='/path/to/hour-script.sh'"
```

*Enabling a Job to run at system reboot*
```
$ ansible myserverhost -s -m cron -a "name='daily-cron-all-servers' special_time=reboot job='/path/to/startup-script.sh'"
```

*Scheduling a Daily job*
```
$ ansible myserverhost -s -m cron -a "name='daily-cron-all-servers' special_time=daily job='/path/to/daily-script.sh'"
```

*Scheduling a Weekly job*
```$ ansible myserverhost -s -m cron -a "name='daily-cron-all-servers' special_time=weekly job='/path/to/daily-script.sh'"
```
 
**17: File Transfer**

The ansible ad-hoc command below is used to copy a file from a source to a destination for a group of hosts (myserverhost) defined in the inventory file. After you enter the password, the output with “change” parameter will be “true”, which means the file has been copied to the destination.
```
$ ansible node1 -m copy -a 'src=/home/ac/test.yml dest=/home/ac/Desktop/ owner=root mode=0644' -u root --become -K

node1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "5631822866afd5f19b928edb3ba018385df22dd3",
    "dest": "/home/ac/Desktop/nginx.yml",
    "gid": 0,
    "group": "root",
    "md5sum": "0d6ffe1069fc25ad4f8ad700277c4634",
    "mode": "0644",
    "owner": "root",
    "size": 280,
    "src": "/root/.ansible/tmp/ansible-tmp-1562253463.3-214622150088155/source",
    "state": "file",
    "uid": 0
}

here, 
--become --> run operations with become (does not imply password prompting)
-K --> ask for privilege escalation password (user this if ssh passwordless is not setup)
-u REMOTE_USER --> connect as this user (default=None)
```

**18: Fetch file from remote to localhost (ansible) machine**

The ansible ad-hoc command below is used to download a file from a remote host defined in the command to the localhost. In this command, we are downloading a file using fetch module from node1 server to a local destination on ansible node.
```
$ ansible node1 -m fetch -a 'src=/etc/passwd dest=/home/ac/example/ flat=yes'
node1 | SUCCESS => {
    "changed": false,
    "checksum": "5631822866afd5f19b928edb3ba018385df22dd3",
    "dest": "/home/ac/example/nginx.yml",
    "file": "/etc/passwd",
    "md5sum": "0d6ffe1069fc25ad4f8ad700277c4634"
}
```

**19: Gathering Facts**

The ansible ad-hoc command below will give you all the ad-hoc information of your system, including all the variables present in the system
```
$ansible all -m setup
```


