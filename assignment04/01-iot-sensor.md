# Ingest and store real-time data from IoT sensors.


## MQTT Topic
MQTT Topic คือหัวข้อที่ใช้ในการรับส่งข้อมูลระหว่าง Publisher และ Subscriber ในระบบ MQTT
หลักการทำงานของ MQTT คือการประกาศTopic ใน Broker จากนั้น Publisher จะส่งข้อมูลไปยัง Topic นั้นๆ และ Subscriber ก็จะได้รับข้อมูลทั้งหมดใน Topic นั้นๆโดยในcodeผมคือ client.publish("iot-frames", jsonData); ซึ่งtopicคือ iot-frames


## MQTT Payload
 esp32s2dev module cucumberจะรับข้อมูลต่าๆจากsensorที่กำหนด เพื่อส่งไปยัง mqtt serverเเต่การส่งข้อมูลไปหาmqtt serverจะต้องมีการกำหนดรูปแบบของ Payloadเพื่อให้mqtt serverสามารถรับข้อมูลได้ โดยรูปเเบบPayloadของกลุ่มผมคือ
 ```StaticJsonDocument<512> jsonDoc;
jsonDoc["id"] = "43245253";  // รหัสเซ็นเซอร์
jsonDoc["name"] = sensor_name;  // ชื่อของเซ็นเซอร์
jsonDoc["place_id"] = "42343243";  // รหัสสถานที่
jsonDoc["date"] = NTP.getTimeDateString(time(NULL), "%Y-%m-%dT%H:%M:%S");  // วันที่และเวลาในรูปแบบ ISO 8601
jsonDoc["timestamp"] = epochTime;  // เวลาปัจจุบันในรูปแบบ epoch time
JsonObject payload = jsonDoc.createNestedObject("payload");
payload["temperature"] = temp;      // อุณหภูมิที่วัดได้จาก BMP280
payload["humidity"] = humid;        // ความชื้นที่วัดได้จาก SHT4x
payload["pressure"] = pressure;     // ความดันอากาศที่วัดได้จาก BMP280
payload["luminosity"] = analogval;  // ความเข้มแสงที่วัดได้จากเซ็นเซอร์แบบอนาล็อก```

char jsonData[512];
serializeJson(jsonDoc, jsonData);
client.publish("iot-frames", jsonData);

การใช้งาน MQTT Payload
Publisher จะสร้าง Payload โดยใส่ข้อมูลลงใน JSON Document จากนั้นใช้ฟังก์ชัน serializeJson() เพื่อแปลงข้อมูล JSON ให้อยู่ในรูปแบบข้อความ แล้วจึงส่งข้อมูลนี้ผ่านคำสั่ง client.publish("iot-frames", jsonData);
Subscriber จะต้องดักรับข้อมูลจาก Topic ที่สนใจ เช่น "iot-frames" และทำการถอดรหัส JSON Payload เพื่อดึงข้อมูลที่ต้องการออกมาใช้งาน

## ESP32
จากboardที่อ.ให้มาคือ esp32s2dev module cucumberโดยsensorที่ใช้จะมี BMP280 ใช้วัดอุณหภูมิและความดัน SHT4x วัดความชื้น เเละ KY-018ที่ต่อเพิ่มมาตรงขา5เพื่อใช้วัดค่าเเสง รวมถึงมีการให้ทำledในการดูสถานะต่างๆ
การทำงาน esp32s2dev module cucumberของผมคือ
1 เมื่อเครื่องเปิดไฟจะสีเเดง
2 ทำการเชื่อมต่อwifiหากเชือมต่อไม่ได้หรือกำลังเชื่อมจะขึ้นไฟสีเหลือง
    2.1 หากเชือมต่อไม่ได้หรือกำลังเชื่อมจะขึ้นไฟสีเหลือ
    2.2 หากเชื่อมต่อได้เเล้วจะขึ้นไฟสีเขียว
3 ทำการเชื่อมต่อmqtt server
    3.1 หากเชือมต่อwifiไม่ได้ให้กลับไปทำfunctionเชื่อมต่อwifi
    3.2 หากเชื่อมต่อwifiได้เเต่เชื่อมmqtt serverไม่ได้ให้ทำการเชื่อมmqtt serverใหม่อีก5วินาทีเเละขึ้นไฟสีขาว
    3.3 หากเชื่อมmqtt serverได้ให้เเสดงไฟสีฟ้า
4 เช็คerrorของsensorต่างๆ
5 อ่านข้อมูลจากsensorต่างๆ
ุ6 เเสดงค่าที่รับจากsensorใน Serial Monitor
7 นำข้อมูลที่ได้จากsensorต่างๆมาเปลี่ยนเป็นรูปเเบบJSONเเละส่งข้อมูลทุกๆ5วินาทีโดยทุดๆครั้งที่ส่งไฟจะเป็นสีน้ำเงิน