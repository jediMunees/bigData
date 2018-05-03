# bigData
Learning bigData - Apache Kafka, SchemaRegistry, Avro schema, Elasticsearch

Learning BigData

1. Install and start Elasticsearch

https://www.elastic.co/downloads/elasticsearch

2. Start confluent kafka including zookeeper, kafka server, schema registry...etc.

https://www.confluent.io/download/

	confluent start

3. Start elasticsearch-sink connector for indexing the data from kafka server to Elasticsearch.

		confluent load elasticsearch-sink

4. Follow this link to push some test data to Elasticsearch from Kafka
https://docs.confluent.io/current/connect/connect-elasticsearch/docs/elasticsearch_connector.html

Get the data for index "test-elasticsearch-sink" from Elasticsearch
	
	curl -XGET 'http://localhost:9200/test-elasticsearch-sink/_search?pretty'

	'docs.count' will be incremented for the corresponding docs added for that particular index.

5. Get the list of indices from Elasticsearch

		curl 'localhost:9200/_cat/indices?v'

6. git clone avro source
	https://github.com/apache/avro.git

7. Go to java dir inside avro

		cd ~/git/avro/lang/java
		./build.sh dist
	
Note: if you face any issues related to javadoc, you can fix the errors or skip javadoc in mvn command.

8. Now you will have avro tools at ~/git/avro/lang/java/tools/target

9. Generate java files for specific avro schema file

		java -jar ~/git/avro/lang/java/tools/target/avro-tools-1.9.0-SNAPSHOT.jar compile schema user.avsc .
	
	Note: currently we are not using this file. But we may use it later for project specific schema..

10. Now you have example/avro/User.java file for this schema file

11. git clone of examples from confluent for java avro producer using schema registry


		git clone https://github.com/confluentinc/examples.git

Generate the java package using examples/kafka-clients/specific-avro-producer/readme.md file.
	
Run the program later using 
	
	java -cp target/uber-clickstream-generating-producer-1.0-SNAPSHOT.jar io.confluent.examples.producer.AvroClicksProducer http://localhost:8081

12. Add new topic 'clicks' to kafkaConnector, which is used by above program.

		curl -X PUT -H "Content-Type: application/json" --data '{"connector.class":"io.confluent.connect.elasticsearch.ElasticsearchSinkConnector","type.name":"kafka-connect","tasks.max":"1","topics":"test-elasticsearch-sink,clicks","name":"elasticsearch-sink","key.ignore":"true","connection.url":"http://localhost:9200"}' localhost:8083/connectors/elasticsearch-sink/config

13. Run the producer program by

		java -cp target/uber-clickstream-generating-producer-1.0-SNAPSHOT.jar io.confluent.examples.producer.AvroClicksProducer http://localhost:8081

13. Get the indices again from Elaticsearch. you can see that new indice is added with name 'clicks' and docs.count is getting incremented.

		curl 'localhost:9200/_cat/indices?v'
		
`-->$ curl 'localhost:9200/_cat/indices?v'
health status index                   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   clicks                  nSXiKJXES-232dwH8nMdiA   5   1        406            0    121.5kb        121.5kb
yellow open   test-elasticsearch-sink 6u15T7_IQ_WWqzEn5yil5Q   5   1          8            0     24.4kb         24.4kb

Get the data for indice 'clicks':

curl -XGET 'http://localhost:9200/clicks/_search?pretty'

{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 459,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "clicks",
        "_type" : "kafka-connect",
        "_id" : "clicks+0+15",
        "_score" : 1.0,
        "_source" : {
          "ip" : "66.249.1.1",
          "timestamp" : 1525331851669,
          "url" : "bar.html",
          "referrer" : "www.example.com",
          "useragent" : "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.125 Safari/537.36",
          "sessionid" : null
        }
      },
      {
        "_index" : "clicks",
        "_type" : "kafka-connect",
        "_id" : "clicks+0+16",
        "_score" : 1.0,
        "_source" : {
          "ip" : "66.249.1.5",
          "timestamp" : 1525333310267,
          "url" : "foo.html",
          "referrer" : "www.example.com",
          "useragent" : "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.125 Safari/537.36",
          "sessionid" : null
        }
      }


Ref:
https://github.com/confluentinc/examples.github

https://github.com/apache/avro.git

https://www.confluent.io/download/

https://www.elastic.co/downloads/elasticsearch

https://docs.confluent.io/current/schema-registry/docs/intro.html

https://docs.confluent.io/current/schema-registry/docs/serializer-formatter.html

Note: All the tests performed in Single PC as standalone mode. For distributed system, we may have to tweak the etc/../properties of confluent configuration to add node clustering for ELK, Kafka. Planning to run multiple VMs for distributed env in my PC. That's still in progress....


