# hadoop-ubuntu

Getting hadoop up and running from scratch is not a trivial task.  Here's how I did it - and hopefully these instructions will help you out if you are doing something similar !

## Prerequisites

* Ubuntu Desktop 14.04

## Hadoop 2.6.0

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
Add to ~/.bashrc
```
export HADOOP_HOME=/usr/local/hadoop
export JAVA_HOME=/usr
```
