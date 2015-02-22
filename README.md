# hadoop-ubuntu

Getting hadoop up and running from scratch is not a trivial task.  Here's how I did it - and hopefully these instructions will help you out if you are doing something similar !

## Prerequisites

* Ubuntu Desktop 14.04

## Hadoop 2.6.0

### Basics

Download the 2.6.0 hadoop tar ball and unpack it

```
wget http://mirror.symnds.com/software/Apache/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
gunzip hadoop-2.6.0.tar.gz
tar xvf hadoop-2.6.0.tar
```

Move the directory

```
sudo mv hadoop-2.6.0/ /usr/local
cd /usr/local
sudo ln -s hadoop-2.6.0/ hadoop
```

Setup user

```
sudo addgroup hadoop
sudo adduser --ingroup hadoop hduser
sudo adduser hduser sudo
sudo chown -R hduser:hadoop /usr/local/hadoop/
```

Setup SSH

```
su - hduser
sudo apt-get install ssh
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

Disable IPV6 in /etc/sysctl.conf 
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
Restart networking via
```
sudo service networking restart 
```
Install java
```
sudo apt-get update
sudo apt-get install default-jdk
```
(Login as hduser if you aren't.)  Add to ~/.bashrc
```
export HADOOP_HOME=/usr/local/hadoop
export JAVA_HOME=$(readlink -f /usr/bin/javac | sed "s:/bin/javac::")
```
Don't forget to source ~/.bashrc

### HDFS

Create a local directory for HDFS
```
su - hduser  # as needed
mkdir /usr/local/hadoop/data
```

### Configuration

#### /usr/local/hadoop/etc/hadoop/hadoop-env.sh

edit /usr/local/hadoop/etc/hadoop/hadoop-env.sh add add/change lines as follows:

```
export JAVA_HOME=[CHANGE TO JAVA_HOME SET ABOVE]
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true -Djava.library.path=$HADOOP_PREFIX/lib"
export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_PREFIX}/lib/native
```

####  /usr/local/hadoop/etc/hadoop/yarn-env.sh

Add the following:

```
export HADOOP_CONF_LIB_NATIVE_DIR=${HADOOP_PREFIX:-"/lib/native"}
export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"
```

#### /usr/local/hadoop/etc/hadoop/core-site.xml

Change whole file so it looks like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
</property>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/usr/local/hadoop/data</value>
</property>
</configuration>
```
