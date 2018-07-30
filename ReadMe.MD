# Steps to run the application

### Start the services (as serendio user)

At analytics 1 machine start Hadoop, Spark, Zookeeper,Kafka and metricbeat services, 

    cd scripts
    ./start-hadoop.sh
    ./start-spark.sh
    ./start-zookeeper.sh    
    ./start-kafka.sh    
    ./start-metricbeat.sh
    
At analytics 1 machine start metricbeat
	
    cd scripts
    ./start-metricbeat.sh
    
At analytics 3 machine start ElasticSearch, Kibana, Logstash, Kafka and metricbeat services

    cd scripts
    ./start-kafka.sh
    ./start-elastic.sh
    ./start-kibana.sh
    ./start-logstash.sh
    ./start-metricbeat.sh


### Create Kafka topics

From analytics1 machine run below commands.

    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic RawStream --replication-factor 1 --partitions 5
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic DirectStream --replication-factor 1 --partitions 5
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic SparkInputStream --replication-factor 1 --partitions 15
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic SparkOutputStream --replication-factor 1 --partitions 15  
    $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper analytics1:2181 --topic FilteredStream --replication-factor 1 --partitions 4
    

### Create checkpoint directories at Hadoop

While running spark applications, the offset is maintained by spark by commiting it into a directory (checkpoint). Make sure it matches the name mentioned in the config.properties file.

    hadoop fs -mkdir -p /serendio/checkpoint
    hadoop fs -mkdir -p /serendio/checkpoint2
    

### Create Index template and lookup index in ElasticSearch

Create lookup index for geolocation caches.

    curl -XPUT 'analytics3:9200/lookup?pretty' -H 'Content-Type: application/json' -d'
    {
    "mappings": {
        "location": {
          "properties": {
            "request": {
              "type":  "keyword" 
            },
            "response": {
              "type":  "keyword" 
            }
          }
        }
      }
    }
    '

Create index template for unil- index

    curl -X PUT "analytics3:9200/_template/unil_template" -H 'Content-Type: application/json' -d'
     {
      "order":0,
      "version":1,
      "index_patterns":[
         "unil-*"
      ],
      "settings" : {
        "number_of_shards" : 1
	  },
      "mappings":{
         "doc":{
            "dynamic":"true",
            "numeric_detection":true,
            "dynamic_templates":[
               {
                  "payload":{
                     "path_match":"payload.*",
                     "path_unmatch":"payload.cells",
                     "mapping":{
                        "type":"long"
                     }
                  }
               }
            ],
            "properties":{
               "@timestamp":{
                  "type":"date"
               },
               "rawToHRTime":{
                  "type":"date"
               },
               "serverTime":{
                  "type":"date",
                  "format":"yyyy-MM-dd HH:mm:ss"
               },
               "generationTime":{
                  "type":"date",
                  "format":"yyyy-MM-dd HH:mm:ss",
                  "ignore_malformed":true
               },
               "rawToHRTimeNS":{
                  "type":"long"
               },
               "coordinate":{
                  "type":"geo_point"
               },
               "payload.servCell":{
                  "type":"object"
               },
               "payload.gpsMoo":{
                  "type":"object"
               },
               "payload.unregMoo":{
                  "type":"object"
               },
               "payload.beaconMoo":{
                  "type":"object"
               },
               "payload.devices":{
                  "type":"object"
               },
               "payload.dsSleep":{
                  "type":"object"
               },
               "payload.svinfo":{
                  "type":"object"
               },
               "flags":{
                  "type":"keyword"
               },
               "serial":{
                  "type":"keyword"
               },
               "type":{
                  "type":"keyword"
               },
               "unitId":{
                  "type":"keyword"
               },
               "satis_id":{
                  "type":"keyword"
               },
               "startTime":{
                  "type":"date"
               },
               "sparkStartTime":{
                  "type":"date"
               },
               "sparkEndTime":{
                  "type":"date"
               },
			   "sparkElapsedTime":{
                  "type":"long"
               },
			   "elapsedTime":{
                  "type":"long"
               }
            }
         }
      }
    }
    '

### Download the source code and compile 

##### Compile spark code

We need to run start spark applications from Analytics 1 machine (master machine). So, login into Analytics 1 machine, clone git repositroy

    git clone git@gitlab.serendio.com:Gary/Geosatis.git

then build the application using below command

    cd Geosatis
    sbt assembly
	
note the path of output jar file (for future uses).

##### Compile test data generator

Test data generator is used to simulate test messages based on the packet_dump.json file. To compile the test data generator (java) application,

    cd Geosatis/TestDataGenerator
    mvn clean package
    
note the path of output jar file (for future uses).    

##### Compile topic splitter

Topic splitter application is used to split raw messages (from RawStream) into two different stream (DirectStream & SparkInputStream). To compile topic splitter,

    cd Geosatis/TestDataGenerator
    mvn clean package
    
note the path of output jar file (for future uses). 


### Run the application (at Analytics 1)

##### Start StreamProcessor (Spark)

Start StreamProcessor application (on it's own terminal) which will read data from SparkInputStream and publish enriched data into SparkOutPutStream topic. 

To run the application,

    spark-submit --class "StreamProcessor" --master spark://analytics1:7077 /path/to/geostatis-assembly-0.1-SNAPSHOT.jar
   
##### Start LocationManager (Spark)

Start LocationManager application (on it's own terminal) which will read data from SparkInputStream and publish enriched data into SparkOutPutStream topic. 

To run the application,

    spark-submit --class "StreamProcessor" --master spark://analytics1:7077 /path/to/geostatis-assembly-0.1-SNAPSHOT.jar

##### Start TestDataGenerator (Java)

Start test data generator to simulate test messages from sample message file (packet_dump.json).

    java -jar /path/to/TestDataGenerator-0.0.1-SNAPSHOT-jar-with-dependencies.jar --timelimit 30 --braceletcount 10000 --file /path/to/packet_dump.json --topic RawStream

##### Start TopicSplitter (Java)

Start topic splitter java application using below command.

    java -jar /path/to/TopicSplitter-0.0.1-SNAPSHOT-jar-with-dependencies.jar
    

### Notes:

If you would like to test the application again, make sure to delete kafka topics, checkpoint directories and unil-index.

To delete kafka topics,

    $KAFKA_HOME/bin/kafka-topics.sh --delete --zookeeper analytics1:2181 --topic RawStream
    $KAFKA_HOME/bin/kafka-topics.sh --delete --zookeeper analytics1:2181 --topic DirectStream
    $KAFKA_HOME/bin/kafka-topics.sh --delete --zookeeper analytics1:2181 --topic SparkInputStream
    $KAFKA_HOME/bin/kafka-topics.sh --delete --zookeeper analytics1:2181 --topic SparkOutputStream
    $KAFKA_HOME/bin/kafka-topics.sh --delete --zookeeper analytics1:2181 --topic FilteredStream

then create the topics again as mentioned earlier.

Clear checkpoint directories,

    hadoop fs -rm -r /serendio/checkpoint/*
    hadoop fs -rm -r /serendio/checkpoint2/*
    

