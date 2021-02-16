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

On ouvre le fichier ***core-site.xml*** et on entre ce qui suit entre <configuration> et </ configuration> :
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
le fichier ***hdfs-site.xml*** et on entre ce qui suit entre <configuration> et </ configuration> :

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
le fichier ***mapred-site.xml*** et on entre ce qui suit entre <configuration> et </ configuration> :
```xml
<configuration>
<property>
<name>mapred.job.tracker</name>
<value>localhost:54311</value>
</property>
</configuration>
```
le fichier ***yarn-site.xml*** et on entre ce qui suit entre <configuration> et </ configuration> :
```xml
<configuration>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
</configuration>
```
Et on formate le Namenode :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop/etc/hadoop$ hdfs namenode -format
WARNING: /usr/local/hadoop/logs does not exist. Creating.
2021-02-16 00:04:28,458 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = mouadkamal-VirtualBox/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 3.2.1
STARTUP_MSG:   classpath = /usr/local/hadoop/etc/hadoop:/usr/local/hadoop/share/hadoop/common/lib/jetty-server-9.3.24.v20180605.jar:/usr/local/hadoop/share/hadoop/common/lib/commons-cli-1.2.jar:/usr/local/hadoop/share/hadoop/common/lib/kerby-util-1.0.1.jar:/usr/local/hadoop/share/hadoop/common/lib/commons-logging-1
...etc
```
Maintenant, il est temps de démarrer le cluster à nœud unique nouvellement installé.
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop/etc/hadoop$ start-dfs.sh
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [mouadkamal-VirtualBox]
mouadkamal-VirtualBox: Warning: Permanently added 'mouadkamal-virtualbox' (ECDSA) to the list of known hosts.
2021-02-16 00:07:35,016 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop/etc/hadoop$ start-yarn.sh
Starting resourcemanager
Starting nodemanagers
```
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop/etc/hadoop$ jps
5345 DataNode
6005 NodeManager
5846 ResourceManager
5190 NameNode
5562 SecondaryNameNode
6381 Jps
```
> Voila !!

![](https://raw.githubusercontent.com/RIALI-MOUAD/Hbase-Media/main/Hadoop.png)
![](https://raw.githubusercontent.com/RIALI-MOUAD/Hbase-Media/main/Hadoop2.png)

#### Installation et configuration de spark en local :
![](https://spark.apache.org/images/spark-logo-trademark.png)
[![](https://img.shields.io/badge/version-2.4.3-green.svg)](https://archive.apache.org/dist/spark/spark-2.4.3/spark-2.4.3-bin-hadoop2.7.tgz)
[![Generic badge](https://img.shields.io/badge/size-230MB-green.svg)](https://shields.io/)

De même pour Apache Hadoop on va exécuter le code suivant :
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ tar -zxvf spark-2.4.3-bin-hadoop2.7.tgz
spark-2.4.3-bin-hadoop2.7/
spark-2.4.3-bin-hadoop2.7/python/
spark-2.4.3-bin-hadoop2.7/python/setup.cfg
spark-2.4.3-bin-hadoop2.7/python/pyspark/
spark-2.4.3-bin-hadoop2.7/python/pyspark/resultiterable.py
spark-2.4.3-bin-hadoop2.7/python/pyspark/python/
spark-2.4.3-bin-hadoop2.7/python/pyspark/python/pyspark/
...etc
```
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ mv spark-2.4.3-bin-hadoop2.7 spark
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mv spark /usr/local/
[sudo] password for hduser: 
```
Ajouter les lignes suivantes au fichier ***.bashrc*** :
```sh
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin
```
```sh
hduser@mouadkamal-VirtualBox:~$ source .bashrc
```
###### Installation Python :

```sh 
hduser@mouadkamal-VirtualBox:~$ sudo apt-get install python
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  fonts-liberation2 fonts-opensymbol gir1.2-gst-plugins-base-1.0
...etc
```
Apres, on aura le resultat suivant :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/spark$ ./bin/pyspark
Python 2.7.17 (default, Sep 30 2020, 13:38:04) 
[GCC 7.5.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
21/02/16 01:15:56 WARN Utils: Your hostname, mouadkamal-VirtualBox resolves to a loopback address: 127.0.1.1; using 10.0.2.15 instead (on interface enp0s3)
21/02/16 01:15:56 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
21/02/16 01:15:59 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.4.3
      /_/

Using Python version 2.7.17 (default, Sep 30 2020 13:38:04)
SparkSession available as 'spark'.
>>> 
```
et :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/spark$ ./bin/spark-shell
21/02/16 01:18:09 WARN Utils: Your hostname, mouadkamal-VirtualBox resolves to a loopback address: 127.0.1.1; using 10.0.2.15 instead (on interface enp0s3)
21/02/16 01:18:09 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
21/02/16 01:18:16 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://10.0.2.15:4040
Spark context available as 'sc' (master = local[*], app id = local-1613438315750).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.3
      /_/
         
Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_71)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
```
###### Connexion de Spark à une distribution de Hadoop :

```sh
  GNU nano 2.9.3                    spark-env.sh                     Modified  

# - SPARK_HISTORY_OPTS, to set config properties only for the history server ($
# - SPARK_SHUFFLE_OPTS, to set config properties only for the external shuffle$
# - SPARK_DAEMON_JAVA_OPTS, to set config properties for all daemons (e.g. "-D$
# - SPARK_DAEMON_CLASSPATH, to set the classpath for all daemons
# - SPARK_PUBLIC_DNS, to set the public dns name of the master or workers

# Generic options for the daemons used in the standalone deploy mode
# - SPARK_CONF_DIR      Alternate conf dir. (Default: ${SPARK_HOME}/conf)
# - SPARK_LOG_DIR       Where log files are stored.  (Default: ${SPARK_HOME}/l$
# - SPARK_PID_DIR       Where the pid file is stored. (Default: /tmp)
# - SPARK_IDENT_STRING  A string representing this instance of spark. (Default$
# - SPARK_NICENESS      The scheduling priority for daemons. (Default: 0)
# - SPARK_NO_DAEMONIZE  Run the proposed command in the foreground. It will no$
# Options for native BLAS, like Intel MKL, OpenBLAS, and so on.
# You might get better performance to enable these options if using native BLA$
# - MKL_NUM_THREADS=1        Disable multi-threading of Intel MKL
# - OPENBLAS_NUM_THREADS=1   Disable multi-threading of OpenBLAS
### in conf/spark-env.sh ###
# If 'hadoop' binary is on your PATH
export SPARK_DIST_CLASSPATH=/usr/local/hadoop
# With explicit path to 'hadoop' binary
export SPARK_DIST_CLASSPATH=/usr/local/hadoop/bin
# Passing a Hadoop configuration directory
export SPARK_DIST_CLASSPATH=/usr/local/hadoop/etc/hadoop


^G Get Help    ^O Write Out   ^W Where Is    ^K Cut Text    ^J Justify
^X Exit        ^R Read File   ^\ Replace     ^U Uncut Text  ^T To Linter

```
