# hadoop-tutorial
Running Hadoop on Ubuntu Linux from https://www.digitalvidya.com/blog/install-hadoop-on-ubuntu-and-run-your-first-mapreduce-program/ with details on how to get running on AWS Free Tier

## Starting EC2
Go to Amazon EC2 and choose `Launch Instance`. Select the `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type` AMI and start this by selecting all of the free tier options and keeping all defaults.

If you do not already have a key pair to use from AWS, create a new one and download the pem file. This will be used to log into the server.

## Log into the EC2
Follow the instructions to connect to the EC2 instance which involves `chmod 400 your-pem-file.pem` and ssh'ing into the machine by running a command like:
`ssh -i your-pem-file.pem  ubuntu@your-public-dns-address`

## Create User
Run the following:
`sudo addgroup hadoop`
`sudo adduser hduser`
`sudo adduser hduser hadoop`
`sudo adduser hduser sudo`

## Prep Environment
Update the package list:
`sudo apt-get update`

Install Java:
`sudo apt-get install default-jdk`
Check that it is version >= 1.6 by running:
`java -version`

Install SSH:
`sudo apt-get install ssh`

Then, setup passwordless entry for localhost using SSH
`su hduser`
`ssh-keygen`
`sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

Then:
`chmod 0600 ~/.ssh/authorized_keys`

Check if it works by:
`ssh localhost`

## Install Hadoop
First, download Hadoop:
`wget wget https://www-us.apache.org/dist/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz`

The url in the tutorial is broken. You can find a URL on this page:
http://hadoop.apache.org/releases.html

Unzip it:
`tar xvzf hadoop-3.1.2.tar.gz`

## Configure Hadoop
Make a directory called `hadoop` and move the `hadoop-3.1.2` directory into it. Them,
`sudo mkdir -p /usr/local/hadoop`
`cd hadoop-3.1.2/`
`sudo mv * /usr/local/hadoop`
`sudo chown -R hduser:hadoop /usr/local/hadoop` # changing owner of the directory

## Setup Config Files:

### ~/.bashrc

Find the path where java is installed:
`readlink -f $(which java)`
JAVA_HOME is the path but does not include bin/java

Open .bashrc
`sudo vi ~/.bashrc`

Paste this and set JAVA_HOME to the path java is installed:
```#HADOOP VARIABLES START

export JAVA_HOME=<your java path>

export HADOOP_HOME=/usr/local/hadoop

export PATH=$PATH:$HADOOP_HOME/bin

export PATH=$PATH:$HADOOP_HOME/sbin

export HADOOP_MAPRED_HOME=$HADOOP_HOME

export HADOOP_COMMON_HOME=$HADOOP_HOME

export HADOOP_HDFS_HOME=$HADOOP_HOME

export YARN_HOME=$HADOOP_HOME

export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native

export HADOOP_OPTS=”-Djava.library.path=$HADOOP_HOME/lib”

#HADOOP VARIABLES END
```

Run `source ~/.bashrc` to apply changes

### hadoop-env.sh

Open this file with `sudo vi /usr/local/hadoop/etc/hadoop/hadoop-env.sh` and set the JAVA_HOME on line 54 to the path from before and save

### core-site.xml

Create directory and changeowner to group:
```$ sudo mkdir -p /app/hadoop/tmp
$ sudo chown hduser:hadoop /app/hadoop/tmp
```

Open the file: `$sudo vi /usr/local/hadoop/etc/hadoop/core-site.xml`

Append the following:
```<configuration>

<property>

 <name>hadoop.tmp.dir</name>

   <value>/app/hadoop/tmp</value>

   <description>A base for other temporary directories.</description>

</property>

<property>

  <name>fs.default.name</name>

   <value>hdfs://localhost:54310</value>

    <description>The name of the default file system.  A URI whose scheme and authority determine the FileSystem implementation.  The uri’s scheme determines the config property (fs.SCHEME.impl) naming the FileSystem implementation class.  The uri’s authority is used to determine the host, port, etc. for a filesystem.</description>

 </property>

</configuration>
```
and save.

### hdfs-site.xml

Make directories:
```
sudo mkdir -p /usr/local/hadoop_store/hdfs/namenode
sudo mkdir -p /usr/local/hadoop_store/hdfs/datanode
sudo chown -R hduser:hadoop /usr/local/hadoop_store
```

Open the file:
`sudo nano /usr/local/hadoop/etc/hadoop/hdfs-site.xml`

And paste in:
```<configuration>

 <property>

  <name>dfs.replication</name>

  <value>1</value>

  <description>Default block replication.The actual number of replications can be specified when the file is created. The default is used if replication is not specified in create time.

  </description>

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

### yarn-site.xml

Open the file: `sudo vi /usr/local/hadoop/etc/hadoop/yarn-site.xml`

Paste in:
``` <configuration>

   <property>

      <name>yarn.nodemanager.aux-services</name>

      <value>mapreduce_shuffle</value>

   </property>

</configuration>
```

## Format Hadoop File System

Run `hadoop namenode -format`


## Start Hadoop Cluster

Execute the scripts from Hadoop one-by-one

Go to this directory:
`cd $HADOOP_HOME/sbin/`

Run `./start-all.sh`


## Run Sample MapReduce Job

Go to `cd /usr/local/hadoop`

Run `hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.2.jar pi 8 20`

This should estimate the value of Pi to 3.15

