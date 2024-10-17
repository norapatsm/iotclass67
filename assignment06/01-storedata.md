# Store data.
- เป็นส่วนที่เก็บข้อมูลหรือที่พักข้อมูลชั่วคราวและค่อยส่งไปยังserviceอื่นๆตามกระบวนการ ส่ววนี้ใช้ MongoDB เป็นฐานข้อมูลและใช้ Mongo Express เป็นตัวจัดการ MongoDB

![Example Image](storedata.png)


# Prometheus
- เป็นส่วนสำหรับการตรวจสอบเหตุการณ์ภายในการทำงานและโครงสร้างของ IoT Event Streaming Architecture สำหรับ  event monitoring การแจ้งเตือนต่างๆ. บันทึกข้อมูลประเภทเมตริกแบบ Real time 

- Prometheus ถูกใช้ในการจัดการและแสดงผลข้อมูลเกี่ยวกับ Metrics ของระบบ เช่น ประสิทธิภาพการทำงานของ Kafka, การใช้งานทรัพยากรของเซิร์ฟเวอร์, และข้อมูลการมอนิเตอร์เซ็นเซอร์แบบเรียลไทม์.

- ช่วยให้สามารถติดตามการทำงานของระบบต่างๆ ในคลัสเตอร์ได้อย่างละเอียด
```cpp
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
    depends_on:
      - nodeexporter
```


# MongoDB
- เป็นส่วนจัดเก็บข้อมูลของ IoT Event Streaming Architecture และเป็นฐานข้อมูลแบบ NoSQL 
- ภายใน mongoDB จะมีข้อมูลที่ถูกประมวลจาก iot-sensor ต่างๆ ทั้ง tmp, humidity เพื่อจัดเก็บข้อมูลจาก iot-sensor หรือ อุปกรณ์ โดยทำงานร่วมกับ Service อื่นๆ ได้ เช่น kafka-connect, Prometheus
```cpp
 mongo:
    image: mongo
    container_name: mongo
    env_file:
      - .env
    restart: unless-stopped
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
      - MONGO_INITDB_DATABASE=${MONGO_DB}
```

## Config
# 1. iot-aggregate-metrics-place-mongodb-sink
```cpp
{
   "name":"iot-aggregate-metrics-place-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-place",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_place",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```
- Config คือการกำหนดให้ Kafka Connector รับข้อมูลจาก Kafka topic ชื่อ iot-aggregate-metrics-place และส่งข้อมูลนั้นไปยังฐานข้อมูล 
MongoDB โดยจะบันทึกข้อมูลลงใน collection ชื่อ iot_aggregate_metrics_place ภายฐานข้อมูล ของ iot โดยข้อมูลจะถูกแปลงเป็น 
JSON ก่อนที่จะถูกบันทึกใน MongoDB

# 2 iot-aggregate-metrics-sensor-mongodb-sink 
```cpp
{
   "name":"iot-aggregate-metrics-sensor-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-sensor",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_sensor",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```
- Kafka Connector นี้จะรับข้อมูลจาก Kafka topic คือ iot-aggregate-metrics-sensor และนำข้อมูลไปบันทึกในฐานข้อมูล MongoDB โดยจะถูกจัดชื่อ iot_aggregate_metrics_sensor 

# 3 iot-frames-mongodb-sink
```cpp
{
   "name":"iot-frames-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-frames",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_frames",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.json.JsonConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```
- ทำงานเช่นเดียวกันกับส่วนบน รับข้อมูลจาก kafka topic (iot-frames) และบันทึกลง MongoDB โดยจะแปลงทั้ง key และ value ของ Kafka message เป็น JSON ก่อนบันทึก

# 4 mqtt-source
```cpp
{
    "name": "mqtt-source",
    "config": {
        "connector.class": "io.confluent.connect.mqtt.MqttSourceConnector",
        "tasks.max": 1,
        "mqtt.server.uri": "tcp://mosquitto:1883",
        "mqtt.topics": "iot-frames",
        "mqtt.username": "kafka-connect",
        "mqtt.password": "kafka-connect",
        "mqtt.qos": 1,
        "kafka.topic": "iot-frames",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable":false,
        "converter.encoding": "UTF-8",
        "value.converter.schemas.enable": false,
        "confluent.topic.bootstrap.servers": "kafka:9092",
        "confluent.topic.replication.factor": 1,
        "transforms":"createMap,createKey,extractString",
        "transforms.createMap.type":"org.apache.kafka.connect.transforms.HoistField$Value",
        "transforms.createMap.field":"id",
        "transforms.createKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
        "transforms.createKey.fields":"id",
        "transforms.extractString.type":"org.apache.kafka.connect.transforms.ExtractField$Value",
        "transforms.extractString.field":"id"
    }
}
```
- Kafka Connector นี้จะเชื่อมต่อกับ MQTT (Mosquitto) และรับข้อมูลจาก MQTT topic ชื่อ iot-frames 
ข้อมูลจะถูกแปลงเป็นรูปแบบByteArray และ JSON ก่อนส่งไปยัง Kafka topic ชื่อ  โดยมีการ process key 
เพื่อสร้างและดึงค่า id สำหรับใช้งานเป็น key ใน Kafka.


# 5 prometheus-connector-sink
```cpp
{
  "name" : "prometheus-connector-sink",
  "config" : {
   "topics":"iot-metrics-time-series",
   "connector.class" : "io.confluent.connect.prometheus.PrometheusMetricsSinkConnector",
   "tasks.max" : "1",
   "confluent.topic.bootstrap.servers":"kafka:9092",
   "prometheus.scrape.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "prometheus.listener.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "value.converter": "org.apache.kafka.connect.json.JsonConverter",
   "key.converter": "org.apache.kafka.connect.json.JsonConverter",
   "value.converter.schemas.enable": false,
   "key.converter.schemas.enable":false,
   "reporter.bootstrap.servers": "kafka:9092",
   "reporter.result.topic.replication.factor": "1",
   "reporter.error.topic.replication.factor": "1",
   "behavior.on.error": "log"
  }
}
```
- ในส่วน kafka connector นี้จะรับข้อมูลจาก Kafka และส่งไปยัง Prometheus เพื่อการมอนิเตอร์และเก็บข้อมูล metrics โดยใช้ PrometheusMetricsSinkConnector
Prometheus จะตรวจสอบและนำข้อมูลเหล่านี้ไป Monitoring และเก็บ metrics