# the when conditional
For this part is required a new VM, with a different distro that doesnt use the `apt` package manager, I chose Rocky Linux, a RedHat clone, that uses `dnf`. Configure the new VM to have openssh-server installed, a user with the same name and password as you have set for the other servers, make sure the user is part of the sudoers, copy the ssh keys to the new VM. On your ansible control host add the new VM's IP address to the inventory file.

## when
we can use `when` to make sure a condition is met before executing a task, our current playbooks update and install packages using the apt module so any of those tasks on a server like our new VM, that doesnt support apt, will fail.

## when: ansible_distribution = " "
separating the tasks by the VM distros

to be sure of what the ansible_distribution names are for our servers we can run this:
`ansible all -m gather_facts | grep ansible_distribution`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m gather_facts | grep ansible_distribution
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Debian",
        "ansible_distribution_major_version": "22",
        "ansible_distribution_release": "jammy",
        "ansible_distribution_version": "22.04",
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Debian",
        "ansible_distribution_major_version": "22",
        "ansible_distribution_release": "jammy",
        "ansible_distribution_version": "22.04",
        "ansible_distribution": "Rocky",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "8",
        "ansible_distribution_release": "Green Obsidian",
        "ansible_distribution_version": "8.5",
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Debian",
        "ansible_distribution_major_version": "22",
        "ansible_distribution_release": "jammy",
        "ansible_distribution_version": "22.04",

```

now edit the install_apache.yml playbook
```
---

- hosts: all # we want to run this on all hosts in inventory
  become: true # we want to be able to run the tasks as sudo
  tasks:

  # for distros using apt
  - name: update repository index
    apt:
      update_cache: yes
    when: ansible_distribution in ["Ubuntu"," Debian"] # if there is a mix of distros that use apt

  - name: install apache2 package # this is a name for the task and it will show up during execution
    apt: # the module that we are going to use
      name: apache2 # the name of the package
      state: latest
    when: ansible_distribution in ["Ubuntu"," Debian"]

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution in ["Ubuntu"," Debian"]

  # for distros using dnf
  - name: update repository index
    dnf:
      update_cache: yes
    when: ansible_distribution == "Rocky" # if there's only one distro that uses dnf

  - name: install apache package (httpd) # this is a name for the task and it will show up during execution
    dnf: # the module that we are going to use
      name: httpd # some packages name change in different distros
      state: latest
    when: ansible_distribution == "Rocky"

```
running the playbook, notice how it will skip the tasks if the condition is not met and viceversa
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml 
BECOME password: 

PLAY [all] ***********************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [update repository index] ***************************************************************************************************************************************
skipping: [10.0.0.213]
changed: [10.0.0.84]
changed: [10.0.0.121]
changed: [10.0.0.106]

TASK [install apache2 package] ***************************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [add php support for apache] ************************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.84]
ok: [10.0.0.121]
ok: [10.0.0.106]

TASK [update repository index] ***************************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

TASK [install apache package (httpd)] ********************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
changed: [10.0.0.213]

TASK [add php support for apache] ************************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
changed: [10.0.0.213]

PLAY RECAP ***********************************************************************************************************************************************************
10.0.0.106                 : ok=4    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.121                 : ok=4    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.213                 : ok=4    changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

## making sure httpd is running in our new VM
unlike in Ubuntu, Rocky and other RedHat-based distros dont start the apache or httpd service after install

```
[anthony@webserver-4 ~]$ systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
   Active: inactive (dead)
     Docs: man:httpd.service(8)
[anthony@webserver-4 ~]$ 

```
starting the service
```
[anthony@webserver-4 ~]$ sudo systemctl start httpd
[sudo] password for anthony: 
```
```
[anthony@webserver-4 ~]$ systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
   Active: active (running) since Thu 2022-07-07 22:50:39 EDT; 8s ago
     Docs: man:httpd.service(8)
 Main PID: 27000 (httpd)
   Status: "Started, listening on: port 80"
    Tasks: 213 (limit: 4936)
   Memory: 30.9M
   CGroup: /system.slice/httpd.service
           ├─27000 /usr/sbin/httpd -DFOREGROUND
           ├─27007 /usr/sbin/httpd -DFOREGROUND
           ├─27008 /usr/sbin/httpd -DFOREGROUND
           ├─27009 /usr/sbin/httpd -DFOREGROUND
           └─27010 /usr/sbin/httpd -DFOREGROUND

```
enable httpd to start at boot
```
[anthony@server-4 ~]$ sudo systemctl enable httpd
[sudo] password for anthony: 
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.

```
the final step is setting the firewall to allow communications in port 80
```
[anthony@webserver-4 ~]$ sudo firewall-cmd --add-port=80/tcp
success
```