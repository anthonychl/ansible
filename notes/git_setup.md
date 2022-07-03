# git setup

## check that git is installed on the ansible-control-host, if not install it.
```
anthony@ansible-control-host:~$ which git
/usr/bin/git
```
## create a github repository for the ansible project

## modify user settings
- go to settings / ssh & pgp keys
- add new ssh key, give it a name, 'anthony default' in my case same as the key we created to ssh into our servers
- copy the ssh key from /home/anthony/.ssh/id_ed25519.pub and paste it in the new github key
```
anthony@ansible-control-host:~$ cat .ssh/id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINvJqGUz9L7DjgTJNTo4niC3yjKo8UeLLS5rh1xCve7Z anthony default
```
## clone the project
- go back to your ansible repository
- click Code, select the SSH clone option and copy to clip board

## ansible-control-host terminal
```
anthony@ansible-control-host:~$ git clone git@github.com:anthonychl/ansible.git
Cloning into 'ansible'...
The authenticity of host 'github.com (140.82.112.3)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
remote: Enumerating objects: 16, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 16 (delta 2), reused 15 (delta 1), pack-reused 0
Receiving objects: 100% (16/16), done.
Resolving deltas: 100% (2/2), done.
```
### check the files have been copied
```
anthony@ansible-control-host:~$ ls ansible
notes  readme.md  scripts
```
## setting up git
```
anthony@ansible-control-host:~/ansible/scripts$ git config --global user.name "Anthony Chaple Lopez"

anthony@ansible-control-host:~/ansible/scripts$ git config --global user.email "anthonychl.dev@gmail.com"
```
## modify the scripts_readme.md file with nano
```
anthony@ansible-control-host:~/ansible/scripts$ nano scripts_readme.md 
```

## git status
```
anthony@ansible-control-host:~/ansible/scripts$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   scripts_readme.md

no changes added to commit (use "git add" and/or "git commit -a")
```
## git add
```
anthony@ansible-control-host:~/ansible/scripts$ git add scripts_readme.md 
```
file ready to be committed
```
anthony@ansible-control-host:~/ansible/scripts$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   scripts_readme.md
```
## git commit
```
anthony@ansible-control-host:~/ansible/scripts$ git commit -m "updated readme file"
[master 9da2c9e] updated readme file
 1 file changed, 1 insertion(+), 1 deletion(-)
```
## git push
```
anthony@ansible-control-host:~/ansible/scripts$ git push origin master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 349 bytes | 174.00 KiB/s, done.
Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com:anthonychl/ansible.git
   2f30aa4..9da2c9e  master -> master
```
