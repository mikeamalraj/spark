# Steps to run the application

### Start all services (as serendio user)

At analytics 1 machine start Hadoop, Spark, Zookeeper and Kafka services

    cd scripts
    ./start-hadoop.sh
    ./start-spark.sh
    ./start-zookeeper.sh    
    ./start-kafka.sh    
    
At analytics 3 machine start ElasticSearch, Kibana, Logstash and Kafka services

    cd scripts
    ./start-kafka.sh


### Download the source code

We need to run start spark applications from Analytics 1 machine (master machine). So, login into Analytics 1 machine, clone git repositroy

    git clone git@gitlab.serendio.com:Gary/Geosatis.git
	

 
