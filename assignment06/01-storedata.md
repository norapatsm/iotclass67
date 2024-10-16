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