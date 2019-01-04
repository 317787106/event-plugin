# Tron eventsubscribe plugin

This is an implementation of Tron eventsubscribe model. 

* **api** module defines IPluginEventListener, a protocol between Java-tron and event plugin. 
* **app** module is an example for loading plugin, developers could use it for debugging.
* **kafkaplugin** module is the implementation for kafka, it implements IPluginEventListener, it receives events subscribed from Java-tron and relay events to kafka server. 

### Setup/Build

1. Clone the repo
2. Go to eventplugin `cd eventplugin` 
3. run `./gradlew build`

* This will produce one plugin zip, named `plugin-kafka-1.0.0.zip`, located in the `eventplugin/build/plugins/` directory.


### Edit **config.conf** of Java-tron， add the following fileds:
```
event.subscribe = {
    path = "/Users/tron/sourcecode/eventplugin/build/plugins/plugin-kafka-1.0.0.zip"
    server = "127.0.0.1:9092"
    topics = [
        {
          triggerName = "block"
          enable = false
          topic = "block"
        },
        {
          triggerName = "transaction"
          enable = false
          topic = "transaction"
        },
        {
          triggerName = "contractevent"
          enable = true
          topic = "contractevent"
        },
        {
          triggerName = "contractlog"
          enable = true
          topic = "contractlog"
        }
    ]
}

```
 * **path**: is the absolute path of "plugin-kafka-1.0.0.zip"
 * **server**: Kafka server address
 * **topics**: each event type maps to one Kafka topic, we support four event types subscribing, block, transaction, contractlog and contractevent.
 * **triggerName**: the trigger type, the value can't be modified.
 * **enable**: plugin can receive nothing if the value is false.
 * **topic**: the value is the kafka topic to receive events. Make sure it has been created and Kafka process is running

##### Install Kafka
On Mac:
```
brew install kafka
```

On Linux:
```
cd /usr/local
wget http://archive.apache.org/dist/kafka/0.8.2.1/kafka_2.11-0.8.2.1.tgz
tar -xzvf kafka_2.11-0.8.2.1.tgz
mv kafka_2.11-0.8.2.1 kafka

add "export PATH=$PATH:/usr/local/kafka/bin" to end of /etc/profile
source /etc/profile


kafka-server-start.sh /usr/local/kafka/config/server.properties &
```


##### Run Kafka
On Mac:
```
zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties
```

On Linux:
/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties &


#### Create topics to receive events, the topic is defined in config.conf

On Mac:
```
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic block
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic transaction
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic contractlog
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic contractevent
```

On Linux:
```
kafka-topics.sh  --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic block
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic transaction
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic contractlog
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic contractevent
```

#### Kafka consumer

On Mac:
```
kafka-console-consumer --bootstrap-server localhost:9092  --topic block --from-beginning
kafka-console-consumer --bootstrap-server localhost:9092  --topic transaction --from-beginning
kafka-console-consumer --bootstrap-server localhost:9092  --topic contractlog --from-beginning
kafka-console-consumer --bootstrap-server localhost:9092  --topic contractevent --from-beginning
```

On Linux:
```
kafka-console-consumer.sh --zookeeper localhost:2181 --topic block --from-beginning
kafka-console-consumer.sh --zookeeper localhost:2181 --topic transaction --from-beginning
kafka-console-consumer.sh --zookeeper localhost:2181 --topic contractlog --from-beginning
kafka-console-consumer.sh --zookeeper localhost:2181 --topic contractevent --from-beginning
```

### Load plugin in Java-tron
* add --es to command line, for example:
```
 java -jar FullNode.jar -p privatekey -c config.conf --es 
```

#### Set event plugin config via http
```
url: http://localhost:8090/wallet/setEventPluginConfig
parameters: TriggerInfo list, which are serialized in json format

message TriggerInfo{
    string triggerName = 1;
    bool enable = 2;
    string topic = 3;
}

```

### Set event plugin config via grpc
```
Wallet-cli:setEventPluginConfig
command: setEventPluginConfig  block|true transaction|false
```

### Event filter
which is defined in config.conf, path: event.subscribe
```
filter = {
       fromblock = "" // the value could be "", "earliest" or a specified block number as the beginning of the queried range
       toblock = "" // the value could be "", "latest" or a specified block number as end of the queried range
       contractAddress = [
           "TVkNuE1BYxECWq85d8UR9zsv6WppBns9iH" // contract address you want to subscribe, if it's set to "", you will receive contract logs/events with any contract address.
       ]

       contractTopic = [
           "f0f1e23ddce8a520eaa7502e02fa767cb24152e9a86a4bf02529637c4e57504b" // contract topic you want to subscribe, if it's set to "", you will receive contract logs/events with any contract topic.
       ]
    }
```
#### Set event plugin filter via http
```
url: http://localhost:8090/wallet/seteventfilter
parameters: EventFilter object, which is seralized in json format

message EventFilter {
    string fromBlock = 1;
    string toBlock = 2;
    repeated string contractAddress = 3;
    repeated string contractTopic = 4;
}

```

#### Set eventFilter via grpc
```
Wallet-cli: seteventfilter
command: seteventfilter 100 1000 address1|address2 topic1|topic2
if parameter is null, input *
for example: seteventfilter * * * *, which means any contract log or event will be subscribed.
```

### Tron event subscribe model
```
* Please refer to https://github.com/tronprotocol/TIPs/issues/12
```