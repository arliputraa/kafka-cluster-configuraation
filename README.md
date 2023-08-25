# Setup Kafka Cluster 3 vm kafka 1 zookeeper

### Create the log directories in file path /home/kafka

    mkdir logs1
    mkdir logs2
    mkdir logs3
    
### Copy server.preperties in file path /home/kafka/config 
    
    cp config/server.properties config/server1.properties
    cp config/server.properties config/server2.properties 
    cp config/server.properties config/server3.properties 

After copy the server properties append this conf:

server1.properties

    broker.id=1
    listeners=PLAINTEXT://192.168.18.198:9092
    advertised.listeners=PLAINTEXT://192.168.18.198:9092
    zookeeper.connect=192.168.18.198:2181
    listeners=PLAINTEXT://:9093
    log.dirs=/kafka/logs1

server2.properties

    broker.id=2
    listeners=PLAINTEXT://192.168.18.204:9093
    advertised.listeners=PLAINTEXT://192.168.18.204:9093
    zookeeper.connect=192.168.18.198:2181
    listeners=PLAINTEXT://:9094
    log.dirs=/kafka/logs2

server3.properties

    broker.id=3
    listeners=PLAINTEXT://192.168.18.205:9094
    advertised.listeners=PLAINTEXT://192.168.18.205:9094
    zookeeper.connect=192.168.18.198:2181
    listeners=PLAINTEXT://:9095
    log.dirs=/kafka/logs3


### Create zookeeper systemd service in node kafka1

    sudo nano /etc/systemd/system/zookeeper.service


Append this:

    [Unit]
    Requires=network.target remote-fs.target
    After=network.target remote-fs.target
    
    [Service]
    Type=simple
    User=devarli
    ExecStart=/home/devarli/kafka/bin/zookeeper-server-start.sh /home/devarli/kafka/config/zookeeper.properties
    ExecStop=/home/devarli/kafka/bin/zookeeper-server-stop.sh
    Restart=on-abnormal
    
    [Install]
    WantedBy=multi-user.target

### Start zookeeper

    sudo systemctl start zookeeper.service

### Create service kafka IN EACH VM/SERVER 

    sudo nano /etc/systemd/system/kafka.service

Append this:

    [Unit] 
    Requires=zookeeper.service 
    After=zookeeper.service 
      
    [Service] 
    Type=simple 
    User=devarli
    ExecStart=/bin/sh -c '/home/devarli/kafka/bin/kafka-server-start.sh /home/devarli/kafka/config/server1.properties > /home/devarli/kafka/logs/kafka.log 2>&1' 
    ExecStop=/home/devarli/kafka/bin/kafka-server-stop.sh 
    Restart=on-abnormal 
      
    [Install] 
    WantedBy=multi-user.target

### Start kafka IN EACH VM/SERVER in file path /home/kafka/bin

    sudo systemctl start kafka

## Kafka Connect setup
For integration to postgresql and neo4j we use neo4j connector and debezium postgresql connector, its mean we need kafka connect because the connector running in kafka connect

### Create connect configuration 
Kafka connect ONLY set up IN NODE KAFKA1

in file path /home/kafka

    mkdir connect && cd connect

in file path /home/kafka/connect 

    wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/1.9.6.Final/debezium-connector-postgres-1.9.6.Final-plugin.tar.gz && tar -xvf debezium-connector-postgres-1.9.6.Final-plugin.tar.gz
    wget https://github.com/neo4j-contrib/neo4j-streams/releases/download/4.1.2/neo4j-kafka-connect-neo4j-2.0.2-kc-oss.zip 
    sudo apt install unzip (if not install unzip)
    unzip neo4j-kafka-connect-neo4j-2.0.2-kc-oss.zip

### Copy connect.properties to kafka-connect-properties in file path /home/kafka/config

    cp connect.properties kafka-connect-properties
    nano kafka-connect-properties

Append this:

    bootstrap.servers=192.168.18.198:9092 #bootsrap server is kafka1 ip
    listeners=HTTP://192.168.18.198:8083
    plugin.path=/home/devarli/kafka/connect #that connect directory we create before

### Create kafka connect running as systemd

    sudo nano /etc/systemd/system/kafka_connect.properties

Append this:

    [Unit]
    Requires=kafka.service
    After=kafka.service
    
    [Service]
    Type=simple
    User=devarli
    ExecStart=/bin/sh -c '/home/kafka1/kafka/bin/connect-distributed.sh /home/kafka1/kafka/config/kafka-connect.properties > /home/kafka1/kafka/logs1/kafka_connect.log 2>&1'
    Restart=on-abnormal
    
    [Install]
    WantedBy=multi-user.target

### After edited the systemd config running the kafka_connect in /home/kafka/bin

    systemctl start kafka_connect.service
    systemctl status kafka_connect.service

![image](https://github.com/arliputraa/kafka-cluster-configuration/assets/110078907/7a59bc67-217e-4464-a570-e8c44757fbcf)

### For more learning Apache Kafka in this link: https://github.com/arliputraa/apache-kafka-instalation

### Optional add monitoring Kafdrop for gui kafka in this link: https://github.com/arliputraa/kafdrop-configuration-for-gui-kafka 

![image](https://github.com/arliputraa/kafka-cluster-configuraation/assets/110078907/3e094f36-9bc3-416e-af4b-d623868c604f)

