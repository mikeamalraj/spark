# Tools and Software installation


**General note**

1. Create downloads folder under home directory and put all your downloaded files in `/home/serendio/downloads`
1. Installation folder: `/opt/`
1. Add custom environment variables in `~/.bashrc` file

## 1. Java installation

Install the latest version of  Java on all three instances.

Download java,

    cd /home/serendio/downloads
    wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz
    
Extract tar file

     tar -xzf jdk-8u171-linux-x64.tar.gz
     
Move extracted java into opt foler

    sudo mkdir /opt/java
    sudo mv jdk1.8.0_171 /opt/java
    
Install java

    sudo update-alternatives --install /usr/bin/java java /opt/java/jdk1.8.0_171/bin/java 1
    sudo update-alternatives --install /usr/bin/javac javac /opt/java/jdk1.8.0_171/bin/javac 1
  
Set default java, if more than one java version installed.

    sudo update-alternatives --config java
    sudo update-alternatives --config javac

Set java environment variable. Open `~/.bashrc` file and add below lines

    export JAVA_HOME=/opt/java/jdk1.8.0_171
	export PATH=$JAVA_HOME/bin:$PATH

then run `source ~/.bashrc`.

To verify java installation, run `java -version`. 

    java version "1.8.0_171"
    Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
	Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

To verify java path, run `echo $JAVA_HOME`.

    /opt/java/jdk1.8.0_171
    

## 2. sbt installation

Install sbt on `analytics1` machine which is used to compile spark source code.

Download sbt,

    cd /home/serendio/downloads
    wget https://piccolo.link/sbt-0.13.17.tgz
    
Extract tar file

    tar -xzf sbt-0.13.17.tgz
    
Move extracted sbt to opt folder

    sudo mkdir /opt/sbt
    sudo mv sbt-0.13.17 /opt/sbt
    
Set environment variable in `~/.bashrc` file

    export SBT_HOME=/opt/sbt/sbt-0.13.17
    export PATH=$SBT_HOME/bin:$PATH
    
then run `source ~/.bashrc`


## 3. Maven installation

Install apache maven on `analytics1` machine which is used to compile java code (TestDataGenerator & TopicSplitter).

Download sbt,

    cd /home/serendio/downloads
    wget http://www-us.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
    
Extract tar file

    tar -xzf apache-maven-3.5.4-bin.tar.gz
    
Move extracted mvn to opt folder

    sudo mkdir /opt/maven
    sudo mv apache-maven-3.5.4 /opt/maven
    
Set environment variable in `~/.bashrc` file

    export MAVEN_HOME=/opt/maven/apache-maven-3.5.3
    export PATH=$MAVEN_HOME/bin:$PATH
    
then run `source ~/.bashrc`


## 4. Git installation

To install git run,

    sudo apt-get install git
    

## 5. Hadoop installation

Hadoop needs to be installed on all three instances.

**At Analytics1, Analytics2, Analytics3**

Download hadoop,
 
    cd ~/downloads
    wget http://www-us.apache.org/dist/hadoop/common/hadoop-2.9.1/hadoop-2.9.1.tar.gz
    tar -xzf hadoop-2.9.1.tar.gz
    sudo mkdir /opt/hadoop
    sudo mv hadoop-2.9.1 /opt/hadoop/

Set hadoop environment variables in `~/.bashrc` file

    export HADOOP_HOME=/opt/hadoop/hadoop-2.7.6
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    export HADOOP_MAPRED_HOME=$HADOOP_HOME
    export HADOOP_COMMON_HOME=$HADOOP_HOME
    export HADOOP_HDFS_HOME=$HADOOP_HOME
    export YARN_HOME=$HADOOP_HOME
    export PATH=$PATH:$HADOOP_HOME/bin
    export PATH=$PATH:$HADOOP_HOME/sbin
    
then run `source ~/.bashrc`

verify hadoop installation `hadoop version`

**At Analytics1 machine**

Go to hadoop configurations folder - `cd $HADOOP_HOME/etc/hadoop`

Add master host at master file, run `sudo vi master` then add below line

    analytics1

Add slave host at slaves file, run `sudo vi slaves` then add below line
   
    analytics1
    analytics2
    analytics3

Add below lines in `core-site.xml`, run `sudo vi core-site.xml`

```
    <property>
        <name>fs.defaultFS</name>
	<value>hdfs://analytics1:9000</value>
    </property>
```

Add below lines in `hdfs-site.xml`, run `sudo vi hdfs-site.xml`
 
 ```
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/opt/hadoop/namenode</value>
     </property>
     <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/opt/hadoop/datanode</value>
     </property>
 ```
 
Create directories to store information about namenode and datanode

    sudo mkdir /opt/hadoop/namenode
    sudo mkdir /opt/hadoop/datanode
    
Create `mapred-site.xml` file. 

    sudo mv mapred-site.xml.template mapred-site.xml
  
Add below lines in `mapred-site.xml`, run `sudo vi mapred-site.xml`

```
  <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>

```

Add below lines in `yarn-site.xml`, run `sudo vi yarn-site.xml`

```
   <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
   </property>
   <property>
      <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
       <name>yarn.resourcemanager.resource-tracker.address</name>
       <value>analytics1:8025</value>
    </property>
    <property>
       <name>yarn.resourcemanager.scheduler.address</name>
       <value>analytics1:8030</value>
     </property>
     <property>
        <name>yarn.resourcemanager.address</name>
        <value>analytics1:8050</value>
     </property>
```

**At Analytics2, Analytics3 machine**


Go to hadoop configuration folder - `cd $HADOOP_HOME/etc/hadoop`

copy `core-site.xml, hdfs-site.xml, mapred-site.xml, yarn-site.xml` from `analytics1`.

Create directory for datanode

    sudo mkdir /opt/hadoop/datanode
    
    
**At Analytics1, Analytics2, Analytics3**

Change ownership of hadoop folders,

    sudo chown -R serendio:serendio /opt/hadoop/hadoop-2.9.1
    sudo chown -R serendio:serendio datanode
    sudo chown -R serendio:serendio namenode


**At Analytics1**

Run below command to format Namenode,

    hadoop namenode -format
    
To hadoop services, `cd $HADOOP_HOME/sbin`

Start Namenode, Datanode service

    ./start-dfs.sh
    
Start Resourcemanager, Nodemanager

    ./start-yarn.sh
    
 
 **Add firewall rules in GCP**
 
    tcp:9000
    tcp:5025
    tcp:5030
    tcp:50070
 
 
 To verify hadoop cluster set-up, check url 'http://analytics1:50070'
 
 
 

    





