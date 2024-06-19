# Main technologies of architecture

## Architecture Overview

![IoT Event Streaming Architecture](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*IUaBLlbVKgmsjbjqzew0ZQ.png)

## Eclipse Mosquitto
Eclipse Mosquito เป็น software opensourceสำหรับการรับส่งข้อความแบบ MQTT(Message Queuing Telemetry Transport เป็น
 โปรโตคอลทสอสารระหว่างอุปกรณ์ต่างๆเป็นแบบ โมเดล public/subscribe ) ซงนำมาใช้งานในการเชื่อมต่ออปปกรณ์ IoTต่างๆ  
Eclipse Mosquito มี 3องค์ประกอบ 
Publisher เป็นส่วนทpublisherส่งข้อความหรือข้อมูลไปยัง broker 
Broker เป็น serverที่รับข้อความหรือข้อมูลจากPublisherแล้วจัดส่งให้กับsubcriber 
Subcriber เป็นผู ้รับข้อมูลและข้อความจากbroker 


## Apache ZooKeeper
เป็นบิรการแบบรวมศูนยสำหรับการรักษาข้อมูลการกำหนดค่า การตั้งชื่อ การซิงโครไนซ์แบบกระจาย และการให้บริการแบบ
 กลุ่ม บริการประเภทนี้ทั้งหมดถกูใช้ในรูปแบบใดรูปแบบหนึ่งโดยแอปพลิเคชั่นแบบกระจาย เป็นตัวจัดการ borker อยู่ตำแหน่ง
 ไหน แจ้ง kafka เมื่อมีการเปลี่ยนแปลง บันทึกค่าตั้งค่า configuration  


## Apache Kafka
เป็น platform สาหรับการรับส่งข้อมลแบบกระจายสามารถประมวลผล และจัดการข้อมูลแบบเรียลไทม์เป็น platform ที่มี
 ประสิทธภาพและใช้งานได้ง่ายเหมาะสำหรับการประมวลข้อมูลและวิเคราะห์ข้อมูล 


## Apache Kafka Connect
ส่วนประกอบเสริมหนึ่งของ Apache Kafka เป็นเฟรมเวิรก์สำหรับการเชื่อมต่อ Kafka กับระบบภายนอก เช่น ฐานข้อมูล การ
 จดเก็บค่าคีย์ และระบบไฟล์ มุ่งเน้นไปที่การสตรีมข้อมูลเข้าและออกจาก Kafka เป็นองค์ประกอบสำคัญของไปป์ไลน์ ETL เมื่อ
 รวมกับ Kafka และเฟรมเวิร์กการประมวลผลสตรีม 


## Apache Kafka Streams
Kafka Streams คือไลบรารีไคลเอ็นต์สำหรับสร้างแอปพลิเคชันและไมโครเซอร์วิส โดยที่ข้อมูลอินพุตและเอาต์พุตจะถูกจัดเก็บ
 ไว้ในคลัสเตอร์ Apache Kafka เป็นการผสมผสานความเรียบงายของการเขียนและการปรับใชแอปพลิเคชั่น Java และ Scala 
มาตรฐานบนฝั่งไคลเอ็นต์เข้ากับประโยชนของเทคโนโลยคลัสเตอร์ฝั่งเซิร์ฟเวอร์ของ Kafka 


## Prometheus



## MongoDB



## Grafana


