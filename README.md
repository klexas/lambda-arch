# Lambda architecture

![Alt text](diagram.png?raw=true "Lambda architecture")

Read about the project [here](https://medium.com/@alexsandrosouza/lambda-architecture-how-to-build-a-big-data-pipeline-part-1-8b56075e83fe)


Our Lambda project receives real-time IoT Data Events coming from Connected Vehicles, 
then ingested to Spark through Kafka. Using the Spark streaming API, we processed and analysed 
IoT data events and transformed them into vehicle information.
While simultaneously the data is also stored into HDFS for Batch processing. 
We performed a series of stateless and stateful transformation using Spark streaming API on 
streams and persisted them to Cassandra database tables. In order to get accurate views, 
we also perform a batch processing and generating a batch view into Cassandra.
We developed responsive web traffic monitoring dashboard using Spring Boot, 
SockJs and Bootstrap which get the views from the Cassandra database and push to the UI using web socket.


All component parts are dynamically managed using Docker, which means you don't need to worry 
about setting up your local environment, the only thing you need is to have Docker installed.

System stack:
- Java 8
- Maven
- ZooKeeper
- Kafka
- Cassandra
- Spark
- Docker
- HDFS


The streaming part of the project was done from iot-traffic-project [InfoQ](https://www.infoq.com/articles/traffic-data-monitoring-iot-kafka-and-spark-streaming)

## How to use
*  Set the KAFKA_ADVERTISED_LISTENERS with your IP in the docker-compose.yml
* `docker-compose -p lambda up`
* `echo "localhost spark-master" >> /etc/hosts`
* `echo "localhost kafka" >> /etc/hosts`
* `echo "localhost namenode" >> /etc/hosts`
* `echo "localhost datanode" >> /etc/hosts`
* `echo "localhost cassandra" >> /etc/hosts`
*  Wait all services be up and running, then...
* `docker exec cassandra-iot cqlsh --username cassandra --password cassandra  -f /schema.cql`
* `docker exec kafka-iot kafka-topics --create --topic iot-data-event --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181` 
* `docker exec namenode hdfs dfs -mkdir /lambda-arch`
* `docker exec namenode hdfs dfs -chmod -R 777 /lambda-arch`

### Miscellaneous

### Spark
Add spark-master to /etc/hosts pointing to localhost

#### Submit a job to master
- mvn package
- `spark-submit --class com.apssouza.lambda.App --master spark://spark-master:7077 /Users/apssouza/Projetos/java/lambda-arch/target/lambda-arch-1.0-SNAPSHOT.jar``


#### GUI
http://localhost:8080 Master
http://localhost:8081 Slave


### HDFS

Comands https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/1/

Open a file - http://localhost:50070/webhdfs/v1/path/to/file/file.csv?op=open

Web file handle - https://hadoop.apache.org/docs/r1.0.4/webhdfs.html

#### Commands :
* `hdfs dfs -mkdir /user`
* `hdfs dfs -mkdir /user/lambda`
* `hdfs dfs -put localhost.csv /user/lambda/`
* Access the file http://localhost:50075/webhdfs/v1/user/lambda/localhost.csv?op=OPEN&namenoderpcaddress=namenode:8020&offset=0

#### Gui
http://localhost:50070
http://localhost:50075


### Kafka
* kafka-topics --create --topic iot-data-event --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
* kafka-console-producer --request-required-acks 1 --broker-list kafka:9092 --topic iot-data-event
* kafka-console-consumer --bootstrap-server kafka:9092 --topic iot-data-event
* kafka-topics --list --zookeeper zookeeper:2181


### Cassandra
- Log in `cqlsh --username cassandra --password cassandra`
- Access the keyspace `use TrafficKeySpace;`
- List data `SELECT * FROM TrafficKeySpace.Total_Traffic;`