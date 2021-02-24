# TP 1 : Installation et utilisation d'Apache Hbase

## Installation et configuration d’Apache Hbase en mode « Standalone » et en mode « Pseudo-distribué » :

### 1- Préparation de l’environnement :

#### Installation de "Apache Hadoop" :
![](https://hadoop.apache.org/hadoop-logo.jpg)

[![](https://img.shields.io/badge/version-2.7.4-green.svg)](https://archive.apache.org/dist/hadoop/core/hadoop-2.7.4/hadoop-2.7.4.tar.gz)
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
##### Etape 4 : Installation d'Apache Hadoop 2.7.4
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ tar -zxvf hadoop-2.7.4.tar.gz
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ mv hadoop-2.7.4 hadoop
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mv hadoop /usr/local/hadoop/
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo chown -R hduser /usr/local/hadoop
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mkdir -p /usr/local/hadoop_store/hdfs/namenode
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mkdir -p /usr/local/hadoop_store/hdfs/datanode
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo chown -R hduser /usr/local/hadoop_store
```

##### Etape 5 : Configuration d'Apache Hadoop 2.7.4

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
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo chown -R hduser /app/hadoop/tmp
```
> On modifie d'abord des fichiers pour la configuration de Hadoop 
Dans le rep ***/usr/local/hadoop/etc/hadoop/*** :
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
STARTUP_MSG:   version = 2.7.4
STARTUP_MSG:   classpath = /usr/local/hadoop/etc/hadoop:/usr/local/hadoop/share/hadoop/common/lib/jetty-server-9.3.24.v20180605.jar:/usr/local/hadoop/share/hadoop/common/lib/commons-cli-1.2.jar:/usr/local/hadoop/share/hadoop/common/lib/kerby-util-1.0.1.jar:/usr/local/hadoop/share/hadoop/common/lib/commons-logging-1
...etc
```
Maintenant, il est temps de démarrer le cluster à nœud unique nouvellement installé.
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop/etc/hadoop$ start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hduser-namenode-mouadkamal-VirtualBox.out
localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-mouadkamal-VirtualBox.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hduser-secondarynamenode-mouadkamal-VirtualBox.out
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
Pour utiliser ces packages de Hadoop, on doit modifier SPARK_DIST_CLASSPATH afin d’inclure les fichiers jar relatifs à ces packages. Pour ce faire, il est préférable d'ajouter une entrée dans ***conf/spark-env.sh*** :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/spark$ cd conf
hduser@mouadkamal-VirtualBox:/usr/local/spark/conf$ cp spark-env.sh.template spark-env.sh
hduser@mouadkamal-VirtualBox:/usr/local/spark/conf$ sudo nano spark-env.sh
[sudo] password for hduser: 
Sorry, try again.
[sudo] password for hduser: 

```
On va insérer le contenu suivant dans spark-env.sh :
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
#### 3-Installation et configuration d’Apache Hbase :

![](https://hbase.apache.org/images/hbase_logo_with_orca_large.png)
[![](https://img.shields.io/badge/version-1.4.7-green.svg)](https://archive.apache.org/dist/hbase/1.4.7/hbase-1.4.7-bin.tar.gz)
[![Generic badge](https://img.shields.io/badge/size-118MB-green.svg)](https://shields.io/)

##### 1- Récupérer les fichiers sources de Hbase :
> Voire version Badge!!
##### 2- Décompresser le fichier récupéré dans le répertoire de votre choix :
on va repeter les memes etapes qu'on a deja fait avec **Apache Hadoop** et **Apache Spark** Alors on execute les commandes suivantes :
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ tar -zxvf hbase-1.4.7-bin.tar.gz
```
```sh
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ ls
hbase-1.4.7  hbase-1.4.7-bin.tar.gz  TP1_Hadoop.pdf  TP2.pdf
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ mv hbase-1.4.7 hbase
hduser@mouadkamal-VirtualBox:~/Desktop/BIG-DATA$ sudo mv hbase /usr/local/
[sudo] password for hduser: 
```
##### 3- Configurer le PATH dans le .bashrc :
on ajoute les lignes suivantes au fichier ***.bashrc*** afin de configurer le PATH vers ***Hbase*** repertoire :
```sh
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
```
d'ou on aura :
```sh 
  GNU nano 2.9.3                       .bashrc                       Modified  


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
#export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
#HADOOP VARIABLES END


export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin

export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin




^G Get Help    ^O Write Out   ^W Where Is    ^K Cut Text    ^J Justify
^X Exit        ^R Read File   ^\ Replace     ^U Uncut Text  ^T To Spell
```

Ensuite, pour maintenir les modifications et mettre a jour le fichier **.bashrc** on tape:
```sh 
hduser@mouadkamal-VirtualBox:~$ source .bashrc
```
##### 4- Configuration du fichier « hbase-env.sh » :
D'abord, il faut rappeler que ***Apache Hbase*** necessite **JAVA 8** - heureusement, on l'a deja installe-, alors il faut ajouter le chemin vers le ***JDK 8*** au fichier ***« hbase-env.sh »*** , d'ou vient l'utlite des commandes suivantes :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hbase$ ls conf
hadoop-metrics2-hbase.properties  hbase-policy.xml  regionservers
hbase-env.cmd                     hbase-site.xml
hbase-env.sh                      log4j.properties
```
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hbase$ sudo nano conf/hbase-env.sh
```
```sh
  GNU nano 2.9.3                  conf/hbase-env.sh                  Modified  


# The directory where pid files are stored. /tmp by default.
# export HBASE_PID_DIR=/var/hadoop/pids

# Seconds to sleep between slave commands.  Unset by default.  This
# can be useful in large clusters, where, e.g., slave rsyncs can
# otherwise arrive faster than the master can service them.
# export HBASE_SLAVE_SLEEP=0.1

# Tell HBase whether it should manage it's own instance of Zookeeper or not.
# export HBASE_MANAGES_ZK=true

# The default log rolling policy is RFA, where the log file is rolled as per t$
# RFA appender. Please refer to the log4j.properties file to see more details $
# In case one needs to do log rolling on a date change, one should set the env$
# HBASE_ROOT_LOGGER to "<DESIRED_LOG LEVEL>,DRFA".
# For example:
# HBASE_ROOT_LOGGER=INFO,DRFA
# The reason for changing default to RFA is to avoid the boundary case of fill$
# DRFA doesn't put any cap on the log size. Please refer to HBase-5655 for mor$
export JAVA_HOME=/opt/java/jdk1.8.0_71/





^G Get Help    ^O Write Out   ^W Where Is    ^K Cut Text    ^J Justify
^X Exit        ^R Read File   ^\ Replace     ^U Uncut Text  ^T To Linter
```

##### 5-Modification de /etc/hosts :
Puisque dans ce TP nous allons travaillé avec un seul serveur, il faut modifier le fichier /etc/hosts pour modifier l’adresse de notre serveur de 127.0.1.1 à l’adresse 127.0.0.1, comme suit :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hbase$ sudo nano /etc/hosts
```
```sh
  GNU nano 2.9.3                     /etc/hosts                      Modified  

127.0.0.1       localhost
127.0.0.1       mouadkamal-VirtualBox

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
Pour prendre les modifications en compte, on redémarre notre machine.

#### 3- Démarrage du cluster Hadoop configuré dans la machine "Hbase" :

on execute les commandes suivantes :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop$ cd ../hadoop_store
hduser@mouadkamal-VirtualBox:/usr/local/hadoop_store$ rm -rf *
hduser@mouadkamal-VirtualBox:/usr/local/hadoop_store$ mkdir -p /usr/local/hadoop_store/hdfs/namenode
hduser@mouadkamal-VirtualBox:/usr/local/hadoop_store$ mkdir -p /usr/local/hadoop_store/hdfs/datanode
hduser@mouadkamal-VirtualBox:/usr/local/hadoop_store$ chown -R hduser /usr/local/hadoop_store/hdfs/datanode
hduser@mouadkamal-VirtualBox:/usr/local/hadoop_store$ chown -R hduser /usr/local/hadoop_store/hdfs/namenode
```
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop_store$ cd ../hadoop
hduser@mouadkamal-VirtualBox:/usr/local/hadoop$ hdfs namenode -format
21/02/16 22:51:55 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = localhost/127.0.0.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.7.4
.
.
.
.
21/02/16 22:51:57 INFO namenode.FSImageFormatProtobuf: Image file /usr/local/hadoop_store/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 of size 323 bytes saved in 0 seconds.
21/02/16 22:51:57 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
21/02/16 22:51:57 INFO util.ExitUtil: Exiting with status 0
21/02/16 22:51:57 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at localhost/127.0.0.1
************************************************************/
```
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop$ start-all.sh
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
21/02/16 22:54:36 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hduser-namenode-mouadkamal-VirtualBox.out
localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-mouadkamal-VirtualBox.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hduser-secondarynamenode-mouadkamal-VirtualBox.out
21/02/16 22:54:59 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-hduser-resourcemanager-mouadkamal-VirtualBox.out
localhost: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hduser-nodemanager-mouadkamal-VirtualBox.out
```
et enfin, pour tester a quel point on a configure Hadoop, on execute :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hadoop$ hdfs dfsadmin -report
```
dans notre cas on a :
```sh
-------------------------------------------------
Live datanodes (1):

Name: 127.0.0.1:50010 (localhost)
Hostname: localhost
Decommission Status : Normal
Configured Capacity: 10499674112 (9.78 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 8140304384 (7.58 GB)
DFS Remaining: 1805803520 (1.68 GB)
DFS Used%: 0.00%
DFS Remaining%: 17.20%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Tue Feb 16 22:55:29 WET 2021
```
![](https://raw.githubusercontent.com/RIALI-MOUAD/Hbase-Media/main/Hdfs-Report.png?token=ALA3YFZY23OIFNMKXWQICJLAFRHWS)

##### 4- Configuration d’Apache Hbase pour un mode « Standalone » :
Apres s'assurer que Hadoop deamons marchent bien on poursuit les etapes suivants :
###### Configuration de « hbase-site.xml » pour un mode « Standalone » :
Pour installer HBase en mode Standalone, on ajoute les lignes suivantes dans le ***« hbase-site.xml »*** entre <configuration> et </configuration> :
```xml
 <property>
   <name>hbase.rootdir</name>
   <value>file:///home/mouad-kamal/hbase</value>
 </property>
 <property>
   <name>hbase.zookeeper.property.dataDir</name>
   <value>/home/mouad-kamal/zookeeper</value>
 </property>
 <property>
   <name>hbase.unsafe.stream.capability.enforce</name>
   <value>false</value>
 </property>
```
Alors le fichier ***hbase-site.xml*** devient :
```xml
  GNU nano 2.9.3                   hbase-site.xml                    Modified  

 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
-->
<configuration>
 <property>
   <name>hbase.rootdir</name>
   <value>file:///home/mouad-kamal/hbase</value>
 </property>
 <property>
   <name>hbase.zookeeper.property.dataDir</name>
   <value>/home/mouad-kamal/zookeeper</value>
 </property>
 <property>
   <name>hbase.unsafe.stream.capability.enforce</name>
   <value>false</value>
 </property>
           [ line 17/37 (45%), col 2/69 (2%), char 644/1248 (51%) ]
^G Get Help    ^O Write Out   ^W Where Is    ^K Cut Text    ^J Justify
^X Exit        ^R Read File   ^\ Replace     ^U Uncut Text  ^T To Spell
```
###### Lancement de Hbase pour un mode « Standalone » :
on lance la commande suivante :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hbase$ ./bin/start-hbase.sh
running master, logging to /usr/local/hbase/logs/hbase-hduser-master-mouadkamal-VirtualBox.out
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
```
> Avec :
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hbase$ jps
3521 ResourceManager
4785 HMaster
3682 NodeManager
2950 NameNode
5111 Jps
3115 DataNode
```
> Voila !! 
```sh
hduser@mouadkamal-VirtualBox:/usr/local/hbase$ hbase shell
```
```sh
HBase Shell
Use "help" to get list of supported commands.
Use "exit" to quit this interactive shell.
Version 1.4.7, r763f27f583cf8fd7ecf79fb6f3ef57f1615dbf9b, Tue Aug 28 14:40:11 PDT 2018

hbase(main):001:0> 

```

