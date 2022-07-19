# managing services
in this example we are tackling the problem in the "when_conditional.md" file where we had to start and enable our httpd service manually after ansible installed it. Unlike in Ubuntu, RedHat based distros like Rocky linux and CentOS dont start services by default after installation. 

## the `service` module, starting and enabling a service  

- site.yml
```
(...)
- hosts: web_servers
(...)

  - name: start httpd (Rocky)
    tags: apache,httpd,rocky
    service: # allows to manage services
      name: httpd # name of the service we want to manage
      state: started # set the state of the service
      enabled: yes # set the service to start at boot
    when: ansible_distribution == "Rocky"

(...) 
```
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass site.yml
BECOME password: 

PLAY [all] *****************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [install updates (Ubuntu)] ********************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.76]
ok: [10.0.0.121]

TASK [install updates (Rocky)] *********************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
skipping: [10.0.0.76]
ok: [10.0.0.213]

PLAY [workstations] ********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.76]

TASK [install unzip] *******************************************************************************************************************************************
ok: [10.0.0.76]

TASK [install terraform] ***************************************************************************************************************************************
ok: [10.0.0.76]

PLAY [web_servers] *********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] ******************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [start httpd (Rocky)] *************************************************************************************************************************************
skipping: [10.0.0.121]
changed: [10.0.0.213]

TASK [copy default html file for site] *************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] **********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] ***************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] *********************************************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] ********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ************************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP *****************************************************************************************************************************************************
10.0.0.106                 : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=5    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.213                 : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.76                  : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
```
[anthony@server-4 ~]$ systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
   Active: active (running) since Mon 2022-07-18 22:16:13 EDT; 1min 17s ago
     Docs: man:httpd.service(8)
 Main PID: 2392 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 4936)
   Memory: 26.3M
   CGroup: /system.slice/httpd.service
           ├─2392 /usr/sbin/httpd -DFOREGROUND
           ├─2411 /usr/sbin/httpd -DFOREGROUND
           ├─2412 /usr/sbin/httpd -DFOREGROUND
           ├─2413 /usr/sbin/httpd -DFOREGROUND
           └─2414 /usr/sbin/httpd -DFOREGROUND
[anthony@server-4 ~]$ 
```
## using `lineinfile` and restarting a service
example of how we can modify a line in a config file, in this case we want to change the email address

-  /etc/httpd/conf/httpd.conf
```
#
# ServerAdmin: Your address, where problems with the server should be
# e-mailed.  This address appears on some server-generated pages, such
# as error documents.  e.g. admin@your-domain.com
#
ServerAdmin root@localhost
```

- site.yml
```
(...)
  - name: change email address for admin (Rocky)
    tags: apache,httpd,rocky
    lineinfile: # module to modify a line in a file
      path: /etc/httpd/conf/httpd.conf 
      regexp: '^ServerAdmin' # regular expression to find the line to be modified
      line: ServerAdmin anthony@localhost # text of the new line
    when: ansible_distribution == "Rocky"
    register: httpd_var # setting a variable to capture the state (if something changed)

  - name: restart httpd (Rocky)
    tags: apache,httpd,rocky
    service:
      name: httpd
      state: restarted  
    when: httpd_var.changed # run this task if httpd_var changed
(...)
```

```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass site.yml
BECOME password: 

PLAY [all] *****************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [install updates (Ubuntu)] ********************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.76]

TASK [install updates (Rocky)] *********************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
skipping: [10.0.0.76]
ok: [10.0.0.213]

PLAY [workstations] ********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.76]

TASK [install unzip] *******************************************************************************************************************************************
ok: [10.0.0.76]

TASK [install terraform] ***************************************************************************************************************************************
ok: [10.0.0.76]

PLAY [web_servers] *********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] ******************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [start httpd (Rocky)] *************************************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [change email address for admin (Rocky)] ******************************************************************************************************************
skipping: [10.0.0.121]
changed: [10.0.0.213]

TASK [restart httpd (Rocky)] ***********************************************************************************************************************************
skipping: [10.0.0.121]
changed: [10.0.0.213]

TASK [copy default html file for site] *************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] **********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] ***************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] *********************************************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] ********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ************************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP *****************************************************************************************************************************************************
10.0.0.106                 : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=5    changed=0    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
10.0.0.213                 : ok=8    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.76                  : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
-  /etc/httpd/conf/httpd.conf
```
#
# ServerAdmin: Your address, where problems with the server should be
# e-mailed.  This address appears on some server-generated pages, such
# as error documents.  e.g. admin@your-domain.com
#
ServerAdmin anthony@localhost
```


