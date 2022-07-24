# templates

## create a templates file in roles/base/
```
anthony@ansible-control-host:~/ansible/scripts/roles/base$ mkdir templates
```
we want to make a template file of the ssh config file for ubuntu
```
anthony@ansible-control-host:~/ansible/scripts/roles/base$ cp /etc/ssh/sshd_config templates/sshd_config_ubuntu.j2
```
ansible uses the jinja2 format for templates (.j2 extension)

now the template for Rocky linux ssh config file, using `scp`
```
anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ scp 10.0.0.213:/etc/ssh/sshd_config sshd_config_rocky.j2
Enter passphrase for key '/home/anthony/.ssh/id_ed25519': 
scp: /etc/ssh/sshd_config: Permission denied
```
if you get the permission denied you can modify the user permissions for that file in the server and copy it and then change the permissions back

or use sudo and change the permissions and ownership of the file once it is copied, like this:
```
anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ sudo scp 10.0.0.213:/etc/ssh/sshd_config sshd_config_rocky.j2
[sudo] password for anthony: 
The authenticity of host '10.0.0.213 (10.0.0.213)' can't be established.
ED25519 key fingerprint is SHA256:VtA9qtOGUP1oNIjtGp+xP/7gLEQ+sasasasasasasas.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.0.213' (ED25519) to the list of known hosts.
root@10.0.0.213's password: 
sshd_config                                                                                                         100% 4269     2.0MB/s   00:00    
anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ ls
sshd_config_rocky.j2  sshd_config_ubuntu.j2
anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ ls -l
total 12
-rw------- 1 root    root    4269 Jul 24 18:58 sshd_config_rocky.j2
-rw-r--r-- 1 anthony anthony 3281 Jul 24 18:51 sshd_config_ubuntu.j2

anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ sudo chmod 0644 sshd_config_rocky.j2

anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ ls -l
total 12
-rw-r--r-- 1 root    root    4269 Jul 24 18:58 sshd_config_rocky.j2
-rw-r--r-- 1 anthony anthony 3281 Jul 24 18:51 sshd_config_ubuntu.j2

anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ sudo chown anthony sshd_config_rocky.j2 

anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ ls -l
total 12
-rw-r--r-- 1 anthony root    4269 Jul 24 18:58 sshd_config_rocky.j2
-rw-r--r-- 1 anthony anthony 3281 Jul 24 18:51 sshd_config_ubuntu.j2

anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ sudo chgrp anthony sshd_config_rocky.j2 

anthony@ansible-control-host:~/ansible/scripts/roles/base/templates$ ls -l
total 12
-rw-r--r-- 1 anthony anthony 4269 Jul 24 18:58 sshd_config_rocky.j2
-rw-r--r-- 1 anthony anthony 3281 Jul 24 18:51 sshd_config_ubuntu.j2
```
## modify your templates
add a variable for the allowed ssh user(s), doesnt really matter where on the file

- sshd_config_ubuntu.j2
```
(...)
AllowUsers {{ ssh_users }}
(...)
```

- sshd_config_rocky.j2
```
(...)
AllowUsers {{ ssh_users }}
(...)
```
## adding the variable and templates to the host variables files
- 10.0.0.120.yml, 10.0.0.105.yml, 10.0.0.83.yml, 10.0.0.75.yml
```
(...)
ssh_users: "anthony jarvis"
ssh_template_file: sshd_config_ubuntu.j2
```
- 10.0.0.213.yml
```
(...)
ssh_users: "anthony jarvis"
ssh_template_file: sshd_config_rocky.j2
```
## adding a task to base to copy the sshd_config file from the template
- .../roles/base/tasks/main.yml
```
- name: add ssh key for jarvis
  tags: always
  authorized_key: # module to add or remove ssh authorized keys for a user account
    user: jarvis
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBRsPEBBSXDFUrCKgcQQOqUUB8jWsasasasasasassasaasas ansible" # copy from ~/.ssh/ansible.pub

- name: generate sshd_config file from template
  tags: ssh
  template:
    src: "{{ ssh_template_file }}"
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: 0644
  notify: restart_sshd
```
## create the handler
```
anthony@ansible-control-host:~/ansible/scripts/roles/base$ mkdir handlers
anthony@ansible-control-host:~/ansible/scripts/roles/base$ cd handlers/
anthony@ansible-control-host:~/ansible/scripts/roles/base/handlers$ nano main.yml
```
- ...roles/base/handlers/main.yml
```
-name: restart_sshd
 service: 
   name: sshd
   state: restarted
```
## run the playbook
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook site.yml

PLAY [all] *******************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.105]
ok: [10.0.0.120]
ok: [10.0.0.83]
ok: [10.0.0.213]
ok: [10.0.0.75]

TASK [update repo cache (Ubuntu)] ********************************************************************************************************************
skipping: [10.0.0.213]
changed: [10.0.0.83]
changed: [10.0.0.120]
changed: [10.0.0.105]
changed: [10.0.0.75]

TASK [update repo cache (Rocky)] *********************************************************************************************************************
skipping: [10.0.0.120]
skipping: [10.0.0.105]
skipping: [10.0.0.83]
skipping: [10.0.0.75]
ok: [10.0.0.213]

PLAY [all] *******************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.105]
ok: [10.0.0.83]
ok: [10.0.0.213]
ok: [10.0.0.75]

TASK [base : add ssh key for jarvis] *****************************************************************************************************************
ok: [10.0.0.83]
ok: [10.0.0.105]
ok: [10.0.0.213]
ok: [10.0.0.120]
ok: [10.0.0.75]

TASK [base : generate sshd_config file from template] ************************************************************************************************
changed: [10.0.0.120]
changed: [10.0.0.105]
changed: [10.0.0.83]
changed: [10.0.0.75]
changed: [10.0.0.213]

RUNNING HANDLER [base : restart_sshd] ****************************************************************************************************************
changed: [10.0.0.83]
changed: [10.0.0.105]
changed: [10.0.0.120]
changed: [10.0.0.213]
changed: [10.0.0.75]

PLAY [workstations] **********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.75]

TASK [workstations : install unzip] ******************************************************************************************************************
ok: [10.0.0.75]

TASK [workstations : install terraform] **************************************************************************************************************
ok: [10.0.0.75]

PLAY [web_servers] ***********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

TASK [web_servers : install apache and php] **********************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

TASK [web_servers : start and enable apache service] *************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

TASK [web_servers : change email address for admin (Rocky)] ******************************************************************************************
skipping: [10.0.0.120]
ok: [10.0.0.213]

TASK [web_servers : copy default html file for site] *************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

PLAY [db_servers] ************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.105]

TASK [db_servers : install mariadb] ******************************************************************************************************************
ok: [10.0.0.105]

PLAY [file_servers] **********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.83]

TASK [file_servers : install samba] ******************************************************************************************************************
ok: [10.0.0.83]

PLAY RECAP *******************************************************************************************************************************************
10.0.0.105                 : ok=8    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.120                 : ok=10   changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.213                 : ok=11   changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.75                  : ok=9    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.83                  : ok=8    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
## check the /etc/ssh/sshd_config file on the servers
they should now have the users we allowed

```
anthony@ansible-control-host:~$ cat /etc/ssh/sshd_config

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.
(...)

AllowUsers anthony jarvis
(...)
```