# adding users and bootstrapping

## creating a new user
- site.yml
```
(...)
- hosts: all
  become: true
  tasks:

  - name: create jarvis user
    tags: always
    user: # using the user module
      name: jarvis # the new user name
      groups: root # user is part of this group

(...)
```
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass site.yml
BECOME password: 

PLAY [all] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [install updates (Ubuntu)] ***********************************************************************************************************************
skipping: [10.0.0.213]
changed: [10.0.0.76]
changed: [10.0.0.121]
changed: [10.0.0.84]
changed: [10.0.0.106]

TASK [install updates (Rocky)] ************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
skipping: [10.0.0.76]
ok: [10.0.0.213]

PLAY [all] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [create jarvis user] *****************************************************************************************************************************
changed: [10.0.0.106]
changed: [10.0.0.84]
changed: [10.0.0.121]
changed: [10.0.0.76]
changed: [10.0.0.213]

PLAY [workstations] ***********************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.76]

TASK [install unzip] **********************************************************************************************************************************
ok: [10.0.0.76]

TASK [install terraform] ******************************************************************************************************************************
ok: [10.0.0.76]

PLAY [web_servers] ************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ***************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] *********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [start httpd (Rocky)] ****************************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [change email address for admin (Rocky)] *********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [restart httpd (Rocky)] **************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.213]

TASK [copy default html file for site] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] ******************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] ************************************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] ***********************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ***************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ********************************************************************************************************************************************
10.0.0.106                 : ok=6    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=7    changed=2    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
10.0.0.213                 : ok=9    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.76                  : ok=7    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=6    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```
### check if the user was created
```
anthony@server-1:~$ cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
anthony:x:1000:1000:anthony:/home/anthony:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
jarvis:x:1001:1001::/home/jarvis:/bin/sh

```
## add ssh ansible key to the new user
- site.yml
```
(...)
  - name: add ssh key for jarvis
    tags: always
    authorized_key: # module to add or remove ssh authorized keys for a user account
      user: jarvis
      key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBRsPEBBSXDFUrCKgcQQOqUUB8jWALTeWmasasasasasas ansible" # copy from ~/.ssh/ansible.pub 
(...)
```

## make the new user part of the sudoers
- site.yml
```
(...)
  - name: add sudoers file for jarvis 
    tags: always 
    copy: 
      src: sudoer_jarvis 
      dest: /etc/sudoers.d/jarvis 
      owner: root 
      group: root 
      mode: 0440

(...)
```
- create the sudoer_jarvis file
```
anthony@ansible-control-host:~/ansible/scripts/files$ nano sudoer_jarvis
```
```
jarvis ALL=(ALL) NOPASSWD: ALL
```

```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass site.yml
BECOME password: 

PLAY [all] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [install updates (Ubuntu)] ***********************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.76]

TASK [install updates (Rocky)] ************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
skipping: [10.0.0.76]
ok: [10.0.0.213]

PLAY [all] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [create jarvis user] *****************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [add ssh key for jarvis] *************************************************************************************************************************
changed: [10.0.0.84]
changed: [10.0.0.106]
changed: [10.0.0.121]
changed: [10.0.0.213]
changed: [10.0.0.76]

TASK [add sudoers file for jarvis] ********************************************************************************************************************
changed: [10.0.0.121]
changed: [10.0.0.106]
changed: [10.0.0.84]
changed: [10.0.0.213]
changed: [10.0.0.76]

PLAY [workstations] ***********************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.76]

TASK [install unzip] **********************************************************************************************************************************
ok: [10.0.0.76]

TASK [install terraform] ******************************************************************************************************************************
ok: [10.0.0.76]

PLAY [web_servers] ************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ***************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] *********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [start httpd (Rocky)] ****************************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [change email address for admin (Rocky)] *********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [restart httpd (Rocky)] **************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.213]

TASK [copy default html file for site] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] ******************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] ************************************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] ***********************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ***************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ********************************************************************************************************************************************
10.0.0.106                 : ok=8    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=9    changed=2    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
10.0.0.213                 : ok=11   changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.76                  : ok=9    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=8    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```
## running our playbook as the user jarvis
- edit the ansible.cfg file
```
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
remote_user = jarvis
```
- run the playbook without the --ask-become-pass parameter, it should work without promptong you for a password
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook site.yml

PLAY [all] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [install updates (Ubuntu)] ***********************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.84]
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.76]

TASK [install updates (Rocky)] ************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
skipping: [10.0.0.76]
ok: [10.0.0.213]

PLAY [all] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.76]

TASK [create jarvis user] *****************************************************************************************************************************
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [add ssh key for jarvis] *************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [add sudoers file for jarvis] ********************************************************************************************************************
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.76]
ok: [10.0.0.213]

PLAY [workstations] ***********************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.76]

TASK [install unzip] **********************************************************************************************************************************
ok: [10.0.0.76]

TASK [install terraform] ******************************************************************************************************************************
ok: [10.0.0.76]

PLAY [web_servers] ************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ***************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] *********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [start httpd (Rocky)] ****************************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [change email address for admin (Rocky)] *********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [restart httpd (Rocky)] **************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.213]

TASK [copy default html file for site] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] ******************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] ************************************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] ***********************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ***************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ********************************************************************************************************************************************
10.0.0.106                 : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=9    changed=0    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
10.0.0.213                 : ok=11   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.76                  : ok=9    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=8    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```
