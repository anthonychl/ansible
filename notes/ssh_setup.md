# ssh setup

## install openssh on all VMs
```
sudo apt install openssh-server
```

## connect to each server from your workstation or ansible-control-host
```
anthony@ansible-control-host:~$ ssh anthony@10.0.0.121
The authenticity of host '10.0.0.121 (10.0.0.121)' can't be established.
ED25519 key fingerprint is SHA256:____
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.0.121' (ED25519) to the list of known hosts.
anthony@10.0.0.121's password:
```
## create ssh key pair with passphrase for your user account
```
anthony@ansible-control-host:~$ ssh-keygen -t ed25519 -C "anthony default"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/anthony/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/anthony/.ssh/id_ed25519
Your public key has been saved in /home/anthony/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:8gjaX7pGCvN/0U3kPHned4jBO/olmll9Zd6ERKEyVTQ anthony default
The key's randomart image is:
+--[ED25519 256]--+
|            .oE. |
|           ..o . |
|          o=...  |
|           oO... |
|    . . S. o B.o+|
|  oo ..+. . =.o+*|
|  .+.o. o. .o.o *|
|    o..o. .= o . |
|     o=o  +..    |
+----[SHA256]-----+
anthony@ansible-control-host:~$ 
```
-t is for type and -C is a comment. 

### check that the new key has been created
```
anthony@ansible-control-host:~$ la -la .ssh
total 24
drwx------ 2 anthony anthony 4096 Jul  2 23:55 .
drwxr-x--- 4 anthony anthony 4096 Jun  1 01:33 ..
-rw------- 1 anthony anthony    0 Jun  1 01:17 authorized_keys
-rw------- 1 anthony anthony  464 Jul  2 23:55 id_ed25519
-rw-r--r-- 1 anthony anthony   97 Jul  2 23:55 id_ed25519.pub
-rw------- 1 anthony anthony 3360 Jul  2 23:30 known_hosts
-rw------- 1 anthony anthony 2098 Jun 24 01:44 known_hosts.old
```
## copy the key to each server
```
ssh-copy-id -i ~/.ssh/id_ed25519.pub 10.0.0.121
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/anthony/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
anthony@10.0.0.121's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '10.0.0.121'"
and check to make sure that only the key(s) you wanted were added.
```
```
anthony@ansible-control-host:~$ ssh 10.0.0.121
Enter passphrase for key '/home/anthony/.ssh/id_ed25519': 
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-33-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jul  3 12:05:39 AM UTC 2022

  System load:            0.0
  Usage of /:             48.2% of 9.75GB
  Memory usage:           23%
  Swap usage:             0%
  Processes:              99
  Users logged in:        1
  IPv4 address for ens18: 10.0.0.121
  IPv6 address for ens18: 2601:583:c205:61b0::62af

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Sat Jul  2 23:26:10 2022 from 10.0.0.76
anthony@webserver-1:~$ 
logout
Connection to 10.0.0.121 closed.
anthony@ansible-control-host:~$ 

```
### authorized keys on 10.0.0.121 aka webserver-1
```
anthony@webserver-1:~$ cat .ssh/authorized_keys 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINvJqGUz9L7DjgTJNTo4niC3yjKo8UeLLS5rh1xCve7Z anthony default
```
## create ssh key for ansible without passphrase
### Be careful to change the name or it will overwrite our previous key
```
anthony@ansible-control-host:~$ ssh-keygen -t ed25519 -C "ansible"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/anthony/.ssh/id_ed25519): /home/anthony/.ssh/ansible   
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/anthony/.ssh/ansible
Your public key has been saved in /home/anthony/.ssh/ansible.pub
The key fingerprint is:
SHA256:Arg8Q3KDjtsapeDe+fo1UMfr8VXTgJreV6AWE7OrKvM ansible
The key's randomart image is:
+--[ED25519 256]--+
|            oo.  |
| . .    .   +o...|
|o = .  . o o.+ +.|
|o= o .. . + o.. o|
|o.*  .. S+ o.. . |
|o= o  ... +.o .  |
|+..    o ... .   |
|.o. . + ..       |
|.. ++o +E        |
+----[SHA256]-----+
```
### check that the new key has been created
```
anthony@ansible-control-host:~$ ls -la .ssh
total 32
drwx------ 2 anthony anthony 4096 Jul  3 00:38 .
drwxr-x--- 4 anthony anthony 4096 Jun  1 01:33 ..
-rw------- 1 anthony anthony  399 Jul  3 00:35 ansible
-rw-r--r-- 1 anthony anthony   89 Jul  3 00:35 ansible.pub
-rw------- 1 anthony anthony    0 Jun  1 01:17 authorized_keys
-rw------- 1 anthony anthony  464 Jul  2 23:55 id_ed25519
-rw-r--r-- 1 anthony anthony   97 Jul  2 23:55 id_ed25519.pub
-rw------- 1 anthony anthony 3360 Jul  2 23:30 known_hosts
-rw------- 1 anthony anthony 2098 Jun 24 01:44 known_hosts.old
```
## copy ansible ssh key to each server
```
anthony@ansible-control-host:~$ ssh-copy-id -i ~/.ssh/ansible.pub 10.0.0.121
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/anthony/.ssh/ansible.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Enter passphrase for key '/home/anthony/.ssh/id_ed25519': 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '10.0.0.121'"
and check to make sure that only the key(s) you wanted were added.
```
### checking that both keys were copied to webserver-1
```
anthony@webserver-1:~$ cat .ssh/authorized_keys 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINvJqGUz9L7DjgTJNTo4niC3yjKo8UeLLS5rh1xCve7Z anthony default
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBRsPEBBSXDFUrCKgcQQOqUUB8jWALTeWmuNpxbHkaLG ansible
```

## using the ansible key to log into a server
-i is for input file. Notice how it logs in without prompting for any further confirmation, its important to keep that ssh key secure.
```
anthony@ansible-control-host:~$ ssh -i ~/.ssh/ansible 10.0.0.121
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-33-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jul  3 12:50:48 AM UTC 2022

  System load:            0.0
  Usage of /:             49.4% of 9.75GB
  Memory usage:           24%
  Swap usage:             0%
  Processes:              100
  Users logged in:        1
  IPv4 address for ens18: 10.0.0.121
  IPv6 address for ens18: 2601:583:c205:61b0::62af

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

45 updates can be applied immediately.
22 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable
```

## using ssh-add 
ssh-add caches the passphrase for as long as we have our terminal open, that way we can ssh into the servers and only type the passphrase that one time
```
anthony@ansible-control-host:~$ eval $(ssh-agent)
Agent pid 1793
anthony@ansible-control-host:~$ ps aux | grep 1793
anthony     1793  0.0  0.1   7968  1076 ?        Ss   00:57   0:00 ssh-agent
anthony     1795  0.0  0.2   6476  2180 pts/1    S+   00:58   0:00 grep --color=auto 1793
anthony@ansible-control-host:~$ ssh-add
Enter passphrase for /home/anthony/.ssh/id_ed25519: 
Identity added: /home/anthony/.ssh/id_ed25519 (anthony default)
anthony@ansible-control-host:~$ 
```
### creating an alias to simplify even more the previous process
edit the .bashrc file
```
anthony@ansible-control-host:~$ nano .bashrc
```
add this and save
```
# ssh-agent
alias ssha='eval $(ssh-agent) && ssh-add'
```
### using the ssha alias
```
anthony@ansible-control-host:~$ ssha
Agent pid 1924
Enter passphrase for /home/anthony/.ssh/id_ed25519: 
Identity added: /home/anthony/.ssh/id_ed25519 (anthony default)
anthony@ansible-control-host:~$ ssh 10.0.0.121
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-33-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jul  3 01:11:47 AM UTC 2022

  System load:            0.0
  Usage of /:             49.4% of 9.75GB
  Memory usage:           24%
  Swap usage:             0%
  Processes:              100
  Users logged in:        1
  IPv4 address for ens18: 10.0.0.121
  IPv6 address for ens18: 2601:583:c205:61b0::62af

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

45 updates can be applied immediately.
22 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Sun Jul  3 00:50:49 2022 from 10.0.0.76
anthony@webserver-1:~$ 
logout
Connection to 10.0.0.121 closed.
anthony@ansible-control-host:~$ 

```

