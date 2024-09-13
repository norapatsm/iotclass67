# MQTT

## iot-sensor-2
คือการจำลองข้อมูลของเซ็นเซอร์ซึ่งที่อยู่ในคอมพิวเตอร์ทำงานภายใน Docker โดยข้อมูลนี้เป็นค่าที่ได้จากการ generateขึ้น จากนั้นส่งข้อมูลผ่าน Protocal MQTT ไปยัง Server เพื่อให้ประมวลผลกับแสดงผล

(ภาพตอนรันโปรแกรม)
![Example Image](code(1).png)

![Example Image](code(2).png)

![Example Image](docker.png)

![Example Image](docker2.png)

## Spring Boot
คือ Framework ใน Spring อันหนึ่ง สามารถช่วยทำให้สร้าง Web application หรือ Web serviceได้ง่ายขึ้น เพราะ Spring Boot มี Auto Configuration ซึ่งช่วยลดความยุ่งยากในการกำหนดค่าต่างๆ และสามารถใช้งานได้ทันที เนื่องจาก Spring Boot มี Java Web Server ที่ built-in มาให้แล้ว ก็คือ Tomcat ทำให้ง่ายต่อการใช้งาน โดยมี Port default คือ 8080 ซึ่งสามารถแก้ไขเปลี่ยน Port ได้ที่ไฟล์ application.properties

## Apache Maven
เป็นเครื่องมือตัวนึงที่ช่วยให้นักพัฒนา (Developer) เขียน Java Application ได้ง่ายขึ้น เป็น Project Management Tools ที่คอยอำนวยความสะดวกสบายในการสร้าง Java Project 
Spring Boot และ Maven ใช้เพื่อ Compile Java ของ iot_senser_2 ใน PC เพื่อให้ sensor ทำงานภายในเครื่อง แล้วส่งข้อมูลไปยัง MQTT 

คำสั่ง Maven
docker run -it --rm --name my-maven-project -v "${PWD}":/usr/src/mymaven -w /usr/src/mymaven maven:3.8-openjdk-8 mvn clean install
docker run -it --rm --name my-maven-project -v .:/usr/src/mymaven -w /usr/src/mymaven maven:3.8-openjdk-8 mvn clean install
