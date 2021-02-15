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

##### Etape 3 : Installation de JAVA 8
![](https://www.racam.fr/wp-content/uploads/2018/06/Screenshot_2018-07-20-java-8-Recherche-Google.png)

[![](https://img.shields.io/badge/version-1.8.0-green.svg)](https://docs.datastax.com/en/jdk-install/doc/jdk-install/installOracleJdkDeb.html)

On installe ***JAVA 8*** dans le répertoire **/opt/java** :
```sh
hduser@mouadkamal-VirtualBox:~$ sudo mkdir /opt/java
[sudo] password for hduser: 
hduser@mouadkamal-VirtualBox:~$ ls -R /opt
/opt:
java

/opt/java:
```
On extrait l'archive en utilisant la commande tar comme indiqué ci-dessous :
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ tar -zxvf jdk-8u71-linux-x64.tar.gz
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mv jdk1.8.0_71/ /opt/java/
```
On utilise la commande update-alternatives pour dire au système où java et ses exécutables sont installés.
```sh
mouad-kamal@mouadkamal-VirtualBox:/home/hduser$ sudo update-alternatives --install /usr/bin/java java /opt/java/jdk1.8.0_71/bin/java 100
update-alternatives: using /opt/java/jdk1.8.0_71/bin/java to provide /usr/bin/java (java) in auto mode
```
```sh
mouad-kamal@mouadkamal-VirtualBox:/home/hduser$ update-alternatives --config java
There is only one alternative in link group java (providing /usr/bin/java): /opt/java/jdk1.8.0_71/bin/java
Nothing to configure.
```

Maintenant,on met à jour aussi javac alternatives,et on ouvre le fichier **/etc/profile** en ecrivant :
```sh
export JAVA_HOME=/opt/java/jdk1.8.0_71/
export JRE_HOME=/opt/java/jdk1.8.0._71/jre
export PATH=$PATH:/opt/java/jdk1.8.0_71/bin:/opt/java/jdk1.8.0_71/jre/bin
```
Après avoir enregistré le fichier profile,on exécute la commande source pour recharger le fichier (en tant que root et avec l'utilisateur hadoop) :
```sh
~$ source /etc/profile
~$ source .bashrc
```
##### Etape 4 : Installation d'Apache Hadoop 3.2.1
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ tar -zxvf hadoop-3.2.1.tar.gz
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ mv hadoop-3.2.1 hadoop
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mv hadoop /usr/local/hadoop/
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo chown -R hduser /usr/local/hadoop
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mkdir -p /usr/local/hadoop_store/hdfs/namenode
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mkdir -p /usr/local/hadoop_store/hdfs/datanode
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo chown -R hduser /usr/local/hadoop_store
```

##### Etape 5 : Configuration d'Apache Hadoop 3.2.1

On modifie le fichier :**.bashrc** en ajoutant les lignes suivantes à la fin du fichier :
```sh
export JAVA_HOME=/opt/java/jdk1.8.0_71/
export JRE_HOME=/opt/java/jdk1.8.0._71/jre
export PATH=$PATH:/opt/java/jdk1.8.0_71/bin:/opt/java/jdk1.8.0_71/jre/bin
#HADOOP VARIABLES START
export JAVA_HOME=/opt/java/jdk1.8.0_71/
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
#export HADOOP_OPTS="­Djava.library.path=$HADOOP_INSTALL/lib"
#HADOOP VARIABLES END
```
Maintenant,on ouvre le fichier ***/usr/local/hadoop/etc/hadoop/hadoop-env.sh*** et on modifie la variable d'environnement
JAVA_HOME :
```sh
# The java implementation to use. By default, this environment
# variable is REQUIRED on ALL platforms except OS X!
export JAVA_HOME=/opt/java/jdk1.8.0_71/
# Location of Hadoop.  By default, Hadoop will attempt to determine
# this location based upon its execution path.
# export HADOOP_HOME=
```
Et On crée le répertoire des fichiers temporaires de hadoop :
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mkdir -p /app/hadoop/tmp
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo chown hduser /app/hadoop/tmp
```
> On modifie d'abord des fichiers pour la configuration de Hadoop 

On ouvre le fichier ***core-site.xml*** et entrez ce qui suit entre <configuration> et </ configuration> :
```xml
<configuration>
<property>
<name>hadoop.tmp.dir</name>
<value>/app/hadoop/tmp</value>
</property>
<property>
<name>fs.default.name</name>
<value>hdfs://localhost:54310</value>
</property>
</configuration>
```
le fichier ***hdfs-site.xml*** et entrez ce qui suit entre <configuration> et </ configuration> :

```xml
<configuration>
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
<property>
<name>dfs.namenode.name.dir</name>
<value>file:/usr/local/hadoop_store/hdfs/namenode</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>file:/usr/local/hadoop_store/hdfs/datanode</value>
</property>
</configuration>
```
le fichier ***mapred-site.xml*** et entrer ce qui suit entre <configuration> et </ configuration> :
```xml
<configuration>
<property>
<name>mapred.job.tracker</name>
<value>localhost:54311</value>
</property>
</configuration>
```
le fichier ***yarn-site.xml*** et entrer ce qui suit entre <configuration> et </ configuration> :
```xml
<configuration>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
</configuration>
```
