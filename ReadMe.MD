# Steps to run the application

### Start the services (as serendio user)

At analytics 1 machine start Hadoop, Spark, Zookeeper and Kafka services

    cd scripts
    ./start-hadoop.sh
    ./start-spark.sh
    ./start-zookeeper.sh    
    ./start-kafka.sh    
    
At analytics 3 machine start ElasticSearch, Kibana, Logstash and Kafka services

    cd scripts
    ./start-kafka.sh
    ./start-elastic.sh
    ./start-kibana.sh
    ./start-logstash.sh


### Download the source code and compile 

We need to run start spark applications from Analytics 1 machine (master machine). So, login into Analytics 1 machine, clone git repositroy

    git clone git@gitlab.serendio.com:Gary/Geosatis.git

then build the application using below command

    cd Geosatis
    sbt assembly
	

### Create Kafka topics

From analytics1 machine run below commands.

    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic RawStream --replication-factor 1 --partitions 5
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic DirectStream --replication-factor 1 --partitions 5
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic SparkInputStream --replication-factor 1 --partitions 15
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic SparkOutputStream --replication-factor 1 --partitions 15  
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic FilteredStream --replication-factor 1 --partitions 4
    


