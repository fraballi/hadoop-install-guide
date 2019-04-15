# Hadoop Cluster Installation

> ## Step-by-step guide for Hadoop Single Node/Cluster deployment

## Single-Node Cluster (*Command Line*)

---

## Docker installation

```bash
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo docker run hello-world

```

## Docker-compose installation

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose --version

```

## Docker custom network

```bash
docker network create hadoop-cluster && \
docker run -it -h hadoop-namenode --name hadoop-namenode --net=host -p 8030:8030 -p 8032:8032 -p 8033:8033 -p 8042:8042 -p 8047:8047 -p 8088:8088 -p 8188:8188 -p 788:8788 -p 9000:9000 -p 9870:9870 -p 10033:10033 -p 19888:19888 -p 50030:50030 -p 50060:50060 -p 50070:50070 -p 50075:50075 -v /var/log/hadoop:/opt/hadoop/logs -v /var/local/hadoop:/root/shared debian:hadoop3
```

## Single-Node Cluster (*docker-compose.yml*)

```yaml
version: "3"
networks:
  hadoop-cluster:
    driver: bridge
services:
  hadoop:
    build: .
    image: debian:hadoop3
    entrypoint: sh /start-cluster.sh
    tty: true
    networks:
    - hadoop-cluster 
    # deploy:
    #   replicas: 3
    volumes:
      - /var/log/hadoop:/opt/hadoop/logs
      - /var/local/hadoop:/root/shared
    ports:
    - "8030:8030"
    - "8032:8032"
    - "8033:8033"
    - "8042:8042"
    - "8047:8047"
    - "8088:8088"
    - "8188:8188"
    - "8788:8788"
    - "9000:9000"
    - "9870:9870"
    - "10033:10033"
    - "19888:19888"
    - "50030:50030"
    - "50060:50060"
    - "50070:50070"
    - "50075:50075"
```

## Running **docker-compose** for *docker-compose.yml*

```bash

cd /path/to/file/docker-compose.yml
docker-compose up

``


### *Installing local tools*

```bash
apt install curl nano less wget ssh rsync telnet
```

## Environment variables setup / entrypoint (*/start-cluster.sh*)

```bash

export JAVA_HOME=/usr
export HADOOP_HOME=/opt/hadoop-3.2.0

export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME/bin

export HDFS_NAMENODE_USER="root"
export HDFS_DATANODE_USER="root"
export HDFS_SECONDARYNAMENODE_USER="root"
export YARN_RESOURCEMANAGER_USER="root"
export YARN_NODEMANAGER_USER="root"

export PATH=$PATH:$JAVA_HOME:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

service ssh start && start-all.sh

tail -f /dev/null

```

## 3-Nodes Cluster

---

### *Cluster Network*

```bash
docker network create --driver=bridge \
--subnet=192.168.0.0/24 \
--ip-range=192.168.0.0/24 \
--gateway=192.168.0.254 \
hadoop-cluster
```

### *Master Node*

```bash
docker run -it -h master --name hdp-master --cpus=1 \
--memory="1024MB" --memory-swap="2048MB" \
--net=hadoop-cluster --ip="192.168.0.1" \
--add-host="slave-1:192.168.0.2" \
--add-host="slave-2:192.168.0.3" \
debian:hadoop3
# bash "sh /root/start-cluster.sh"
```

### *Slave (1) Node*

```bash
docker run -it -h slave-1 --name hdp-slave-1 --cpus=1 \
--memory="1024MB" --memory-swap="2048MB" \
--workdir="/home/hadoop/" --net=hadoop-cluster --ip="192.168.0.2" \
--add-host="master:192.168.0.1" \
--add-host="slave-2:192.168.0.3" \
debian:hadoop3
# --entrypoint="sh /start-slave.sh" \
```

### *Slave (2) Node*

```bash
docker run -it -h slave-2 --name hdp-slave-2 --cpus=1 \
--memory="1024MB" --memory-swap="2048MB" \
--workdir="/home/hadoop/" --net=hadoop-cluster --ip="192.168.0.3" \
--add-host="master:192.168.0.1" \
--add-host="slave-1:192.168.0.2" \
debian:hadoop3
# --entrypoint="sh /start-slave.sh" \
```

---

## Default configuration (Slaves)

## File: **/etc/hadoop/workers**

### Add hosts values:
>
> e.g (Master Node)

```bash
master
```

> e.g (Slaves Nodes)

```bash
slave-1
slave-2
```

---

## File: **$HADOOP_CONF_DIR/slaves**

> e.g (Slaves Nodes)

```bash
slave-1
slave-2
```

---

## [Install Java JDK 8](http://tipsonubuntu.com/2016/07/31/install-oracle-java-8-9-ubuntu-16-04-linux-mint-18/)

```bash
sudo add-apt-repository ppa:webupd8team/java

sudo apt update; sudo apt install oracle-java8-installer

javac -version

sudo apt install oracle-java8-set-default
```

---

## File: **$HADOOP_CONF_DIR/hadoop-env.sh**

```bash
export JAVA_HOME=/usr
```

---

## File: **$HADOOP_CONF_DIR/core-site.xml**

```xml
 <property>
  <name>fs.default.name</name>
  <value>hdfs://master:9000</value>
 </property>

```

---

## File: **$HADOOP_CONF_DIR/mapred-site.xml**

```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
 </property>
<property>
  <name>yarn.app.mapreduce.am.resource.mb</name>
  <value>512</value>
</property>
<property>
  <name>mapreduce.map.memory.mb</name>
  <value>256</value>
</property>
<property>
  <name>mapreduce.reduce.memory.mb</name>
  <value>256</value>
</property>

```

---

## File: **$HADOOP_CONF_DIR/yarn-site.xml**

```xml
<property>
  <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
 <name>yarn.acl.enable</name>
 <value>0</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>master</value>
</property>
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
<property>
  <name>yarn.nodemanager.resource.memory-mb</name>
  <value>1536</value>
</property>
<property>
  <name>yarn.scheduler.maximum-allocation-mb</name>
  <value>1536</value>
</property>
<property>
  <name>yarn.scheduler.minimum-allocation-mb</name>
  <value>128</value>
</property>
<property>
  <name>yarn.nodemanager.vmem-check-enabled</name>
  <value>false</value>
</property>

```

---

## File: **$HADOOP_CONF_DIR/hdfs-site.xml**

Add these properties for HDFS *namenode/datanode*:

> ### Exception (Master Node): **dfs.name.dir**=**file:**///home/hadoop/hdfs/namenode

```xml
<configuration>
 <property>
  <name>dfs.replication</name>
  <value>1</value>
 </property>
 <property>
  <name>dfs.permission</name>
  <value>false</value>
 </property>
 <property>
  <name>dfs.replication</name>
  <value>1</value>
 </property>
 <property>
  <name>dfs.name.dir</name>
  <value>master://home/hadoop/hdfs/namenode</value>
 </property>
 <property>
  <name>dfs.data.dir</name>
  <value>file:///home/hadoop/hdfs/datanode</value>
 </property>
</configuration>

```

---

## Docker Documentation

- [Docker: Installation on Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

- [Docker: Compose - Installation](https://docs.docker.com/compose/install/)

- [Docker: Compose - Getting Started](https://docs.docker.com/compose/gettingstarted/)

- [Docker: Specify custom networks](https://docs.docker.com/compose/networking/#specify-custom-networks)

## Apache Hadoop Documentation

[Apache Hadoop: Official Documentation](https://hadoop.apache.org/docs/r3.0.2/)

- [Apache Hadoop: Setting up a Single Node Cluster](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

- [Apache Hadoop: Hadoop Cluster Setup](https://hadoop.apache.org/docs/r3.2.0/hadoop-project-dist/hadoop-common/ClusterSetup.html)

- [How to install Hadoop on Ubuntu 18.04 Bionic Beaver Linux](https://linuxconfig.org/how-to-install-hadoop-on-ubuntu-18-04-bionic-beaver-linux)

- [Install Hadoop: Setting up a Single Node Hadoop Cluster](https://www.linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/)
