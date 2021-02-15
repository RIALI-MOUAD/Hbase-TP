# TP 1 : Installation et utilisation d'Apache Hbase

## Installation et configuration d’Apache Hbase en mode « Standalone » et en mode « Pseudo-distribué » :

### 1- Préparation de l’environnement :

#### Installation de "Apache Hadoop" :
![](https://hadoop.apache.org/hadoop-logo.jpg)

[![](https://img.shields.io/badge/version-3.2.1-green.svg)](https://archive.apache.org/dist/hadoop/core/hadoop-3.2.1/hadoop-3.2.1.tar.gz)
[![Generic badge](https://img.shields.io/badge/size-359.2MB-green.svg)](https://shields.io/)

##### Etape 1 : Création d'un utilisateur hduser :
```shell
mouad-kamal@mouadkamal-VirtualBox:~$ sudo adduser hduser
Adding user `hduser' ...
Adding new group `hduser' (1002) ...
Adding new user `hduser' (1002) with group `hduser' ...
Creating home directory `/home/hduser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
Sorry, passwords do not match
passwd: Authentication token manipulation error
passwd: password unchanged
Try again? [y/N] Y
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hduser
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] 
```
```sh
mouad-kamal@mouadkamal-VirtualBox:~$ sudo adduser hduser sudo
Adding user `hduser' to group `sudo' ...
Adding user hduser to group sudo
Done.
```
D'abord on redémarre la machine virtuelle et on utilise le nouveau compte hduser.

##### Etape 2 : Mise en place de la clé ssh

On installe le paquet nécessaire pour ssh en tapant la commande :

```sh 
hduser@mouadkamal-VirtualBox:~$ sudo apt-get install openssh-server
[sudo] password for hduser: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
openssh-server is already the newest version (1:7.6p1-4ubuntu0.3).
```

Maintenant, il faut mettre en place la clé ssh pour son propre compte. Pour cela,on exécute les commandes suivantes :
```sh
hduser@mouadkamal-VirtualBox:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hduser/.ssh/id_rsa): 
Created directory '/home/hduser/.ssh'.
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:69oKER0r+4CycvU+XmZjGZUN22+TxRGLVkXWBBf3m8I hduser@mouadkamal-VirtualBox
The key's randomart image is:
+---[RSA 2048]----+
|      .   .   .O%|
|     . o   *  o+*|
|    o o   + oo .+|
|   . +   .  o. oo|
|. . =   S    E=o |
| o . =   +   ... |
|o . . o O        |
|..   o.B .       |
|     .=+o        |
+----[SHA256]-----+
```

```sh
hduser@mouadkamal-VirtualBox:~$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
hduser@mouadkamal-VirtualBox:~$ chmod 0600 ~/.ssh/authorized_keys
```

On copie la clé public sur le serveur localhost :
```sh
hduser@mouadkamal-VirtualBox:~$ ssh-copy-id -i /home/hduser/.ssh/id_rsa.pub -f hduser@localhost
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/hduser/.ssh/id_rsa.pub"

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'hduser@localhost'"
and check to make sure that only the key(s) you wanted were added.
```
On teste la connexion à localhost :
```sh
hduser@mouadkamal-VirtualBox:~$ ssh hduser@localhost
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-28-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

359 packages can be updated.
292 updates are security updates.

New release '20.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Your Hardware Enablement Stack (HWE) is supported until April 2023.

```
```sh
hduser@mouadkamal-VirtualBox:~$ exit
logout
Connection to localhost closed.
```
