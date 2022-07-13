# improving or optimizing a playbook

## v1 installing multiple packages on a single step
```
---

- hosts: all
  become: true
  tasks:

  - name: update repository index (apt)
    apt:
      update_cache: yes
    when: ansible_distribution in ["Ubuntu"," Debian"] 

  - name: install apache2 and php (apt)
    apt: 
      name: 
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution in ["Ubuntu"," Debian"]

  - name: update repository index (dnf)
    dnf:
      update_cache: yes
    when: ansible_distribution == "Rocky" 

  - name: install apache package (httpd) and php (dnf)
    dnf: 
      name:
        - httpd
        - php
      state: latest
    when: ansible_distribution == "Rocky"
```
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml
BECOME password: 

PLAY [all] ******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]

TASK [update repository index (apt)] ****************************************************************************************************************
skipping: [10.0.0.213]
changed: [10.0.0.121]
changed: [10.0.0.84]
changed: [10.0.0.106]

TASK [install apache2 and php (apt)] ****************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]

TASK [update repository index (dnf)] ****************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

TASK [install apache package (httpd) and php (dnf)] *************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY RECAP ******************************************************************************************************************************************
10.0.0.106                 : ok=3    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=3    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.213                 : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.84                  : ok=3    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
```

## v2 moving the update index step together with the packages installation
```
---

- hosts: all
  become: true
  tasks:

  - name: update repo index and install apache2 and php (apt)
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
      update_cache: yes
    when: ansible_distribution in ["Ubuntu"," Debian"]

  - name: update repo index and install apache (httpd) and php (dnf)
    dnf:
      name:
        - httpd
        - php
      state: latest
      update_cache: yes
    when: ansible_distribution == "Rocky"
```
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml
BECOME password: 

PLAY [all] ******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.213]
ok: [10.0.0.84]

TASK [update repo index and install apache2 and php (apt)] ******************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.84]
ok: [10.0.0.121]
ok: [10.0.0.106]

TASK [update repo index and install apache package (httpd) and php (dnf)] ***************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY RECAP ******************************************************************************************************************************************
10.0.0.106                 : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.121                 : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.213                 : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```

## merging everything into one task
using `package` and variables
```
---

- hosts: all
  become: true
  tasks:

  - name: update repo index and install apache and php
    package: # using package for ansible to detect whether to use apt or dnf
      name:
        - "{{ apache_pkg }}" # these are variables
        - "{{ php_pkg }}"
      state: latest
      update_cache: yes
```
declaring the variables in the inventory file
```
10.0.0.121  apache_pkg=apache2 php_pkg=libapache2-mod-php
10.0.0.106  apache_pkg=apache2 php_pkg=libapache2-mod-php
10.0.0.84  apache_pkg=apache2 php_pkg=libapache2-mod-php
10.0.0.213  apache_pkg=httpd php_pkg=php
```
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml
BECOME password: 

PLAY [all] ******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]

TASK [update repo index and install apache and php] *************************************************************************************************
ok: [10.0.0.213]
ok: [10.0.0.84]
ok: [10.0.0.121]
ok: [10.0.0.106]

PLAY RECAP ******************************************************************************************************************************************
10.0.0.106                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.121                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.213                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.84                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
this last example may or may not be optimal depending on the scenario because we are declaring variables in the inventory file, but it's an example of what can be done.