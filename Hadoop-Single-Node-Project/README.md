# Hadoop Single-Node Cluster
## About Apache Hadoop

Apache Hadoop is a set of algorithms (an open-source software framework written in Java) for distributed storage and distributed processing of very large data sets (Big Data) on computer clusters built from commodity hardware. All the modules in Hadoop are designed with a fundamental assumption that hardware failures (of individual machines, or racks of machines) are commonplace and thus should be automatically handled in software by the framework.

In this tutorial I will describe the required steps for setting up a pseudo-distributed, single-node Hadoop cluster backed by the Hadoop Distributed File System.

## Benefits of using Hadoop

The architecture of Hadoop allows you to scale your hardware as and when you need to. New nodes can be added incrementally without having to worry about the change in data formats or the handling of applications that sit on the file system.

One of the most important features of Hadoop is that it allows you to save enormous amounts of money by substituting cheap commodity servers for expensive ones. This is possible because Hadoop transfers the responsibility of fault tolerance from the hardware layer to the application layer.

## Environment Versions
```bash
Ubuntu 19.04 LTS Xenial Xerus 64-bit Server Edition
Hadoop 3.2.0
```
## Single-node Installation

The report here will describe the required steps for setting up a single-node Hadoop cluster backed by the Hadoop Distributed File System, running on Ubuntu Linux. Hadoop is a framework written in Java for running applications on large clusters of commodity hardware and incorporates features similar to those of the Google File System (GFS) and of the MapReduce computing paradigm. Hadoop’s HDFS is a highly fault-tolerant distributed file system and, like Hadoop in general, designed to be deployed on low-cost hardware. It provides high throughput access to application data and is suitable for applications that have large data sets.

Before we start, we will understand the meaning of the following:

## DataNode

A DataNode stores data in the Hadoop File System. A functional file system has more than one DataNode, with the data replicated across them.

## NameNode

The NameNode is the centrepiece of an HDFS file system. It keeps the directory of all files in the file system, and tracks where across the cluster the file data is kept. It does not store the data of these file itself.

## NodeManager

The NodeManager (NM) is YARN’s per-node agent, and takes care of the individual compute nodes in a Hadoop cluster. This includes keeping up-to date with the ResourceManager (RM), overseeing containers’ life-cycle management; monitoring resource usage (memory, CPU) of individual containers, tracking node-health, log’s management and auxiliary services which may be exploited by different YARN applications.

## ResourceManager

ResourceManager (RM) is the master that arbitrates all the available cluster resources and thus helps manage the distributed applications running on the YARN system. It works together with the per-node NodeManagers (NMs) and the per-application ApplicationMasters (AMs).

## Secondary Namenode

Secondary Namenode whole purpose is to have a checkpoint in HDFS. It is just a helper node for namenode.

## Install Java

Hadoop requires Java to be installed, so let’s begin by installing Java, update the source list:
```bash
petabytzuser@petabytz:~$ sudo apt-get update
```

The OpenJDK project is the default version of Java, that is provided from a supported Ubuntu repository.
```bash
petabytzuser@petabytz:~$ sudo apt-get install openjdk-8-jdk
```

These commands will update the package information on your server and then install Java. After executing these commands, execute the following command to verify that Java has been installed:
```bash
petabytzuser@petabytz:~$ java -version
```
If Java has been installed, this should display the version details as illustrated in the following output:
```bash
openjdk version "1.8.0_01-internal"
OpenJDK Runtime Environment (build 1.8.0_01-internal-b15)
OpenJDK 64-Bit Server VM (build 25.01-b15, mixed mode)
```
If you already have Java JDK installed on your system, then you need not run the above command.

## Adding a dedicated Hadoop system user
```bash
petabytzuser@petabytz:~$ sudo addgroup hadoop_petabytz_group
petabytzuser@petabytz:~$ sudo adduser --ingroup hadoop_petabytz_group petabytzuser1
```
This will add the user petabytzuser1 and the group hadoop_group to the local machine. Add petabytzuser1 to the sudo group:
```bash
petabytzuser@petabytz:~$ sudo adduser petabytzuser1 sudo
```
![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(31).png)

## Configuring SSH

The hadoop control scripts rely on SSH to peform cluster-wide operations. For example, there is a script for stopping and starting all the daemons in the clusters. To work seamlessly, SSh needs to be etup to allow password-less login for the hadoop user from machines in the cluster. The simplest ay to achive this is to generate a public/private key pair, and it will be shared across the cluster.

Hadoop requires SSH access to manage its nodes, i.e. remote machines plus your local machine. For our single-node setup of Hadoop, we therefore need to configure SSH access to localhost for the hadoop user user we created in the earlier.

We have to generate an SSH key for the petabytzuser1 user.
```bash
petabytzuser@petabytz:~$ su - petabytzuser1```
![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(32).png)
```bash
petabytzuser1@petabytz:~$ ssh-keygen -t rsa -P ''
```

-P '', here indicates an empty password

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(33).png)

You have to enable SSH access to your local machine with this newly created key which is done by the following command:
```bash
petabytzuser1@petabytz:~$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
```
![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(34).png)

The final step is to test the SSH setup by connecting to the local machine with the ```petabytzuser1``` user. The step is also needed to save your local machine’s host key fingerprint to the ```petabytzuser1``` user’s known hosts file.
```bash
petabytzuser1@petabytz:~$ ssh localhost
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(35).png)

If the SSH connection fails, we can try the following (optional):

* Enable debugging with ```ssh -vvv localhost``` and investigate the error in detail.
* Check the SSH server configuration in ```/etc/ssh/sshd_config```. If you made any changes to the SSH server configuration file, you can force a configuration reload with ```sudo /etc/init.d/ssh reload```.

## Main Installation

Start by switching to ```petabytzuser1```
```bash
petabytzuser@petabytz:~$ su - ```petabytzuser1```
```
Download and extract last version Hadoop binary code from [hadoop.apache.org](https://hadoop.apache.org/). I use current stable version Hadoop 3.2.0.
```bash
$ wget http://ftp.tc.edu.tw/pub/Apache/hadoop/common/hadoop-3.2.0/hadoop-3.2.0.tar.gz```
$ sudo tar -xvzf hadoop-*.tar.gz -C /usr/local/ && sudo mv /usr/local/hadoop-* /usr/local/hadoop
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(36).png)

## Setup Environment Variables for Hadoop

### Note : Go to root folder and type ```ls -a``` to find the ```.bashrc``` file

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(37).png)

Add the following entries to ```.bashrc``` file

Set Hadoop-related environment variables:
```bash
# Set Hadoop-related environment variables
export HADOOP_HOME=/usr/local/hadoop
# Add Hadoop bin/ directory to PATH
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(40).png)

Make change to take effect:
```bash
$ source ~/.bashrc
```

## Configuration

## hadoop-env.sh

Go to the hadoop-env.sh file and update changes given below.

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(38).png)

path : ```$HADOOP_HOME/etc/hadoop/hadoop-env.sh```
```bash
export JAVA_HOME=${JAVA_HOME}

export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(41).png)

Now we create the directory and set the required ownerships and permissions
```bash
petabytzuser1@petabytz:~$ sudo mkdir -p /app/hadoop/tmp
petabytzuser1@petabytz:~$ sudo chown petabytzuser1:hadoop_petabytz_group /app/hadoop/tmp
petabytzuser1@petabytz:~$ sudo chmod 750 /app/hadoop/tmp
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(39).png)

The last line gives read and write permissions to the /app/hadoop/tmp directory

Error: If you forget to set the required ownerships and permissions, you will see a java.io.IO Exception when you try to format the name node.

Go to the ```core-site.xml``` file and update changes given below.

path : ```$HADOOP_HOME/etc/hadoop/core-site.xml```

Paste the following between ```<configuration></configuration>```
```
<property>
    <name>hadoop.tmp.dir</name>
    <value>/app/hadoop/tmp</value>
    <description>A base for other temporary directories.</description>
</property>

<property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:54310</value>
    <description>The name of the default file system.  A URI whose
    scheme and authority determine the FileSystem implementation.  The
    uri's scheme determines the config property (fs.SCHEME.impl) naming
    the FileSystem implementation class.  The uri's authority is used to
    determine the host, port, etc. for a filesystem.</description>
</property>
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(42).png)

Go to the ```mapred-site.xml``` file and update changes given below.

Path : ```$HADOOP_HOME/etc/hadoop/mapred-site.xml```
```bash
<property>
    <name>mapred.job.tracker</name>
    <value>localhost:54311</value>
    <description>The host and port that the MapReduce job tracker runs
    at.  If "local", then jobs are run in-process as a single map
    and reduce task.
    </description>
</property>
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(43).png)

And In file ```$HADOOP_HOME/etc/hadoop/hdfs-site.xml```
```bash
<property>
    <name>dfs.replication</name>
    <value>1</value>
    <description>Default block replication.
    The actual number of replications can be specified when the file is created.
    The default is used if replication is not specified in create time.
    </description>
</property>
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(44).png)


### Formatting the HDFS filesystem via the NameNode

* Fsimage files which contains filesystem images which is basically the metadata file in your NameNode. And edit logs are the files which contains the recent changes in the file system, which is later merged in the fsimage.  
* Formatting the namenode deletes the information from namenode directory. The NameNode directory is specified in hdfs-site.xml file in dfs.namenode.name.dir property.
* Formatting the file system means initializing the directory specified by the dfs.name.dir variable.
Never format, up and running Hadoop filesystem. You will lose all your data stored in the HDFS.  

To format the filesystem (which simply initializes the directory specified by the dfs.name.dir variable). Run the command
```bash
petabytzuser1@petabytz:~$ hdfs namenode -format
```
![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(45).png)

### Starting single-node cluster

Run the command (start-dfs.sh & start-yarn.sh OR start-all.sh)
```bash
petabytzuser1@petabytz:~$ start-dfs.sh && start-yarn.sh
```
![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(46).png)

This will startup a Namenode, DataNode, ResourceManager and a NodeManager on the machine. Verify this by typing in the following command:
```bash
petabytzuser1@petabytz:~$ jps
9237 NameNode
10293 Jps
9783 ResourceManager
9592 SecondaryNameNode
9385 DataNode
9935 NodeManager
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(47).png)

If you can see a result similar to the depicted in the screenshot above, it means that you now have a functional instance of Hadoop running on your server. The output means that we now have a functional instance of Hadoop running on our server.

## Stop Hadoop

We run stop-dfs.sh && stop-yarn.sh (or stop-all.sh) to stop all the daemons running on our machine:
```bash
petabytzuser1@petabytz:~$ stop-dfs.sh && stop-yarn.sh
```
![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(48).png)

## Hadoop Web Interfaces

```http://localhost:9870``` – web UI of the NameNode daemon

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(50).png)

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(51).png)

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(52).png)

You can run ```http://localhost:8088``` for verfiying hadoop installation is successfully done or not. You can check find clusters and active nodes in thi UI.

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(49).png)

## Run the MapReduce wordcount example
1. Create the input directory.
```bash
hdfs dfs -mkdir -p /user/root/input
```

2. Copy a file from the local file system to the HDFS.
```bash
sudo hdfs dfs -copyFromLocal local-file /user/root/input
```

![](https://github.com/Petabytz/Hadoop-Projects/blob/master/Hadoop-Single-Node-Project/Screenshot%20(53).png)
