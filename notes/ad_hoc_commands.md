# ad hoc commands

## install ansible in our ansible-control-host
```
sudo apt install ansible
```

## create a file named inventory with the ip addresses of the servers we are going to manage with ansible
```
anthony@ansible-control-host:~/ansible/scripts$ nano inventory
```
## git add inventory
```
anthony@ansible-control-host:~/ansible/scripts$ git add inventory 
```
## git commit and push
```
anthony@ansible-control-host:~/ansible/scripts$ git commit -m "first version of inventory file added, 3 hosts"
[master bc7a460] first version of inventory file added, 3 hosts
 1 file changed, 3 insertions(+)
 create mode 100644 scripts/inventory

anthony@ansible-control-host:~/ansible/scripts$ git push origin master
Enter passphrase for key '/home/anthony/.ssh/id_ed25519': 
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 397 bytes | 198.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:anthonychl/ansible.git
   7886b1b..bc7a460  master -> master
```
## first ansible command 
making a connection with each server inside the inventory and making sure everything is working: `--key-file` is our ansible ssh key,`-i` is for the input file (inventory), `-m` is for module in this case `ping` which is what makes the connection, it is not quite as in the linux ping command.
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all --key-file ~/.ssh/ansible -i inventory -m ping
10.0.0.84 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
10.0.0.121 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
10.0.0.106 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
we got three successes which means ansible was able to connect to all three servers in the inventory.

## create a local ansible config file, this file will override the file /etc/ansible/ansible.cfg
```
anthony@ansible-control-host:~/ansible/scripts$ nano ansible.cfg

anthony@ansible-control-host:~/ansible/scripts$ ls
ansible.cfg  inventory  scripts_readme.md

anthony@ansible-control-host:~/ansible/scripts$ cat ansible.cfg 
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
```
## shortened version of the previous command
`ansible all -m ping`

since we set the defaults in the config file now we can omit some parameters 
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m ping
10.0.0.84 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
10.0.0.121 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
10.0.0.106 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
## list hosts 
`ansible all --list-hosts`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all --list-hosts
  hosts (3):
    10.0.0.121
    10.0.0.106
    10.0.0.84
```

## get info about all the hosts 
`ansible all -m gather_facts`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m gather_facts
10.0.0.121 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.0.121"
        ],
        "ansible_all_ipv6_addresses": [
            "2601:583:c205:61b0::62af",
            "fe80::c8e0:f0ff:fe66:7fe9"
        ],
        "ansible_apparmor": {
            "status": "enabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "04/01/2014",
        "ansible_bios_vendor": "SeaBIOS",
        "ansible_bios_version": "rel-1.15.0-0-g2dd4b9b3f840-prebuilt.qemu.org",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "NA",
        "ansible_board_serial": "NA",
        "ansible_board_vendor": "NA",
        "ansible_board_version": "NA",
        "ansible_chassis_asset_tag": "NA",
        "ansible_chassis_serial": "NA",
        "ansible_chassis_vendor": "QEMU",
        "ansible_chassis_version": "pc-i440fx-6.2",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-5.15.0-33-generic",
            "ro": true,
            "root": "/dev/mapper/ubuntu--vg-ubuntu--lv"
        },
        "ansible_date_time": {
            "date": "2022-07-03",
            "day": "03",
            "epoch": "1656819135",
            "hour": "03",
            "iso8601": "2022-07-03T03:32:15Z",
            "iso8601_basic": "20220703T033215026013",
            "iso8601_basic_short": "20220703T033215",
            "iso8601_micro": "2022-07-03T03:32:15.026013Z",
            "minute": "32",
            "month": "07",
            "second": "15",
            "time": "03:32:15",
            "tz": "UTC",
            "tz_offset": "+0000",
            "weekday": "Sunday",
            "weekday_number": "0",
            "weeknumber": "26",
            "year": "2022"
        },
        "ansible_default_ipv4": {
            "address": "10.0.0.121",
            "alias": "ens18",
            
        [truncated, there's a lot of data output]

```
## get info of a host 
`ansible all -m gather_facts --limit 10.0.0.84`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m gather_facts --limit 10.0.0.84
10.0.0.84 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.0.84"
        ],
        "ansible_all_ipv6_addresses": [
            "2601:583:c205:61b0::d2c0",
            "fe80::b0f9:baff:fe30:ce3a"
        ],
        "ansible_apparmor": {
            "status": "enabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "04/01/2014",
        "ansible_bios_vendor": "SeaBIOS",
        "ansible_bios_version": "rel-1.15.0-0-g2dd4b9b3f840-prebuilt.qemu.org",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "NA",
        "ansible_board_serial": "NA",
        "ansible_board_vendor": "NA",
        "ansible_board_version": "NA",
        "ansible_chassis_asset_tag": "NA",
        "ansible_chassis_serial": "NA",
        "ansible_chassis_vendor": "QEMU",
        "ansible_chassis_version": "pc-i440fx-6.2",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-5.15.0-39-generic",
            "ro": true,
            "root": "/dev/mapper/ubuntu--vg-ubuntu--lv"
        },
        "ansible_date_time": {
            "date": "2022-07-03",
            "day": "03",
            "epoch": "1656819667",
            "hour": "03",
            "iso8601": "2022-07-03T03:41:07Z",
            "iso8601_basic": "20220703T034107186339",
            "iso8601_basic_short": "20220703T034107",
            "iso8601_micro": "2022-07-03T03:41:07.186339Z",
            "minute": "41",
            "month": "07",
            "second": "07",
            "time": "03:41:07",
            "tz": "UTC",
            "tz_offset": "+0000",
            "weekday": "Sunday",
            "weekday_number": "0",
            "weeknumber": "26",
            "year": "2022"
        },
        "ansible_default_ipv4": {
            "address": "10.0.0.84",
            "alias": "ens18",
            "broadcast": "",

             [truncated, there's a lot of data output]

```
## adding ansible.cfg to our git repository
```
anthony@ansible-control-host:~/ansible/scripts$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	ansible.cfg

nothing added to commit but untracked files present (use "git add" to track)

anthony@ansible-control-host:~/ansible/scripts$ git add ansible.cfg 

anthony@ansible-control-host:~/ansible/scripts$ git commit -m "ansible config file added"
[master 1471c2e] ansible config file added
 1 file changed, 4 insertions(+)
 create mode 100644 scripts/ansible.cfg
 
anthony@ansible-control-host:~/ansible/scripts$ git push origin master 
Enter passphrase for key '/home/anthony/.ssh/id_ed25519': 
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 457 bytes | 457.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:anthonychl/ansible.git
   bc7a460..1471c2e  master -> master
```
