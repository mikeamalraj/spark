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

    curl -XPUT 'analytics1:9200/lookup?pretty' -H 'Content-Type: application/json' -d'
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

    curl -X PUT "localhost:9200/_template/unil_template" -H 'Content-Type: application/json' -d'
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

    cd Geosatis
    mvn clean package
    
note the path of output jar file (for future uses).    

##### Compile topic splitter

