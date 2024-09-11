# IoT Docker compose
Docker compose เป็นส่วนเริ่มต้นทุก service ที่อยู่ไฟล์ yaml 

## How to start docker compose
command
- docker compose up ในโฟลเดอร์ที่มีไฟล์ docker-compose.yml หรือใช้วิธีรัน คอนเทนเนอร์ service แต่ละส่วน เพื่อเริ่ม service แบบขั้นตอน

## ภายใน docker compose
- ส่วนนี้คือ volume สำหรับเป็นพื้นที่จะเก็บservice log และ service เริ่มต้นอย่าง Zookeper หน้าที่ของ Zookeper คือ ดูแลทุก serviceและดูแลข้อมูลรวมถึงช่วยประสานงานสำหรับ cluster kafka
```
volumes:
    prometheus_data: {}
    grafana_data: {}
    zookeeper-data:
      driver: local
    zookeeper-log:
      driver: local
    kafka-data:
      driver: local

services:

  # ZooKeeper is a centralized service for maintaining configuration information,
  # naming, providing distributed synchronization, and providing group services.
  # It provides distributed coordination for our Kafka cluster.
  # http://zookeeper.apache.org/
  zookeeper:
    image: confluentinc/cp-zookeeper
    container_name: zookeeper
    # ZooKeeper is designed to "fail-fast", so it is important to allow it to
    # restart automatically.
    restart: unless-stopped
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
      - zookeeper-log:/var/lib/zookeeper/log
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_LOG4J_ROOT_LOGLEVEL: INFO
      ZOOKEEPER_LOG4J_PROP: INFO,ROLLINGFILE
      ZOOKEEPER_LOG_MAXFILESIZE: 10MB
      ZOOKEEPER_LOG_MAXBACKUPINDEX: 10
      ZOOKEEPER_SNAP_COUNT: 10
      ZOOKEEPER_AUTOPURGE_SNAP_RETAIN_COUNT: 10
      ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: 3
```
- ส่วนของ service apache kafka สำหรับ streamimg ข้อมูล แบบ realtime ซึ่งการทำงานของ kafka นั้นจำเป็นต้องให้ kafka ให้addressไปยัง zookeeper เพื่อให้ customer รู้
```
# Kafka is a distributed streaming platform. It is used to build real-time streaming
  # data pipelines that reliably move data between systems and platforms, and to build
  # real-time streaming applications that transform or react to the streams of data.
  # http://kafka.apache.org/
  kafka:
    image: confluentinc/cp-kafka
    container_name: kafka
    volumes:
      - kafka-data:/var/lib/kafka
    restart: unless-stopped
    environment:
      # Required. Instructs Kafka how to get in touch with ZooKeeper.
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_NUM_PARTITIONS: 1
      KAFKA_COMPRESSION_TYPE: gzip
      # Required when running in a single-node cluster, as we are. We would be able to take the default if we had
      # three or more nodes in the cluster.
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      # Required. Kafka will publish this address to ZooKeeper so clients know
      # how to get in touch with Kafka. "PLAINTEXT" indicates that no authentication
      # mechanism will be used.
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
    links:
      - zookeeper
```
- ส่วนของ kafka rest proxy เป็น service ที่ให้ restful API เพื่อเชื่อมต่อกับ Kafka Cluster เพื่อเข้าถึงและจัดการข้อมูลใน kafka ผ่าน http api
```
 # The Kafka REST Proxy provides a RESTful interface to a Kafka cluster.
  # It makes it easy to produce and consume messages, view the state
  # of the cluster, and perform administrative actions without using
  # the native Kafka protocol or clients.
  # https://github.com/confluentinc/kafka-rest
  kafka-rest-proxy:
    image: confluentinc/cp-kafka-rest:latest
    container_name: kafka-rest-proxy
    environment:
      # Specifies the ZooKeeper connection string. This service connects
      # to ZooKeeper so that it can broadcast its endpoints as well as
      # react to the dynamic topology of the Kafka cluster.
      KAFKA_REST_ZOOKEEPER_CONNECT: zookeeper:2181
      # The address on which Kafka REST will listen for API requests.
      KAFKA_REST_LISTENERS: http://0.0.0.0:8082/
      # Required. This is the hostname used to generate absolute URLs in responses.
      # It defaults to the Java canonical hostname for the container, which might
      # not be resolvable in a Docker environment.
      KAFKA_REST_HOST_NAME: kafka-rest-proxy
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster will dynamically change. Thanks, ZooKeeper!
      KAFKA_REST_BOOTSTRAP_SERVERS: kafka:9092
    # Kafka REST relies upon Kafka, ZooKeeper
    # This will instruct docker to wait until those services are up
    # before attempting to start Kafka REST.
    restart: unless-stopped
    ports:
      - "9999:8082"
    depends_on:
      - zookeeper
      - kafka
```
- kafka connect เป็นส่วนในที่ kafka จะเชื่อมต่อกับระบบหรือservice ภายนอก เช่น iot-processor หรือ MongoDB หรือ MQTT protocol ทำให้สื่อสารเข้าออกจากkafkaได้โดยใช้ connectors ที่มีการตั้งค่าไว้
```
# Kafka Connect, an open source component of Apache Kafka,
  # is a framework for connecting Kafka with external systems
  # such as databases, key-value stores, search indexes, and file systems.
  # https://docs.confluent.io/current/connect/index.html
  kafka-connect:
    image: confluentinc/cp-kafka-connect:latest
    hostname: kafka-connect
    container_name: kafka-connect
    environment:
      # Required.
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster can dynamically change. Thanks, ZooKeeper!
      CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
      # Required. A unique string that identifies the Connect cluster group this worker belongs to.
      CONNECT_GROUP_ID: kafka-connect-group
      # Connect will actually use Kafka topics as a datastore for configuration and other data. #meta
      # Required. The name of the topic where connector and task configuration data are stored.
      CONNECT_CONFIG_STORAGE_TOPIC: kafka-connect-meta-configs
      # Required. The name of the topic where connector and task configuration offsets are stored.
      CONNECT_OFFSET_STORAGE_TOPIC: kafka-connect-meta-offsets
      # Required. The name of the topic where connector and task configuration status updates are stored.
      CONNECT_STATUS_STORAGE_TOPIC: kafka-connect-meta-status
      # Required. Converter class for key Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. Converter class for value Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. The hostname that will be given out to other workers to connect to.
      CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect"
      CONNECT_REST_PORT: 8083
      # The next three are required when running in a single-node cluster, as we are.
      # We would be able to take the default (of 3) if we had three or more nodes in the cluster.
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: "1"
      #Connectos path
      CONNECT_PLUGIN_PATH: "/usr/share/java,/data/connectors/"
      CONNECT_LOG4J_ROOT_LOGLEVEL: "INFO"
    restart: unless-stopped
    volumes:
      - ./kafka_connect/data:/data
    command: 
      - bash 
      - -c 
      - |
        echo "Launching Kafka Connect worker"
        /etc/confluent/docker/run & 
        #
        echo "Waiting for Kafka Connect to start listening on http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors ⏳"
        while [ $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) -ne 200 ] ; do 
          echo -e $$(date) " Kafka Connect listener HTTP state: " $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) " (waiting for 200)"
          sleep 5 
        done
        nc -vz $$CONNECT_REST_ADVERTISED_HOST_NAME $$CONNECT_REST_PORT
        echo -e "\n--\n+> Creating Kafka Connect MongoDB sink Current PATH ($$PWD)"
        /data/scripts/create_mongo_sink.sh 
        echo -e "\n--\n+> Creating MQTT Source Connect Current PATH ($$PWD)"
        /data/scripts/create_mqtt_source.sh
        echo -e "\n--\n+> Creating Kafka Connect Prometheus sink Current PATH ($$PWD)"
        /data/scripts/create_prometheus_sink.sh
        sleep infinity
    # kafka-connect relies upon Kafka and ZooKeeper.
    # This will instruct docker to wait until those services are up
    # before attempting to start kafka-connect.
    depends_on:
      - zookeeper
      - kafka
```
- ส่วนของ mosquitto ทำหน้าที่หลักในการประสานการสื่อสารระหว่างอุปกรณ์ IoT ในระบบ Mosquitto ทำหน้าที่เป็น message broker สำหรับโปรโตคอล MQTT ซึ่งจะจัดการกับการส่งข้อความระหว่าง clients โดยใช้โมเดล publish/subscribe ซึ่งช่วยให้ clients สามารถส่ง (publish) ข้อความไปยัง topics ที่กำหนด และ clients อื่นๆ สามารถรับ (subscribe) ข้อความจาก topics 
```
# Eclipse Mosquitto is an open source (EPL/EDL licensed) message broker that implements the MQTT protocol versions 5.0, 3.1.1 and 3.1. Mosquitto is lightweight and is suitable for use on all devices from low power single board computers to full servers.
  # The MQTT protocol provides a lightweight method of carrying out messaging using a publish/subscribe model. This makes it suitable for Internet of Things messaging such as with low power sensors or mobile devices such as phones, embedded computers or microcontrollers.
  # The Mosquitto project also provides a C library for implementing MQTT clients, and the very popular mosquitto_pub and mosquitto_sub command line MQTT clients.
  mosquitto:
    image: eclipse-mosquitto:latest
    hostname: mosquitto
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config

```
- ส่วนของ mongo DB เป็นเหมือนที่เก็บข้อมูลและที่พักข้อมูลที่รับมาจาก sensor เพื่อรอส่งไปยัง service ค่างๆที่เชื่อมต่อกับ Mongo DB

```
 mongo:
    image: mongo:4.4.20
    container_name: mongo
    env_file:
      - .env
    restart: unless-stopped
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
      - MONGO_INITDB_DATABASE=${MONGO_DB}
```
- ส่วนของ Visualization หรือส่วนแสดงผล โดย grafana จะแสดงผลในรูปแบบ dashboard โชว์ทุกข้อมูลที่รับมากจาก sensor 

```
# Grafana is a multi-platform open source analytics and interactive visualization web application. 
  # It provides charts, graphs, and alerts for the web when connected to supported data sources. 
  # It is expandable through a plug-in system. End users can create complex monitoring dashboards using interactive query builders.
  # https://github.com/hanattaw/grafana-flowcharting
  # from link above, some fixed for plugin flowcharting. You need to manually install plugin.
  grafana:
    image: grafana/grafana:latest-ubuntu
    container_name: grafana
    user: '0'
    volumes:
      - ./grafana/data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped
    links:
       - prometheus
    ports:
      - '8085:3000'
```
-  Prometheus ใช้สำหรับการตรวจสอบและการแจ้งเตือนเหตุการณ์ (event monitoring and alerting) โดยเน้นการเก็บข้อมูลเมตริกในรูปแบบ time series

```
# Prometheus is a free software application used for event monitoring and alerting.
  # It records real-time metrics in a time series database built using a HTTP pull model, with flexible queries and real-time alerting.
  # The project is written in Go and licensed under the Apache 2 License, with source code available on GitHub,
  # and is a graduated project of the Cloud Native Computing Foundation, along with Kubernetes and Envoy.
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/:/etc/prometheus/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    restart: unless-stopped
    ports:
      - '8086:9090'
```
- ส่วน iot_sensor_1 เป็น sensor จำลองโดยดึงจาก ssanchez11/iot_sensor:0.0.1-SNAPSHOT มาจำลองเพื่อส่งข้อมูลงไปยัง kafka

```
 # # IoT Sensor 1
  iot_sensor_1:
    image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT
    container_name: iot_sensor_1
    restart: unless-stopped
    environment:
      - sensor.id=${IOT_SENSOR_1_ID}
      - sensor.name=${IOT_SENSOR_1_NAME}
      - sensor.place.id=${IOT_SENSOR_1_PLACE_ID}
    depends_on:
      iot-processor:
        condition: service_started
        restart: true
```
- ส่วน iot-processor ใช้ ประมวลผลข้อมูลจากเซ็นเซอร์ IoT รับข้อมูลจาก Kafka Connect และประมวลผลข้อมูล IoT 

```
# IoT Processor
  iot-processor:
    image: ssanchez11/iot_processor:0.0.1-SNAPSHOT
    container_name: iot-processor
    restart: unless-stopped
    ports:
      - '8080:8080'
    depends_on:
      kafka-connect:
        condition: service_started
        restart: true
```


## วิธีการรัน script แต่ละ service เพื่อทำตามลำดับขั้นตอน
## start-service #0
```
#!/bin/bash

docker compose up zookeeper kafka
```
- script แรกเริ่มรัน container service zookeper และ kafka เพื่อเริ่มการประมวลผลข้อมูลและการดูแลserviceและดูแลcluster ทั้งหมด ส่วนนี้จะต้องเริ่มทำงานก่อน เมื่อส่วนนี้รันถึง Zookeper จะต้องรอทำงานก็ต้องเริ่ม script ส่วนต่อไป
- command
- $ sh start_0zookeeper_kafka.sh
## start-service #1
```
#!/bin/bash

docker compose up kafka-rest-proxy kafka-connect mosquitto mongo grafana prometheus
```
- เริ่มส่วน service ที่เป็นการประมวลผล, เก็บข้อมูล, และแสดงผล โดยใช้ service ที่เกี่ยวข้อง
- command เมื่อส่วนนี้ทำงานถึง kafka-connect เริ่ม respond 0 หรือ 200 ก็เริ่มส่วน script ของ iot-processor
- $ sh start_1kafka_service.sh
## start-service #2
```
#!/bin/bash

docker compose up iot-processor
```
- เริ่ม service iot-processor เพื่อให้มันเริ่มทำงานก่อนที่มีข้อมูลจาก sensor ส่งเข้ามา เราจะรู้ได้ว่าส่วนนี้ทำงานแล้วก็ต่อเมื่อส่วนของ kafka connect มีการ respond 200 
- command
- $ sh start_2iot_processor.sh
## start-service #3
```
#!/bin/bash

docker compose up iot_sensor_1
```
- เริ่ม container iot_sensor จำลองในการส่งข้อมูลไปให้ iot-processor ประมวลผลข้อมูลและส่งต่อไปยัง kafka connect
เราจะรู้output ก็ต่อเมื่อ มีข้อมูลแสดงที่ dashboard ของ grafana เรียบน้อย 
- command
- $ sh start_3iot_sensor.sh


## Error we found
บางครั้งอาจมีบริการหรือ container ที่พัง ลองลบ container หรือ ลบ volume และลอง docker compose up ใหม่

## How to solve the problems.


## Output

- [|] IoT Sensor - Dashboards - Grafana 
- [ ] UI for Apache Ka
- [ ] Mongo Expr
- [ ] Node Expor
- [ ] Prometheus Time Series Collection and Processing Ser
- [ ] Prometheus Pushgateway
- [ ] ZooNavigator


### IoT Sensor - Dashboards - Grafana URL
- จะมีค่าต่างๆออกมาที่dashboardของ grafana ทั้งค่า temp, humidity,ความดัน และกราฟ time series 

### UI for Apache Kafka

### Mongo Express

### Node Exporter

### Prometheus Time Series Collection and Processing Server

### Prometheus Pushgateway

### ZooNavigator