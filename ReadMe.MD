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
  
## 5. Spark installation
 
 Install spark on all three instances.
 
 **At Analytics1, Analytics2, Analytics3**
 
Add host name, `sudo vi /etc/host` 
   
    10.128.0.2 analytics1
    10.138.0.2 analytics2
    10.142.0.2 analytics3
    
Download spark
 
    cd ~/downloads
    wget http://www-us.apache.org/dist/spark/spark-2.3.0/spark-2.3.0-bin-hadoop2.7.tgz
    tar -xzf spark-2.3.0-bin-hadoop2.7.tgz
    sudo mkdir /opt/spark
    sudo mv spark-2.3.0-bin-hadoop2.7 /opt/spark/

Set spark environment variable, `vi ~/.bashrc`

    export SPARK_HOME=/opt/spark/spark-2.3.0-bin-hadoop2.7
    export PATH=$SPARK_HOME/bin:$PATH   

then run `source ~/.bashrc`

To verify spark installtion run `spark-shell`.

**At Analytics1**

Generate ssh key to access slave machines without having to provide password.

    ssh-keygen -t rsa
    
Copy generated ssh key to `Analytics2 & Analytics3`

    ssh-copy-id serendio@analytics2
    ssh-copy-id serendio@analytics3
    
OR

Download `id_rsa.pub` file from path `~/.ssh/id_rsa.pub` (Analytics 1 machine). Then upload `id_rsa.pub` to `analytics2 & analytics3` machine. 

Login to analytics2, analytics3 and run `cat id_rsa.pub >> .ssh/authorized_keys`



To verify the setup run below commands at `analytics1` machine

    ssh serendio@analytics2
    ssh serendio@analytics3
    
Edit spark configurations for cluster setup, go to spark configuration directory `cd $SPARK_HOME/conf`

Create `spark-env.sh` file, `sudo cp spark-env.sh.template spark-env.sh`. Then `sudo vi spark-env.sh`.

    export SPARK_MASTER_HOST = 'analytics1'
    
Create `slaves` file, `sudo cp slaves.template slaves`. Then `sudo vi slaves` & add below lines

    analytics1
    analytics2
    analytics3

Create `spark-defaults.conf` file, `sudo cp spark-defaults.conf.template spark-defaults.conf`. Then `sudo vi spark-defaults.conf` & add below lines

    spark.eventLog.enabled          true
    spark.eventLog.dir              file:/opt/spark-events
    spark.history.fs.logDirectory   file:/opt/spark-events

Create `/opt/spark-events` directory.

    sudo mkdir /opt/spark-events
    sudo chown -R serendio:serendio /opt/spark-events
        
Change owenership of spark directory

    sudo chown -R serendio:serendio /opt/spark/spark-2.3.0-bin-hadoop2.7
   
Add firewall rules in GCP

    4040 to 4050  - Spark context web UI
    8080   - spark master web ui port
    7077   - spark master port
    18080  - Spark history server
    8081   - Workers Web UI

Start spark cluster, `cd $SPARK_HOME/sbin`

    ./start-all.sh
    

To verify spark cluter setup, run `jps` at `analytics1` machine. It should show,

    Master
    Worker

At `analytics2, analytics3` machine run `jps`. It should show,

    Worker

Check spark WebUI - http://analytics1:50070. It should show all 3 machines running.


## 6. Hadoop installation

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
    
To start hadoop services, `cd $HADOOP_HOME/sbin`

Start Namenode, Datanode service

    ./start-dfs.sh
    
Start Resourcemanager, Nodemanager

    ./start-yarn.sh
    
 
 **Add firewall rules in GCP**
 
    tcp:9000
    tcp:5025
    tcp:5030
    tcp:50070
 
 
 To verify hadoop cluster set-up, check url http://analytics1:50070.
 
 
## 7. Kafka installation

Install kafka on all three machines.

**At Analytics1, Analytics2, Analytics3**

Download Kafka

    cd ~/downloads
    wget http://www-eu.apache.org/dist/kafka/1.1.0/kafka_2.11-1.1.0.tgz
    tar -xzf kafka_2.11-1.1.0.tgz
    sudo mkdir /opt/kafka
    sudo mv kafka_2.11-1.1.0 /opt/kafka
    
Set kafka environment variables,

    vi ~/.bashrc
    export KAFKA_HOME=/opt/kafka/kafka_2.11-1.1.0
    
then `source ~/.bashrc`

Add firewall rules in GCP

    2181 - zookeeper
    9092 - kafka broker
    
**At Analytics1 machine**
    
Go to Kafka installation path, `cd $KAFKA_HOME/config`

Edit zookeeper properties, `sudo vi zookeeper.properties`
   
    dataDir=/opt/kafka/zookeeper

Create `/opt/kafka/zookeeper` directory

    sudo mkdir /opt/kafka/zookeeper
    
Edit kafka broker properties, `sudo vi server.properties`

    broker.id=0
    delete.topic.enable = true
    listeners=PLAINTEXT://:9092
    advertised.listeners=PLAINTEXT://analytics1:9092
    log.dirs=/opt/kafka/kafka-logs
    zookeeper.connect=analytics1:2181
    
Create `/opt/kafka/kafka-logs` directory
 
    sudo mkdir /opt/kafka/kafka-logs
			
Change owenership of kafka directory

    sudo chown -R serendio:serendio /opt/kafka
    
**At Analytics2 machine**
    
Go to Kafka installation path, `cd $KAFKA_HOME/config`
    
Edit kafka broker properties, `sudo vi server.properties`

    broker.id=1
    delete.topic.enable = true
    listeners=PLAINTEXT://:9092
    advertised.listeners=PLAINTEXT://analytics2:9092
    log.dirs=/opt/kafka/kafka-logs
    zookeeper.connect=analytics1:2181
    
Create `/opt/kafka/kafka-logs` directory
 
    sudo mkdir /opt/kafka/kafka-logs
			
Change owenership of kafka directory

    sudo chown -R serendio:serendio /opt/kafka

**At Analytics3 machine**
    
Go to Kafka installation path, `cd $KAFKA_HOME/config`
    
Edit kafka broker properties, `sudo vi server.properties`

    broker.id=2
    delete.topic.enable = true
    listeners=PLAINTEXT://:9092
    advertised.listeners=PLAINTEXT://analytics3:9092
    log.dirs=/opt/kafka/kafka-logs
    zookeeper.connect=analytics1:2181
    
Create `/opt/kafka/kafka-logs` directory
 
    sudo mkdir /opt/kafka/kafka-logs
			
Change owenership of kafka directory

    sudo chown -R serendio:serendio /opt/kafka


## 8. ElasticSearch installation

Install ElasticSearch on all three machines.

Download ElasticSearch,

    cd ~/downloads
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.3.0.deb
 
 Install ElasticSearch
 
     sudo dpkg -i elasticsearch-6.3.0.deb
     sudo update-rc.d elasticsearch defaults 95 10
     
 Add firewall rules in GCP
 
     9200 - elasticsearch instance
     9001 -  elasticsearch monitor
     
  **At Analytics1 machine**
  
  Update configurations,
  
      sudo -i
      cd /etc/elasticsearch/
      
  Edit `elasticsearch.yml` file `vi elasticsearch.yml`
    
     cluster.name: geosatis
     node.name: analytics1
     network.host: ["analytics1","localhost"]
     discovery.zen.ping.unicast.hosts: ["analytics1", "analytics2", "analytics3"]
     xpack.security.enabled: false
     xpack.monitoring.enabled: true
     xpack.monitoring.collection.enabled: true
     xpack.monitoring.exporters:
        geosatis:
                type: http
                host: ["http://analytics1:9200"]

  
  Start elasticsearch serivce 
  
     sudo -i service elasticsearch start
     
     
 **At Analytics2 machine**
  
  Update configurations,
  
      sudo -i
      cd /etc/elasticsearch/
      
  Edit `elasticsearch.yml` file `vi elasticsearch.yml`
    
     cluster.name: geosatis
     node.name: analytics2
     network.host: ["analytics2","localhost"]
     discovery.zen.ping.unicast.hosts: ["analytics1", "analytics2", "analytics3"]
     xpack.security.enabled: false
     xpack.monitoring.enabled: true
     xpack.monitoring.collection.enabled: true
     xpack.monitoring.exporters:
        geosatis:
                type: http
                host: ["http://analytics2:9200"]
  
  Start elasticsearch serivce 
  
     sudo -i service elasticsearch start
     
     
 **At Analytics3 machine**
  
  Update configurations,
  
      sudo -i
      cd /etc/elasticsearch/
      
  Edit `elasticsearch.yml` file `vi elasticsearch.yml`
    
     cluster.name: geosatis
     node.name: analytics3
     network.host: ["analytics3","localhost"]
     discovery.zen.ping.unicast.hosts: ["analytics1", "analytics2", "analytics3"]
     xpack.security.enabled: false
     xpack.monitoring.enabled: true
     xpack.monitoring.collection.enabled: true
     xpack.monitoring.exporters:
        geosatis:
                type: http
                host: ["http://analytics3:9200"]
  
  Start elasticsearch serivce 
  
     sudo -i service elasticsearch start

Verify elasticsearch setup

    curl -XGET 'http://analytics1:9200/_cluster/state?pretty'


## 9. Cerebro installation [ElasticSearch Monitoring tool]

**At Analytics1**

Download and install cerebro monitoring tool.

    cd ~/downloads
    wget https://github.com/lmenezes/cerebro/releases/download/v0.7.3/cerebro-0.7.3.tgz
    tar -xzf cerebro-0.7.3.tgz
    sudo mkdir /opt/cerebro
    sudo mv cerebro-0.7.3 /opt/cerebro/
    
start cerebro,
   
    cd /opt/cerebro/cerebro-0.7.3
    bin/cerebro -Dhttp.port=9001 -Dhttp.address=0.0.0.0
    
Check cerebro in url -  http://analytics1:9001


## 10. Kibana installation

Install Kibana on `Analytics 3` machine.

    wget https://artifacts.elastic.co/downloads/kibana/kibana-6.3.0-amd64.deb
    sudo dpkg -i kibana-6.3.0-amd64.deb
    sudo update-rc.d kibana defaults 95 10
    sudo -i service kibana start
 
Edit kibana cofigurations `sudo vi /etc/kibana/kibana.yml`

    server.host: "analytics3"
    server.name: "analytics3"
    elasticsearch.url: "http://analytics3:9200"
    xpack.security.enabled: false
    xpack.monitoring.enabled: true
    xpack.monitoring.elasticsearch.url: "http://analytics3:9200"
    xpack.monitoring.kibana.collection.enabled: true

    
## 11. Logstash installation

Install Logstash on `Analytics 3` machine.

     wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
     sudo apt-get install apt-transport-https
     echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
     sudo apt-get update && sudo apt-get install logstash
     sudo service logstash start
     

Edit Logstash cofigurations `sudo vi /etc/kibana/logstash.yml`

    xpack.monitoring.enabled: true
    xpack.monitoring.elasticsearch.url: "http://analytics3:9200"
    xpack.monitoring.elasticsearch.username:
    xpack.monitoring.elasticsearch.password:

Edit `pipelines.yml` file

```
- pipeline.id: directstream
  path.config: "/etc/logstash/conf.d/directstream.conf"
  pipeline.workers: 5
- pipeline.id: sparkoutputstream
  path.config: "/etc/logstash/conf.d/sparkoutputstream.conf"
  pipeline.workers: 5
```

Create `conf.d` folder then add logstash pipelines [directstream.conf & sparkoutputstream.conf] from git repository.
     

## 12. Metricbeat installation

Install metricbeat on all three instances.

    curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.3.0-amd64.deb
    sudo dpkg -i metricbeat-6.3.0-amd64.deb
    
Edit 'metricbeat.yml' file ,`cd /etc/metricbeat`

    setup.dashboards.enabled: true
    setup.kibana:
        host: "analytics3:5601"
    output.elasticsearch:
        hosts: ["analytics3:9200"]
    xpack.monitoring.enabled: true
    	
Edit kafka & System modules `cd /modules.d`
 
kafka modules - `sudo vi kafka.yml`

    - module: kafka
    hosts: ["analytics1:9092"]
  
system modules - `sudo vi system.yml`
  
   ```- module: system
  period: 1m
  metricsets:
    - filesystem
    - fsstat
    - cpu
    - core
    - load
    - memory
    - network
    - process
```

Start metricbeat,

    sudo service metricbeat start
    
    
