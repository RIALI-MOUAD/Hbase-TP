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
