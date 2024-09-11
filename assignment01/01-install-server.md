# Install Server and Docker

rufus-3.13.exe
-install ubuntu-20.04.6-live-server-amd64.iso ลงในเครื่องที่ต้องการที่จะเป็น server สำหรับ ทุก service ในการทำงานของ IOT-EVENT-streaming


## How to install Server
install ubuntu-20.04.6-live-server-amd64.iso และนำ ubuntu-server ลงที่ gateway dell
และ configuration ชื่อของ ubuntu, password, ip ของ server, และจัดการ พื้นที่disk และรอให้ 
ubuntu setup จนเสร็จสิ้น
จนเห็นส่วนที่ให้ ใส่ชื่อและpassword นั่นคือ install server เสร็จสมบูรณ์ 

## How to install Docker
การ install docker 
conf: https://docs.docker.com/engine/install/ubuntu/
ทำตาม url และ command ต่างๆ
# Add Docker's official GPG key:
1.Set up Docker's apt repository.
#sudo apt-get update
#sudo apt-get install ca-certificates curl
#sudo install -m 0755 -d /etc/apt/keyrings
#sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
#sudo chmod a+r /etc/apt/keyrings/docker.asc

2.Install the Docker packages.
@ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

3.Verify that the Docker Engine installation is successful by running the hello-world image.
#sudo docker run hello-world

tip:(แนะนำให้ ไม่ควรใช้ root หรือ sudo ในการใช้ command สำหรับการทำงานของ docker ทุกประเภท)
1. Create the docker group.
#sudo groupadd docker
2. Add your user to the docker group.
#sudo usermod -aG docker $USER
3. ออกจากระบบและเข้าสู่ระบบใหม่อีกครั้งเพื่อให้การเป็นสมาชิกกลุ่มของคุณได้รับการประเมินใหม่
#newgrp docker
4. ตรวจสอบว่าคุณสามารถรันdockerคำสั่งได้โดยไม่ต้องใช้sudo.
#docker run hello-world

#sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
#sudo chmod g+rwx "$HOME/.docker" -R


