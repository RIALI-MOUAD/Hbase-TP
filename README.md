# TP 1 : Installation et utilisation d'Apache Hbase

## Installation et configuration d’Apache Hbase en mode « Standalone » et en mode « Pseudo-distribué » :

### 1- Préparation de l’environnement :

#### Installation de "Apache Hadoop" :
![](https://hadoop.apache.org/hadoop-logo.jpg)

[![](https://img.shields.io/badge/version-3.2.1-green.svg)](https://archive.apache.org/dist/hadoop/core/hadoop-3.2.1/hadoop-3.2.1.tar.gz)
[![Generic badge](https://img.shields.io/badge/size-359.2MB-green.svg)](https://shields.io/)

##### Etape 1 : Création d'un utilisateur hduser :
```shell
mouad-kamal@mouadkamal-VirtualBox:~$ sudo adduser hduer
[sudo] password for mouad-kamal: 
Adding user `hduer' ...
Adding new group `hduer' (1001) ...
Adding new user `hduer' (1001) with group `hduer' ...
Creating home directory `/home/hduer' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hduer
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 


Is the information correct? [Y/n] y
```
D'abord on redémarre la machine virtuelle et on utilise le nouveau compte hduser.
##### Etape 2 : Mise en place de la clé ssh 

```shell
mouad-kamal@mouadkamal-VirtualBox:~$ sudo apt-get install openssh-server
Reading package lists... Done
Building dependency tree       
Reading state information... Done
openssh-server is already the newest version (1:7.6p1-4ubuntu0.3).

```
Maintenant, il faut mettre en place la clé ssh pour son propre compte. Pour cela, on exécute les commandes suivantes :

```sh
mouad-kamal@mouadkamal-VirtualBox:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/mouad-kamal/.ssh/id_rsa): 
Your identification has been saved in /home/mouad-kamal/.ssh/id_rsa.
Your public key has been saved in /home/mouad-kamal/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:e5oMGxv8gvuhCQ/m3G1JG0q7y88HdDTlGGGVH0jL8B4 mouad-kamal@mouadkamal-VirtualBox
The key's randomart image is:
+---[RSA 2048]----+
|        ==+o     |
|       .oBo..    |
|       ...E. .   |
|      . .. ..    |
|     . .S .      |
|    ..+  .       |
|  +. =*=. .      |
| + *+==X.+       |
|  o XBBo=        |
+----[SHA256]-----+
```
```sh
mouad-kamal@mouadkamal-VirtualBox:~$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
mouad-kamal@mouadkamal-VirtualBox:~$ chmod 0600 ~/.ssh/authorized_keys
```
```sh


```

