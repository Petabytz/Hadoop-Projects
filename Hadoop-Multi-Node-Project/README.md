## Running Hadoop on Ubuntu Linux (Multi-Node Cluster)

From single-node clusters to a multi-node cluster

We will build a multi-node cluster merge three or more single-node clusters into one multi-node cluster in which one Ubuntu box will become the designated master, and the other box will become only a slave.

## Environment Versions

```bash
Ubuntu 16.04 LTS Xenial Xerus 64-bit Server Edition
Hadoop 2.7.2
```

## Prerequisites

Configuring single-node clusters first, here we have used three single node clusters. Shutdown each single-node cluster with the following command:
```bash
petabytzuser2@petabytzserver:~$ $HADOOP_HOME/sbin/stop-all.sh
```

## Networking

The easiest is to put three machines in the same network with regard to hardware and software configuration.

Update /etc/hosts on both machines. Put the alias to the ip addresses of all the machines. Here we are creating a cluster of 3 machines, one is master, one is datanode1 and other is datanode2:
```bash
petabytzuser2@petabytzserver:$ sudo vim /etc/hosts
```

Add the following lines for two node cluster
```
192.168.1.18    masternode  # IP address of the master node
192.168.1.20    datanode1  # IP address of the datanode1 node
192.168.1.21    datanode2  # IP address of the datanode2 node
```

## SSH access

The petabytzuser2 user on the master (ssh petabytzuser2@petabytzserver) must be able to connect:

* to its own user account on the master – i.e. ssh master in this context.
* to the hduser1 user account on the slave (i.e. ssh petabytzuser2@datanode1) via a password-less SSH login.

## Set up password-less SSH login between cluster:
```bash
petabytzuser2@petabytzserver:~$ ssh-copy-id -i $HOME/.ssh/id_rsa.pub petabytzuser2@datanode1
petabytzuser2@petabytzserver:~$ ssh-copy-id -i $HOME/.ssh/id_rsa.pub petabytzuser2@datanode2
```

Connect with user petabytzuser2 from the master to the user account hduser1 on the datanode1 and datanode2.
From master to master
```bash
petabytzuser2@petabytzserver:~$ ssh masternode
```

From master to datanode1
```bash
petabytzuser2@masternode:~$ ssh datanode1
```

From datanode1 to datanode2
```bash
petabytzuser2@datanode1:~$ ssh datanode2
```

## Hadoop

## Cluster Overview

This will describe how to configure one Ubuntu box as a master node and the other Ubuntu box as a slave node.

## Configuration

## $HADOOP_HOME/etc/hadoop/*-site.xml (All nodes.)

Open this file in the ```$HADOOP_HOME/etc/hadoop/``` directory:
```bash
$ sudo nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

GO to ```$HADOOP_HOME/etc/hadoop/core-site.xml (All nodes.)```

```bash
<property>
    <name>hadoop.tmp.dir</name>
    <value>/app/hadoop/tmp</value>
    <description>A base for other temporary directories.</description>
</property>

<property>
    <name>fs.default.name</name>
    <value>hdfs://master:54310</value>
    <description>The name of the default file system.  A URI whose
    scheme and authority determine the FileSystem implementation.  The
    uri's scheme determines the config property (fs.SCHEME.impl) naming
    the FileSystem implementation class.  The uri's authority is used to
    determine the host, port, etc. for a filesystem.</description>
</property>
```

## $HADOOP_HOME/etc/hadoop/mapred-site.xml (All nodes.)

Open this file in the ```$HADOOP_HOME/etc/hadoop/``` directory

```bash
$ sudo nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

Change the mapred.job.tracker parameter (in ```$HADOOP_HOME/etc/hadoop/mapred-site.xml```), which specifies the JobTracker (MapReduce master) host and port and add mapred.framework.name property.

## $HADOOP_HOME/etc/hadoop/mapred-site.xml (All nodes.)

```bash
<property>
    <name>mapred.job.tracker</name>
    <value>master:54311</value>
    <description>The host and port that the MapReduce job tracker runs
    at.  If "local", then jobs are run in-process as a single map
    and reduce task.
    </description>
</property>

<property>
    <name>mapred.framework.name</name>
    <value>yarn</value>
</property>
```

## $HADOOP_HOME/etc/hadoop/hdfs-site.xml (All nodes.)

Open this file in the ```$HADOOP_HOME/etc/hadoop/``` directory

```bash
$ sudo nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Change the dfs.replication parameter (in ```$HADOOP_HOME/etc/hadoop/hdfs-site.xml```) which specifies the default block replication. We have two nodes as slave available, so we set dfs.replication to 2. Changes to be like this:

```bash
<property>
    <name>dfs.replication</name>
    <value>2</value>
    <description>Default block replication.
        The actual number of replications can be specified when the file is created.
        The default is used if replication is not specified in create time.
        </description>
</property>
```

Paste the following between ```<configuration></configuration>``` in file ```$HADOOP_HOME/etc/hadoop/yarn-site.xml```:

```bash
<property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>masternode:8025</value>
</property>
<property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>masternode:8035</value>
</property>
<property>
    <name>yarn.resourcemanager.address</name>
    <value>masternode:8050</value>
</property>
```

## Applying Master node specific Hadoop configuration: (Only for master nodes)

These are some configuration to be applied over Hadoop master nodes (Since we have only one master node it will be applied to only one master node.)

Remove existing Hadoop data folder (which was created while single-cluster hadoop setup.)
```bash
petabytzuser2@masternode:~$ sudo rm -rf /app/hadoop/tmp
```

Make same (```/app/hadoop/tmp```) directory and create NameNode (```/usr/local/hadoop_tmp/hdfs/namenode```) directory:

```bash
petabytzuser2@masternode:~$ sudo mkdir -pv /app/hadoop/tmp/hdfs/namenode
```

Make petabytzuser2 as owner of that directory:
```bash
petabytzuser2@masternode:~$ sudo chown petabytzuser2:hadoop_petabyt_group -R /app/hadoop/tmp/
```

## Applying Slave node specific Hadoop configuration (Only for slave nodes)

Since we have three slave nodes, we will be applying the following changes over datanode1 and datanode2 nodes:

Remove existing Hadoop_data folder (which was created while single node hadoop setup)
```bash
$ sudo rm -rf /app/hadoop/tmp
```

Creates same (```/app/hadoop/tmp```) folder, an inside this folder again create DataNode (```/app/hadoop/tmp/hdfs/namenode```) directory:
```bash
$ sudo mkdir -pv /app/hadoop/tmp/hdfs/datanode
```

Make hduser as owner of that directory
```bash
$ sudo chown petabytzuser2:hadoop_petabytz_group -R /app/hadoop/tmp/
```

## Formatting the HDFS filesystem via the NameNode (Only for master nodes)

Format the cluster’s HDFS file system
```bash
$ hdfs namenode -format
```

## Starting the multi-node cluster (Only for master nodes)

```bash
petabytzuser2@masternode:~$ start-dfs.sh && start-yarn.sh
```

By this command the NameNode daemon is started on master, and DataNode daemons are started on all slaves (here: datanode1 and datanode2).

## Track/Monitor/Verify Hadoop cluster (Run on any Node)

Verify Hadoop daemons on Master, run the following commands
```bash
petabytzuser2@masternode:~$ jps
7104 Jps
6386 SecondaryNameNode
6555 ResourceManager
6158 NameNode
```

Verify Hadoop daemons on any slave (here: datanode1 and datanode2), DataNode and NodeManager should run:
```bash
$ jps
1344 DataNode
1653 Jps
1463 NodeManager
```

## Monitor Hadoop ResourseManage and Hadoop NameNode via web-version

ResourceManager: http://masternode:8088
