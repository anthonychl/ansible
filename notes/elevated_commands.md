# commands with elevated privileges
running commands that need sudo privileges, there's a condition for the examples below, we assume that the password for every server is the same.

## updating apt cache on all servers (the equivalent of sudo apt update)
`ansible all -m apt -a update_cache=yes --become --ask-become-pass`
- -m module
- -a arguments, if there's more than one argument, put inside double quotes "arg1 arg2"
- --become by default is sudo
- --ask-become-pass ask for sudo password
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m apt -a update_cache=yes --become --ask-become-pass
BECOME password: 
10.0.0.121 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": true,
    "changed": true
}
10.0.0.106 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": true,
    "changed": true
}
10.0.0.84 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": true,
    "changed": true
}
```
## installing a package (vim-nox)
`ansible all -m apt -a name=vim-nox --become --ask-become-pass`
- -name  the package to be installed
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m apt -a name=vim-nox --become --ask-become-pass
BECOME password: 
10.0.0.106 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": false,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": ...
    "stdout_lines": ...
        "Setting up vim-nox (2:8.2.3995-1ubuntu2) ...",
        "update-alternatives: using /usr/bin/vim.nox to provide /usr/bin/vim (vim) in auto mode",
        "update-alternatives: using /usr/bin/vim.nox to provide /usr/bin/vimdiff (vimdiff) in auto mode",
        "update-alternatives: using /usr/bin/vim.nox to provide /usr/bin/rvim (rvim) in auto mode",
        "update-alternatives: using /usr/bin/vim.nox to provide /usr/bin/rview (rview) in auto mode",
        "update-alternatives: using /usr/bin/vim.nox to provide /usr/bin/vi (vi) in auto mode",
        "update-alternatives: using /usr/bin/vim.nox to provide /usr/bin/view (view) in auto mode",
        "update-alternatives: using /usr/bin/vim.nox to provide /usr/bin/ex (ex) in auto mode",
        "Setting up ruby (1:3.0~exp1) ...",
        "Setting up ruby-rubygems (3.3.5-2) ...",
        "Processing triggers for man-db (2.10.2-1) ...",
        "Processing triggers for mailcap (3.70+nmu1ubuntu1) ...",
        "Processing triggers for libc-bin (2.35-0ubuntu3) ...",
        "NEEDRESTART-VER: 3.5",
        "NEEDRESTART-KCUR: 5.15.0-39-generic",
        "NEEDRESTART-KEXP: 5.15.0-39-generic",
        "NEEDRESTART-KSTA: 1"
    ]
}
        [truncated output]
```
## trying to install a package that already exists
`ansible all -m apt -a name=tmux --become --ask-become-pass`

check how "changed": false, if a package is found it wont be installed again or updated

```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m apt -a name=tmux --become --ask-become-pass
BECOME password: 
10.0.0.121 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": false,
    "changed": false
}
10.0.0.106 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": false,
    "changed": false
}
10.0.0.84 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": false,
    "changed": false
}

```
## update a package already installed to latest version
`ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass
BECOME password: 
10.0.0.84 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1656942870,
    "cache_updated": false,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nSuggested packages:\n  zenity | kdialog\nThe following packages will be upgraded:\n  snapd\n1 upgraded, 0 newly installed, 0 to remove and 20 not upgraded.\nNeed to get 21.6 MB of archives.\nAfter this operation, 79.9 kB of additional disk space will be used.\nGet:1 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 snapd amd64 2.55.5+22.04 [21.6 MB]\nFetched 21.6 MB in 4s (4799 kB/s)\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 112493 files and directories currently installed.)\r\nPreparing to unpack .../snapd_2.55.5+22.04_amd64.deb ...\r\nUnpacking snapd (2.55.5+22.04) over (2.55.3+22.04ubuntu1) ...\r\nSetting up snapd (2.55.5+22.04) ...\r\nsnapd.failure.service is a disabled or a static unit not running, not starting it.\r\nsnapd.snap-repair.service is a disabled or a static unit not running, not starting it.\r\nProcessing triggers for dbus (1.12.20-2ubuntu4) ...\r\nProcessing triggers for mailcap (3.70+nmu1ubuntu1) ...\r\nProcessing triggers for man-db (2.10.2-1) ...\r\nNEEDRESTART-VER: 3.5\nNEEDRESTART-KCUR: 5.15.0-39-generic\nNEEDRESTART-KEXP: 5.15.0-39-generic\nNEEDRESTART-KSTA: 1\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "Suggested packages:",
        "  zenity | kdialog",
        "The following packages will be upgraded:",
        "  snapd",
        "1 upgraded, 0 newly installed, 0 to remove and 20 not upgraded.",
        "Need to get 21.6 MB of archives.",
        "After this operation, 79.9 kB of additional disk space will be used.",
        "Get:1 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 snapd amd64 2.55.5+22.04 [21.6 MB]",
        "Fetched 21.6 MB in 4s (4799 kB/s)",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 112493 files and directories currently installed.)",
        "Preparing to unpack .../snapd_2.55.5+22.04_amd64.deb ...",
        "Unpacking snapd (2.55.5+22.04) over (2.55.3+22.04ubuntu1) ...",
        "Setting up snapd (2.55.5+22.04) ...",
        "snapd.failure.service is a disabled or a static unit not running, not starting it.",
        "snapd.snap-repair.service is a disabled or a static unit not running, not starting it.",
        "Processing triggers for dbus (1.12.20-2ubuntu4) ...",
        "Processing triggers for mailcap (3.70+nmu1ubuntu1) ...",
        "Processing triggers for man-db (2.10.2-1) ...",
        "NEEDRESTART-VER: 3.5",
        "NEEDRESTART-KCUR: 5.15.0-39-generic",
        "NEEDRESTART-KEXP: 5.15.0-39-generic",
        "NEEDRESTART-KSTA: 1"
    ]
}
        [output truncated]
```
## upgrading packages on all servers (the equivalent of sudo apt dist-upgrade)
`ansible all -m apt -a "upgrade=dist" --become --ask-become-pass`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m apt -a "upgrade=dist" --become --ask-become-pass
BECOME password: 
10.0.0.84 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "msg": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nCalculating upgrade...\nThe following NEW packages will be installed:\n  linux-headers-5.15.0-40 linux-headers-5.15.0-40-generic\n  linux-image-5.15.0-40-generic linux-modules-5.15.0-40-generic\n  linux-modules-extra-5.15.0-40-generic\nThe following packages will be upgraded:\n  apparmor libapparmor1 libldap-2.5-0 libldap-common libnss-systemd\n  libpam-systemd libsystemd0 libudev1 linux-generic linux-headers-generic\n  linux-image-generic python3-distupgrade python3-software-properties\n  software-properties-common systemd systemd-sysv systemd-timesyncd\n  ubuntu-advantage-tools ubuntu-release-upgrader-core udev\n20 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.\nNeed to get 119 MB of archives.\nAfter this operation, 561 MB of additional disk space will be used.\nGet:1 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd-timesyncd amd64 249.11-0ubuntu3.3 [31.2 kB]\nGet:2 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libsystemd0 amd64 249.11-0ubuntu3.3 [318 kB]\nGet:3 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libnss-systemd amd64 249.11-0ubuntu3.3 [133 kB]\nGet:4 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd-sysv amd64 249.11-0ubuntu3.3 [10.5 kB]\nGet:5 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libpam-systemd amd64 249.11-0ubuntu3.3 [203 kB]\nGet:6 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd amd64 249.11-0ubuntu3.3 [4579 kB]\nGet:7 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 udev amd64 249.11-0ubuntu3.3 [1557 kB]\nGet:8 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libudev1 amd64 249.11-0ubuntu3.3 [77.4 kB]\nGet:9 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libapparmor1 amd64 3.0.4-2ubuntu2.1 [38.7 kB]\nGet:10 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 ubuntu-advantage-tools amd64 27.9~22.04.1 [849 kB]\nGet:11 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 apparmor amd64 3.0.4-2ubuntu2.1 [590 kB]\nGet:12 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 ubuntu-release-upgrader-core all 1:22.04.11 [26.2 kB]\nGet:13 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-distupgrade all 1:22.04.11 [106 kB]\nGet:14 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libldap-2.5-0 amd64 2.5.12+dfsg-0ubuntu0.22.04.1 [184 kB]\nGet:15 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libldap-common all 2.5.12+dfsg-0ubuntu0.22.04.1 [16.4 kB]\nGet:16 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-modules-5.15.0-40-generic amd64 5.15.0-40.43 [22.0 MB]\nGet:17 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-image-5.15.0-40-generic amd64 5.15.0-40.43 [10.9 MB]\nGet:18 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-modules-extra-5.15.0-40-generic amd64 5.15.0-40.43 [62.2 MB]\nGet:19 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-generic amd64 5.15.0.40.42 [1700 B]\nGet:20 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-image-generic amd64 5.15.0.40.42 [2568 B]\nGet:21 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-5.15.0-40 all 5.15.0-40.43 [12.2 MB]\nGet:22 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-5.15.0-40-generic amd64 5.15.0-40.43 [2780 kB]\nGet:23 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-generic amd64 5.15.0.40.42 [2430 B]\nGet:24 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 software-properties-common all 0.99.22.2 [14.1 kB]\nGet:25 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-software-properties all 0.99.22.2 [28.8 kB]\nPreconfiguring packages ...\nFetched 119 MB in 49s (2441 kB/s)\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 112493 files and directories currently installed.)\r\nPreparing to unpack .../systemd-timesyncd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking systemd-timesyncd (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../libsystemd0_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libsystemd0:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nSetting up libsystemd0:amd64 (249.11-0ubuntu3.3) ...\r\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 112493 files and directories currently installed.)\r\nPreparing to unpack .../0-libnss-systemd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libnss-systemd:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../1-systemd-sysv_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking systemd-sysv (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../2-libpam-systemd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libpam-systemd:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../3-systemd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking systemd (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../4-udev_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking udev (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../5-libudev1_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libudev1:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nSetting up libudev1:amd64 (249.11-0ubuntu3.3) ...\r\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 112493 files and directories currently installed.)\r\nPreparing to unpack .../00-libapparmor1_3.0.4-2ubuntu2.1_amd64.deb ...\r\nUnpacking libapparmor1:amd64 (3.0.4-2ubuntu2.1) over (3.0.4-2ubuntu2) ...\r\nPreparing to unpack .../01-ubuntu-advantage-tools_27.9~22.04.1_amd64.deb ...\r\nUnpacking ubuntu-advantage-tools (27.9~22.04.1) over (27.8~22.04.1) ...\r\nPreparing to unpack .../02-apparmor_3.0.4-2ubuntu2.1_amd64.deb ...\r\nUnpacking apparmor (3.0.4-2ubuntu2.1) over (3.0.4-2ubuntu2) ...\r\nPreparing to unpack .../03-ubuntu-release-upgrader-core_1%3a22.04.11_all.deb ...\r\nUnpacking ubuntu-release-upgrader-core (1:22.04.11) over (1:22.04.10) ...\r\nPreparing to unpack .../04-python3-distupgrade_1%3a22.04.11_all.deb ...\r\nUnpacking python3-distupgrade (1:22.04.11) over (1:22.04.10) ...\r\nPreparing to unpack .../05-libldap-2.5-0_2.5.12+dfsg-0ubuntu0.22.04.1_amd64.deb ...\r\nUnpacking libldap-2.5-0:amd64 (2.5.12+dfsg-0ubuntu0.22.04.1) over (2.5.11+dfsg-1~exp1ubuntu3.1) ...\r\nPreparing to unpack .../06-libldap-common_2.5.12+dfsg-0ubuntu0.22.04.1_all.deb ...\r\nUnpacking libldap-common (2.5.12+dfsg-0ubuntu0.22.04.1) over (2.5.11+dfsg-1~exp1ubuntu3.1) ...\r\nSelecting previously unselected package linux-modules-5.15.0-40-generic.\r\nPreparing to unpack .../07-linux-modules-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-modules-5.15.0-40-generic (5.15.0-40.43) ...\r\nSelecting previously unselected package linux-image-5.15.0-40-generic.\r\nPreparing to unpack .../08-linux-image-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-image-5.15.0-40-generic (5.15.0-40.43) ...\r\nSelecting previously unselected package linux-modules-extra-5.15.0-40-generic.\r\nPreparing to unpack .../09-linux-modules-extra-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-modules-extra-5.15.0-40-generic (5.15.0-40.43) ...\r\nPreparing to unpack .../10-linux-generic_5.15.0.40.42_amd64.deb ...\r\nUnpacking linux-generic (5.15.0.40.42) over (5.15.0.39.40) ...\r\nPreparing to unpack .../11-linux-image-generic_5.15.0.40.42_amd64.deb ...\r\nUnpacking linux-image-generic (5.15.0.40.42) over (5.15.0.39.40) ...\r\nSelecting previously unselected package linux-headers-5.15.0-40.\r\nPreparing to unpack .../12-linux-headers-5.15.0-40_5.15.0-40.43_all.deb ...\r\nUnpacking linux-headers-5.15.0-40 (5.15.0-40.43) ...\r\nSelecting previously unselected package linux-headers-5.15.0-40-generic.\r\nPreparing to unpack .../13-linux-headers-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-headers-5.15.0-40-generic (5.15.0-40.43) ...\r\nPreparing to unpack .../14-linux-headers-generic_5.15.0.40.42_amd64.deb ...\r\nUnpacking linux-headers-generic (5.15.0.40.42) over (5.15.0.39.40) ...\r\nPreparing to unpack .../15-software-properties-common_0.99.22.2_all.deb ...\r\nUnpacking software-properties-common (0.99.22.2) over (0.99.22.1) ...\r\nPreparing to unpack .../16-python3-software-properties_0.99.22.2_all.deb ...\r\nUnpacking python3-software-properties (0.99.22.2) over (0.99.22.1) ...\r\nSetting up libapparmor1:amd64 (3.0.4-2ubuntu2.1) ...\r\nSetting up systemd (249.11-0ubuntu3.3) ...\r\nSetting up python3-distupgrade (1:22.04.11) ...\r\nSetting up libldap-common (2.5.12+dfsg-0ubuntu0.22.04.1) ...\r\nSetting up libldap-2.5-0:amd64 (2.5.12+dfsg-0ubuntu0.22.04.1) ...\r\nSetting up apparmor (3.0.4-2ubuntu2.1) ...\r\nInstalling new version of config file /etc/apparmor.d/abstractions/exo-open ...\r\nReloading AppArmor profiles \r\nSkipping profile in /etc/apparmor.d/disable: usr.sbin.rsyslogd\r\nSetting up python3-software-properties (0.99.22.2) ...\r\nSetting up systemd-timesyncd (249.11-0ubuntu3.3) ...\r\nSetting up udev (249.11-0ubuntu3.3) ...\r\nSetting up ubuntu-release-upgrader-core (1:22.04.11) ...\r\nSetting up linux-headers-5.15.0-40 (5.15.0-40.43) ...\r\nSetting up ubuntu-advantage-tools (27.9~22.04.1) ...\r\nInstalling new version of config file /etc/ubuntu-advantage/uaclient.conf ...\r\nCreated symlink /etc/systemd/system/multi-user.target.wants/ubuntu-advantage.service -> /lib/systemd/system/ubuntu-advantage.service.\r\nSetting up linux-headers-5.15.0-40-generic (5.15.0-40.43) ...\r\nSetting up systemd-sysv (249.11-0ubuntu3.3) ...\r\nSetting up libnss-systemd:amd64 (249.11-0ubuntu3.3) ...\r\nSetting up linux-headers-generic (5.15.0.40.42) ...\r\nSetting up software-properties-common (0.99.22.2) ...\r\nSetting up libpam-systemd:amd64 (249.11-0ubuntu3.3) ...\r\nSetting up linux-image-5.15.0-40-generic (5.15.0-40.43) ...\r\nI: /boot/vmlinuz.old is now a symlink to vmlinuz-5.15.0-39-generic\r\nI: /boot/initrd.img.old is now a symlink to initrd.img-5.15.0-39-generic\r\nI: /boot/vmlinuz is now a symlink to vmlinuz-5.15.0-40-generic\r\nI: /boot/initrd.img is now a symlink to initrd.img-5.15.0-40-generic\r\nSetting up linux-modules-extra-5.15.0-40-generic (5.15.0-40.43) ...\r\nSetting up linux-image-generic (5.15.0.40.42) ...\r\nSetting up linux-modules-5.15.0-40-generic (5.15.0-40.43) ...\r\nSetting up linux-generic (5.15.0.40.42) ...\r\nProcessing triggers for libc-bin (2.35-0ubuntu3) ...\r\nProcessing triggers for man-db (2.10.2-1) ...\r\nProcessing triggers for dbus (1.12.20-2ubuntu4) ...\r\nProcessing triggers for initramfs-tools (0.140ubuntu13) ...\r\nupdate-initramfs: Generating /boot/initrd.img-5.15.0-39-generic\r\nProcessing triggers for linux-image-5.15.0-40-generic (5.15.0-40.43) ...\r\n/etc/kernel/postinst.d/initramfs-tools:\r\nupdate-initramfs: Generating /boot/initrd.img-5.15.0-40-generic\r\n/etc/kernel/postinst.d/zz-update-grub:\r\nSourcing file `/etc/default/grub'\r\nSourcing file `/etc/default/grub.d/init-select.cfg'\r\nGenerating grub configuration file ...\r\nFound linux image: /boot/vmlinuz-5.15.0-40-generic\r\nFound initrd image: /boot/initrd.img-5.15.0-40-generic\r\nFound linux image: /boot/vmlinuz-5.15.0-39-generic\r\nFound initrd image: /boot/initrd.img-5.15.0-39-generic\r\nFound linux image: /boot/vmlinuz-5.15.0-33-generic\r\nFound initrd image: /boot/initrd.img-5.15.0-33-generic\r\nWarning: os-prober will not be executed to detect other bootable partitions.\r\nSystems on them will not be added to the GRUB boot configuration.\r\nCheck GRUB_DISABLE_OS_PROBER documentation entry.\r\ndone\r\nNEEDRESTART-VER: 3.5\nNEEDRESTART-KCUR: 5.15.0-39-generic\nNEEDRESTART-KEXP: 5.15.0-40-generic\nNEEDRESTART-KSTA: 3\nNEEDRESTART-SVC: apache2.service\nNEEDRESTART-SVC: dbus.service\nNEEDRESTART-SVC: ModemManager.service\nNEEDRESTART-SVC: multipathd.service\nNEEDRESTART-SVC: networkd-dispatcher.service\nNEEDRESTART-SVC: packagekit.service\nNEEDRESTART-SVC: polkit.service\nNEEDRESTART-SVC: qemu-guest-agent.service\nNEEDRESTART-SVC: rsyslog.service\nNEEDRESTART-SVC: ssh.service\nNEEDRESTART-SVC: systemd-logind.service\nNEEDRESTART-SVC: udisks2.service\nNEEDRESTART-SVC: unattended-upgrades.service\nNEEDRESTART-SVC: user@1000.service\n",
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nCalculating upgrade...\nThe following NEW packages will be installed:\n  linux-headers-5.15.0-40 linux-headers-5.15.0-40-generic\n  linux-image-5.15.0-40-generic linux-modules-5.15.0-40-generic\n  linux-modules-extra-5.15.0-40-generic\nThe following packages will be upgraded:\n  apparmor libapparmor1 libldap-2.5-0 libldap-common libnss-systemd\n  libpam-systemd libsystemd0 libudev1 linux-generic linux-headers-generic\n  linux-image-generic python3-distupgrade python3-software-properties\n  software-properties-common systemd systemd-sysv systemd-timesyncd\n  ubuntu-advantage-tools ubuntu-release-upgrader-core udev\n20 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.\nNeed to get 119 MB of archives.\nAfter this operation, 561 MB of additional disk space will be used.\nGet:1 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd-timesyncd amd64 249.11-0ubuntu3.3 [31.2 kB]\nGet:2 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libsystemd0 amd64 249.11-0ubuntu3.3 [318 kB]\nGet:3 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libnss-systemd amd64 249.11-0ubuntu3.3 [133 kB]\nGet:4 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd-sysv amd64 249.11-0ubuntu3.3 [10.5 kB]\nGet:5 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libpam-systemd amd64 249.11-0ubuntu3.3 [203 kB]\nGet:6 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd amd64 249.11-0ubuntu3.3 [4579 kB]\nGet:7 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 udev amd64 249.11-0ubuntu3.3 [1557 kB]\nGet:8 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libudev1 amd64 249.11-0ubuntu3.3 [77.4 kB]\nGet:9 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libapparmor1 amd64 3.0.4-2ubuntu2.1 [38.7 kB]\nGet:10 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 ubuntu-advantage-tools amd64 27.9~22.04.1 [849 kB]\nGet:11 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 apparmor amd64 3.0.4-2ubuntu2.1 [590 kB]\nGet:12 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 ubuntu-release-upgrader-core all 1:22.04.11 [26.2 kB]\nGet:13 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-distupgrade all 1:22.04.11 [106 kB]\nGet:14 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libldap-2.5-0 amd64 2.5.12+dfsg-0ubuntu0.22.04.1 [184 kB]\nGet:15 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libldap-common all 2.5.12+dfsg-0ubuntu0.22.04.1 [16.4 kB]\nGet:16 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-modules-5.15.0-40-generic amd64 5.15.0-40.43 [22.0 MB]\nGet:17 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-image-5.15.0-40-generic amd64 5.15.0-40.43 [10.9 MB]\nGet:18 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-modules-extra-5.15.0-40-generic amd64 5.15.0-40.43 [62.2 MB]\nGet:19 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-generic amd64 5.15.0.40.42 [1700 B]\nGet:20 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-image-generic amd64 5.15.0.40.42 [2568 B]\nGet:21 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-5.15.0-40 all 5.15.0-40.43 [12.2 MB]\nGet:22 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-5.15.0-40-generic amd64 5.15.0-40.43 [2780 kB]\nGet:23 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-generic amd64 5.15.0.40.42 [2430 B]\nGet:24 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 software-properties-common all 0.99.22.2 [14.1 kB]\nGet:25 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-software-properties all 0.99.22.2 [28.8 kB]\nPreconfiguring packages ...\nFetched 119 MB in 49s (2441 kB/s)\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 112493 files and directories currently installed.)\r\nPreparing to unpack .../systemd-timesyncd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking systemd-timesyncd (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../libsystemd0_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libsystemd0:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nSetting up libsystemd0:amd64 (249.11-0ubuntu3.3) ...\r\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 112493 files and directories currently installed.)\r\nPreparing to unpack .../0-libnss-systemd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libnss-systemd:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../1-systemd-sysv_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking systemd-sysv (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../2-libpam-systemd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libpam-systemd:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../3-systemd_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking systemd (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../4-udev_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking udev (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nPreparing to unpack .../5-libudev1_249.11-0ubuntu3.3_amd64.deb ...\r\nUnpacking libudev1:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...\r\nSetting up libudev1:amd64 (249.11-0ubuntu3.3) ...\r\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 112493 files and directories currently installed.)\r\nPreparing to unpack .../00-libapparmor1_3.0.4-2ubuntu2.1_amd64.deb ...\r\nUnpacking libapparmor1:amd64 (3.0.4-2ubuntu2.1) over (3.0.4-2ubuntu2) ...\r\nPreparing to unpack .../01-ubuntu-advantage-tools_27.9~22.04.1_amd64.deb ...\r\nUnpacking ubuntu-advantage-tools (27.9~22.04.1) over (27.8~22.04.1) ...\r\nPreparing to unpack .../02-apparmor_3.0.4-2ubuntu2.1_amd64.deb ...\r\nUnpacking apparmor (3.0.4-2ubuntu2.1) over (3.0.4-2ubuntu2) ...\r\nPreparing to unpack .../03-ubuntu-release-upgrader-core_1%3a22.04.11_all.deb ...\r\nUnpacking ubuntu-release-upgrader-core (1:22.04.11) over (1:22.04.10) ...\r\nPreparing to unpack .../04-python3-distupgrade_1%3a22.04.11_all.deb ...\r\nUnpacking python3-distupgrade (1:22.04.11) over (1:22.04.10) ...\r\nPreparing to unpack .../05-libldap-2.5-0_2.5.12+dfsg-0ubuntu0.22.04.1_amd64.deb ...\r\nUnpacking libldap-2.5-0:amd64 (2.5.12+dfsg-0ubuntu0.22.04.1) over (2.5.11+dfsg-1~exp1ubuntu3.1) ...\r\nPreparing to unpack .../06-libldap-common_2.5.12+dfsg-0ubuntu0.22.04.1_all.deb ...\r\nUnpacking libldap-common (2.5.12+dfsg-0ubuntu0.22.04.1) over (2.5.11+dfsg-1~exp1ubuntu3.1) ...\r\nSelecting previously unselected package linux-modules-5.15.0-40-generic.\r\nPreparing to unpack .../07-linux-modules-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-modules-5.15.0-40-generic (5.15.0-40.43) ...\r\nSelecting previously unselected package linux-image-5.15.0-40-generic.\r\nPreparing to unpack .../08-linux-image-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-image-5.15.0-40-generic (5.15.0-40.43) ...\r\nSelecting previously unselected package linux-modules-extra-5.15.0-40-generic.\r\nPreparing to unpack .../09-linux-modules-extra-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-modules-extra-5.15.0-40-generic (5.15.0-40.43) ...\r\nPreparing to unpack .../10-linux-generic_5.15.0.40.42_amd64.deb ...\r\nUnpacking linux-generic (5.15.0.40.42) over (5.15.0.39.40) ...\r\nPreparing to unpack .../11-linux-image-generic_5.15.0.40.42_amd64.deb ...\r\nUnpacking linux-image-generic (5.15.0.40.42) over (5.15.0.39.40) ...\r\nSelecting previously unselected package linux-headers-5.15.0-40.\r\nPreparing to unpack .../12-linux-headers-5.15.0-40_5.15.0-40.43_all.deb ...\r\nUnpacking linux-headers-5.15.0-40 (5.15.0-40.43) ...\r\nSelecting previously unselected package linux-headers-5.15.0-40-generic.\r\nPreparing to unpack .../13-linux-headers-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...\r\nUnpacking linux-headers-5.15.0-40-generic (5.15.0-40.43) ...\r\nPreparing to unpack .../14-linux-headers-generic_5.15.0.40.42_amd64.deb ...\r\nUnpacking linux-headers-generic (5.15.0.40.42) over (5.15.0.39.40) ...\r\nPreparing to unpack .../15-software-properties-common_0.99.22.2_all.deb ...\r\nUnpacking software-properties-common (0.99.22.2) over (0.99.22.1) ...\r\nPreparing to unpack .../16-python3-software-properties_0.99.22.2_all.deb ...\r\nUnpacking python3-software-properties (0.99.22.2) over (0.99.22.1) ...\r\nSetting up libapparmor1:amd64 (3.0.4-2ubuntu2.1) ...\r\nSetting up systemd (249.11-0ubuntu3.3) ...\r\nSetting up python3-distupgrade (1:22.04.11) ...\r\nSetting up libldap-common (2.5.12+dfsg-0ubuntu0.22.04.1) ...\r\nSetting up libldap-2.5-0:amd64 (2.5.12+dfsg-0ubuntu0.22.04.1) ...\r\nSetting up apparmor (3.0.4-2ubuntu2.1) ...\r\nInstalling new version of config file /etc/apparmor.d/abstractions/exo-open ...\r\nReloading AppArmor profiles \r\nSkipping profile in /etc/apparmor.d/disable: usr.sbin.rsyslogd\r\nSetting up python3-software-properties (0.99.22.2) ...\r\nSetting up systemd-timesyncd (249.11-0ubuntu3.3) ...\r\nSetting up udev (249.11-0ubuntu3.3) ...\r\nSetting up ubuntu-release-upgrader-core (1:22.04.11) ...\r\nSetting up linux-headers-5.15.0-40 (5.15.0-40.43) ...\r\nSetting up ubuntu-advantage-tools (27.9~22.04.1) ...\r\nInstalling new version of config file /etc/ubuntu-advantage/uaclient.conf ...\r\nCreated symlink /etc/systemd/system/multi-user.target.wants/ubuntu-advantage.service -> /lib/systemd/system/ubuntu-advantage.service.\r\nSetting up linux-headers-5.15.0-40-generic (5.15.0-40.43) ...\r\nSetting up systemd-sysv (249.11-0ubuntu3.3) ...\r\nSetting up libnss-systemd:amd64 (249.11-0ubuntu3.3) ...\r\nSetting up linux-headers-generic (5.15.0.40.42) ...\r\nSetting up software-properties-common (0.99.22.2) ...\r\nSetting up libpam-systemd:amd64 (249.11-0ubuntu3.3) ...\r\nSetting up linux-image-5.15.0-40-generic (5.15.0-40.43) ...\r\nI: /boot/vmlinuz.old is now a symlink to vmlinuz-5.15.0-39-generic\r\nI: /boot/initrd.img.old is now a symlink to initrd.img-5.15.0-39-generic\r\nI: /boot/vmlinuz is now a symlink to vmlinuz-5.15.0-40-generic\r\nI: /boot/initrd.img is now a symlink to initrd.img-5.15.0-40-generic\r\nSetting up linux-modules-extra-5.15.0-40-generic (5.15.0-40.43) ...\r\nSetting up linux-image-generic (5.15.0.40.42) ...\r\nSetting up linux-modules-5.15.0-40-generic (5.15.0-40.43) ...\r\nSetting up linux-generic (5.15.0.40.42) ...\r\nProcessing triggers for libc-bin (2.35-0ubuntu3) ...\r\nProcessing triggers for man-db (2.10.2-1) ...\r\nProcessing triggers for dbus (1.12.20-2ubuntu4) ...\r\nProcessing triggers for initramfs-tools (0.140ubuntu13) ...\r\nupdate-initramfs: Generating /boot/initrd.img-5.15.0-39-generic\r\nProcessing triggers for linux-image-5.15.0-40-generic (5.15.0-40.43) ...\r\n/etc/kernel/postinst.d/initramfs-tools:\r\nupdate-initramfs: Generating /boot/initrd.img-5.15.0-40-generic\r\n/etc/kernel/postinst.d/zz-update-grub:\r\nSourcing file `/etc/default/grub'\r\nSourcing file `/etc/default/grub.d/init-select.cfg'\r\nGenerating grub configuration file ...\r\nFound linux image: /boot/vmlinuz-5.15.0-40-generic\r\nFound initrd image: /boot/initrd.img-5.15.0-40-generic\r\nFound linux image: /boot/vmlinuz-5.15.0-39-generic\r\nFound initrd image: /boot/initrd.img-5.15.0-39-generic\r\nFound linux image: /boot/vmlinuz-5.15.0-33-generic\r\nFound initrd image: /boot/initrd.img-5.15.0-33-generic\r\nWarning: os-prober will not be executed to detect other bootable partitions.\r\nSystems on them will not be added to the GRUB boot configuration.\r\nCheck GRUB_DISABLE_OS_PROBER documentation entry.\r\ndone\r\nNEEDRESTART-VER: 3.5\nNEEDRESTART-KCUR: 5.15.0-39-generic\nNEEDRESTART-KEXP: 5.15.0-40-generic\nNEEDRESTART-KSTA: 3\nNEEDRESTART-SVC: apache2.service\nNEEDRESTART-SVC: dbus.service\nNEEDRESTART-SVC: ModemManager.service\nNEEDRESTART-SVC: multipathd.service\nNEEDRESTART-SVC: networkd-dispatcher.service\nNEEDRESTART-SVC: packagekit.service\nNEEDRESTART-SVC: polkit.service\nNEEDRESTART-SVC: qemu-guest-agent.service\nNEEDRESTART-SVC: rsyslog.service\nNEEDRESTART-SVC: ssh.service\nNEEDRESTART-SVC: systemd-logind.service\nNEEDRESTART-SVC: udisks2.service\nNEEDRESTART-SVC: unattended-upgrades.service\nNEEDRESTART-SVC: user@1000.service\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "Calculating upgrade...",
        "The following NEW packages will be installed:",
        "  linux-headers-5.15.0-40 linux-headers-5.15.0-40-generic",
        "  linux-image-5.15.0-40-generic linux-modules-5.15.0-40-generic",
        "  linux-modules-extra-5.15.0-40-generic",
        "The following packages will be upgraded:",
        "  apparmor libapparmor1 libldap-2.5-0 libldap-common libnss-systemd",
        "  libpam-systemd libsystemd0 libudev1 linux-generic linux-headers-generic",
        "  linux-image-generic python3-distupgrade python3-software-properties",
        "  software-properties-common systemd systemd-sysv systemd-timesyncd",
        "  ubuntu-advantage-tools ubuntu-release-upgrader-core udev",
        "20 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.",
        "Need to get 119 MB of archives.",
        "After this operation, 561 MB of additional disk space will be used.",
        "Get:1 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd-timesyncd amd64 249.11-0ubuntu3.3 [31.2 kB]",
        "Get:2 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libsystemd0 amd64 249.11-0ubuntu3.3 [318 kB]",
        "Get:3 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libnss-systemd amd64 249.11-0ubuntu3.3 [133 kB]",
        "Get:4 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd-sysv amd64 249.11-0ubuntu3.3 [10.5 kB]",
        "Get:5 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libpam-systemd amd64 249.11-0ubuntu3.3 [203 kB]",
        "Get:6 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 systemd amd64 249.11-0ubuntu3.3 [4579 kB]",
        "Get:7 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 udev amd64 249.11-0ubuntu3.3 [1557 kB]",
        "Get:8 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libudev1 amd64 249.11-0ubuntu3.3 [77.4 kB]",
        "Get:9 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libapparmor1 amd64 3.0.4-2ubuntu2.1 [38.7 kB]",
        "Get:10 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 ubuntu-advantage-tools amd64 27.9~22.04.1 [849 kB]",
        "Get:11 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 apparmor amd64 3.0.4-2ubuntu2.1 [590 kB]",
        "Get:12 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 ubuntu-release-upgrader-core all 1:22.04.11 [26.2 kB]",
        "Get:13 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-distupgrade all 1:22.04.11 [106 kB]",
        "Get:14 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libldap-2.5-0 amd64 2.5.12+dfsg-0ubuntu0.22.04.1 [184 kB]",
        "Get:15 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libldap-common all 2.5.12+dfsg-0ubuntu0.22.04.1 [16.4 kB]",
        "Get:16 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-modules-5.15.0-40-generic amd64 5.15.0-40.43 [22.0 MB]",
        "Get:17 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-image-5.15.0-40-generic amd64 5.15.0-40.43 [10.9 MB]",
        "Get:18 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-modules-extra-5.15.0-40-generic amd64 5.15.0-40.43 [62.2 MB]",
        "Get:19 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-generic amd64 5.15.0.40.42 [1700 B]",
        "Get:20 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-image-generic amd64 5.15.0.40.42 [2568 B]",
        "Get:21 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-5.15.0-40 all 5.15.0-40.43 [12.2 MB]",
        "Get:22 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-5.15.0-40-generic amd64 5.15.0-40.43 [2780 kB]",
        "Get:23 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-headers-generic amd64 5.15.0.40.42 [2430 B]",
        "Get:24 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 software-properties-common all 0.99.22.2 [14.1 kB]",
        "Get:25 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-software-properties all 0.99.22.2 [28.8 kB]",
        "Preconfiguring packages ...",
        "Fetched 119 MB in 49s (2441 kB/s)",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 112493 files and directories currently installed.)",
        "Preparing to unpack .../systemd-timesyncd_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking systemd-timesyncd (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Preparing to unpack .../libsystemd0_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking libsystemd0:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Setting up libsystemd0:amd64 (249.11-0ubuntu3.3) ...",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 112493 files and directories currently installed.)",
        "Preparing to unpack .../0-libnss-systemd_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking libnss-systemd:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Preparing to unpack .../1-systemd-sysv_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking systemd-sysv (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Preparing to unpack .../2-libpam-systemd_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking libpam-systemd:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Preparing to unpack .../3-systemd_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking systemd (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Preparing to unpack .../4-udev_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking udev (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Preparing to unpack .../5-libudev1_249.11-0ubuntu3.3_amd64.deb ...",
        "Unpacking libudev1:amd64 (249.11-0ubuntu3.3) over (249.11-0ubuntu3.1) ...",
        "Setting up libudev1:amd64 (249.11-0ubuntu3.3) ...",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 112493 files and directories currently installed.)",
        "Preparing to unpack .../00-libapparmor1_3.0.4-2ubuntu2.1_amd64.deb ...",
        "Unpacking libapparmor1:amd64 (3.0.4-2ubuntu2.1) over (3.0.4-2ubuntu2) ...",
        "Preparing to unpack .../01-ubuntu-advantage-tools_27.9~22.04.1_amd64.deb ...",
        "Unpacking ubuntu-advantage-tools (27.9~22.04.1) over (27.8~22.04.1) ...",
        "Preparing to unpack .../02-apparmor_3.0.4-2ubuntu2.1_amd64.deb ...",
        "Unpacking apparmor (3.0.4-2ubuntu2.1) over (3.0.4-2ubuntu2) ...",
        "Preparing to unpack .../03-ubuntu-release-upgrader-core_1%3a22.04.11_all.deb ...",
        "Unpacking ubuntu-release-upgrader-core (1:22.04.11) over (1:22.04.10) ...",
        "Preparing to unpack .../04-python3-distupgrade_1%3a22.04.11_all.deb ...",
        "Unpacking python3-distupgrade (1:22.04.11) over (1:22.04.10) ...",
        "Preparing to unpack .../05-libldap-2.5-0_2.5.12+dfsg-0ubuntu0.22.04.1_amd64.deb ...",
        "Unpacking libldap-2.5-0:amd64 (2.5.12+dfsg-0ubuntu0.22.04.1) over (2.5.11+dfsg-1~exp1ubuntu3.1) ...",
        "Preparing to unpack .../06-libldap-common_2.5.12+dfsg-0ubuntu0.22.04.1_all.deb ...",
        "Unpacking libldap-common (2.5.12+dfsg-0ubuntu0.22.04.1) over (2.5.11+dfsg-1~exp1ubuntu3.1) ...",
        "Selecting previously unselected package linux-modules-5.15.0-40-generic.",
        "Preparing to unpack .../07-linux-modules-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...",
        "Unpacking linux-modules-5.15.0-40-generic (5.15.0-40.43) ...",
        "Selecting previously unselected package linux-image-5.15.0-40-generic.",
        "Preparing to unpack .../08-linux-image-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...",
        "Unpacking linux-image-5.15.0-40-generic (5.15.0-40.43) ...",
        "Selecting previously unselected package linux-modules-extra-5.15.0-40-generic.",
        "Preparing to unpack .../09-linux-modules-extra-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...",
        "Unpacking linux-modules-extra-5.15.0-40-generic (5.15.0-40.43) ...",
        "Preparing to unpack .../10-linux-generic_5.15.0.40.42_amd64.deb ...",
        "Unpacking linux-generic (5.15.0.40.42) over (5.15.0.39.40) ...",
        "Preparing to unpack .../11-linux-image-generic_5.15.0.40.42_amd64.deb ...",
        "Unpacking linux-image-generic (5.15.0.40.42) over (5.15.0.39.40) ...",
        "Selecting previously unselected package linux-headers-5.15.0-40.",
        "Preparing to unpack .../12-linux-headers-5.15.0-40_5.15.0-40.43_all.deb ...",
        "Unpacking linux-headers-5.15.0-40 (5.15.0-40.43) ...",
        "Selecting previously unselected package linux-headers-5.15.0-40-generic.",
        "Preparing to unpack .../13-linux-headers-5.15.0-40-generic_5.15.0-40.43_amd64.deb ...",
        "Unpacking linux-headers-5.15.0-40-generic (5.15.0-40.43) ...",
        "Preparing to unpack .../14-linux-headers-generic_5.15.0.40.42_amd64.deb ...",
        "Unpacking linux-headers-generic (5.15.0.40.42) over (5.15.0.39.40) ...",
        "Preparing to unpack .../15-software-properties-common_0.99.22.2_all.deb ...",
        "Unpacking software-properties-common (0.99.22.2) over (0.99.22.1) ...",
        "Preparing to unpack .../16-python3-software-properties_0.99.22.2_all.deb ...",
        "Unpacking python3-software-properties (0.99.22.2) over (0.99.22.1) ...",
        "Setting up libapparmor1:amd64 (3.0.4-2ubuntu2.1) ...",
        "Setting up systemd (249.11-0ubuntu3.3) ...",
        "Setting up python3-distupgrade (1:22.04.11) ...",
        "Setting up libldap-common (2.5.12+dfsg-0ubuntu0.22.04.1) ...",
        "Setting up libldap-2.5-0:amd64 (2.5.12+dfsg-0ubuntu0.22.04.1) ...",
        "Setting up apparmor (3.0.4-2ubuntu2.1) ...",
        "Installing new version of config file /etc/apparmor.d/abstractions/exo-open ...",
        "Reloading AppArmor profiles ",
        "Skipping profile in /etc/apparmor.d/disable: usr.sbin.rsyslogd",
        "Setting up python3-software-properties (0.99.22.2) ...",
        "Setting up systemd-timesyncd (249.11-0ubuntu3.3) ...",
        "Setting up udev (249.11-0ubuntu3.3) ...",
        "Setting up ubuntu-release-upgrader-core (1:22.04.11) ...",
        "Setting up linux-headers-5.15.0-40 (5.15.0-40.43) ...",
        "Setting up ubuntu-advantage-tools (27.9~22.04.1) ...",
        "Installing new version of config file /etc/ubuntu-advantage/uaclient.conf ...",
        "Created symlink /etc/systemd/system/multi-user.target.wants/ubuntu-advantage.service -> /lib/systemd/system/ubuntu-advantage.service.",
        "Setting up linux-headers-5.15.0-40-generic (5.15.0-40.43) ...",
        "Setting up systemd-sysv (249.11-0ubuntu3.3) ...",
        "Setting up libnss-systemd:amd64 (249.11-0ubuntu3.3) ...",
        "Setting up linux-headers-generic (5.15.0.40.42) ...",
        "Setting up software-properties-common (0.99.22.2) ...",
        "Setting up libpam-systemd:amd64 (249.11-0ubuntu3.3) ...",
        "Setting up linux-image-5.15.0-40-generic (5.15.0-40.43) ...",
        "I: /boot/vmlinuz.old is now a symlink to vmlinuz-5.15.0-39-generic",
        "I: /boot/initrd.img.old is now a symlink to initrd.img-5.15.0-39-generic",
        "I: /boot/vmlinuz is now a symlink to vmlinuz-5.15.0-40-generic",
        "I: /boot/initrd.img is now a symlink to initrd.img-5.15.0-40-generic",
        "Setting up linux-modules-extra-5.15.0-40-generic (5.15.0-40.43) ...",
        "Setting up linux-image-generic (5.15.0.40.42) ...",
        "Setting up linux-modules-5.15.0-40-generic (5.15.0-40.43) ...",
        "Setting up linux-generic (5.15.0.40.42) ...",
        "Processing triggers for libc-bin (2.35-0ubuntu3) ...",
        "Processing triggers for man-db (2.10.2-1) ...",
        "Processing triggers for dbus (1.12.20-2ubuntu4) ...",
        "Processing triggers for initramfs-tools (0.140ubuntu13) ...",
        "update-initramfs: Generating /boot/initrd.img-5.15.0-39-generic",
        "Processing triggers for linux-image-5.15.0-40-generic (5.15.0-40.43) ...",
        "/etc/kernel/postinst.d/initramfs-tools:",
        "update-initramfs: Generating /boot/initrd.img-5.15.0-40-generic",
        "/etc/kernel/postinst.d/zz-update-grub:",
        "Sourcing file `/etc/default/grub'",
        "Sourcing file `/etc/default/grub.d/init-select.cfg'",
        "Generating grub configuration file ...",
        "Found linux image: /boot/vmlinuz-5.15.0-40-generic",
        "Found initrd image: /boot/initrd.img-5.15.0-40-generic",
        "Found linux image: /boot/vmlinuz-5.15.0-39-generic",
        "Found initrd image: /boot/initrd.img-5.15.0-39-generic",
        "Found linux image: /boot/vmlinuz-5.15.0-33-generic",
        "Found initrd image: /boot/initrd.img-5.15.0-33-generic",
        "Warning: os-prober will not be executed to detect other bootable partitions.",
        "Systems on them will not be added to the GRUB boot configuration.",
        "Check GRUB_DISABLE_OS_PROBER documentation entry.",
        "done",
        "NEEDRESTART-VER: 3.5",
        "NEEDRESTART-KCUR: 5.15.0-39-generic",
        "NEEDRESTART-KEXP: 5.15.0-40-generic",
        "NEEDRESTART-KSTA: 3",
        "NEEDRESTART-SVC: apache2.service",
        "NEEDRESTART-SVC: dbus.service",
        "NEEDRESTART-SVC: ModemManager.service",
        "NEEDRESTART-SVC: multipathd.service",
        "NEEDRESTART-SVC: networkd-dispatcher.service",
        "NEEDRESTART-SVC: packagekit.service",
        "NEEDRESTART-SVC: polkit.service",
        "NEEDRESTART-SVC: qemu-guest-agent.service",
        "NEEDRESTART-SVC: rsyslog.service",
        "NEEDRESTART-SVC: ssh.service",
        "NEEDRESTART-SVC: systemd-logind.service",
        "NEEDRESTART-SVC: udisks2.service",
        "NEEDRESTART-SVC: unattended-upgrades.service",
        "NEEDRESTART-SVC: user@1000.service"
    ]
}

        [truncated ouput]
```

## rebooting all servers
`ansible all -m reboot --become --ask-become-pass`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible all -m reboot --become --ask-become-pass
BECOME password: 
10.0.0.106 | CHANGED => {
    "changed": true,
    "elapsed": 56,
    "rebooted": true
}
10.0.0.84 | CHANGED => {
    "changed": true,
    "elapsed": 56,
    "rebooted": true
}
10.0.0.121 | CHANGED => {
    "changed": true,
    "elapsed": 72,
    "rebooted": true
}
```