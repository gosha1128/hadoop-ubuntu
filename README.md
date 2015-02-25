# hadoop-ubuntu

Getting hadoop up and running from scratch is not a trivial task.  Here's how I did it - and hopefully these instructions will help you out if you are doing something similar !

## Prerequisites

* Ubuntu Desktop 14.04

## Hadoop 2.4.0 or 2.6.0

### Basics

Download the 2.X.0 hadoop tar ball and unpack it

```
wget http://mirror.symnds.com/software/Apache/hadoop/common/hadoop-2.X.0/hadoop-2.X.0.tar.gz
gunzip hadoop-2.X.0.tar.gz
tar xvf hadoop-2.X.0.tar
```

Move the directory

```
sudo mv hadoop-2.X.0/ /usr/local
cd /usr/local
sudo ln -s hadoop-2.X.0/ hadoop
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

#### /usr/local/hadoop/etc/hadoop/mapred-site.xml

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
 
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

#### /usr/local/hadoop/etc/hadoop/hdfs-site.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
 
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
</configuration>
```

#### /usr/local/hadoop/etc/hadoop/yarn-site.xml

````
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>localhost:8025</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>localhost:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>localhost:8050</value>
    </property>
</configuration>
````
#### Test Configuration

````
/usr/local/hadoop/bin/hadoop namenode -format
````

### Start Services

As hduser
````
/usr/local/hadoop/sbin/start-dfs.sh
/usr/local/hadoop/sbin/start-yarn.sh
````

### Test WordCount

Download a large text file
````
wget http://www.gutenberg.org/ebooks/20417.txt.utf-8
````

Make an HDFS directory
````
/usr/local/hadoop/bin/hadoop fs -mkdir /test
````

Copy the text file
````
/usr/local/hadoop/bin/hadoop fs -copyFromLocal /tmp/gutenberg /test
````

Run the Map-Reduce job
````
/usr/local/hadoop/bin/hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop*examples*.jar wordcount /test/gutenberg /test/gutenberg-output
````

Retrieve output
````
/usr/local/hadoop/bin/hadoop fs -get /test/gutenberg-output
````

Note:  If you ran this sample already, you may need to remove a previous output directory:
````
/usr/local/hadoop/bin/hadoop fs -rmr /test/gutenberg-output
````


## Apache Spark 1.2.1

### Hadoop 2.4 build

Download hadooop 2.4 tar ball and unpack it
````
wget [URL]/spark-1.2.1-bin-hadoop2.4.tar.gz
tar zvxf spark-1.2.1-bin-hadoop2.4.tar.gz
````

### Spark Batch Example on HADOOP Yarn Cluster

Run builtin PI example on yarn cluster ( make sure hadoop hdfs and yarn daemons are started )
````
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn-cluster --num-executors 3 --driver-memory 100m --executor-memory 100m --executor-cores 1 lib/spark-examples*.jar 10
````

### Spark Shell Example ( Standalong )

Run a spark shell (standalone!) example:
````
./bin/spark-shell
scala> val textFile = sc.textFile("README.md")    // create a reference to the README.md file
scala> textFile.count                             // count the number of lines in the file
scala> textFile.first                             // print the first line
````

### Spark Shell Example on Cluster

Run a spark shell on a cluster example ( 4 workers ):
````
cp ./conf/spark-env.sh.template ./conf/spark-env.sh
echo "export SPARK_WORKER_INSTANCES=4" >> ./conf/spark-env.sh
````


