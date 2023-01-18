<p align="center"><a href="https://pve22.ipv9.me/#v1:0:18:4:::::::"> <img src="Screenshots/logo.png"width=300 alt="Paris"></a><a href="http://172.31.0.211/icingaweb2"> <img src="Screenshots\Icinga_logo.svg"width=300 alt="Paris"></a></p>

# <p align="center">การติดตั้ง และการใช้งาน Icinga ผ่าน Container ใน Proxmox</p>

-----

# สารบัญ
|<p align="center"> เนื้อหา </p>| 
| ------------- |
|<DL>[การติดตั้ง Icinga](#การติดตั้ง-icinga)<DT>[\|-ส่วน IDODB](#ส่วน-idodb)<DT><DT>[\|-ส่วน Icinga Core](#ส่วน-icinga-core)<DT><DT>[\|-ส่วน Icinga Web](#ส่วน-icinga-web)</DT></DL>|
|<DL>[การใช้งาน Icingaweb2](#การใช้งาน-icingaweb2)<DT>[\|-การจัดการ Dashboard](#การจัดการ-dashboard)<DT><DT>[\|-การสร้าง Host](#การสร้าง-host)<DT>[\|-การใช้ SNMP ในการ Monitor](#การใช้-snmp-ในการ-monitor)</DT></DL>|
[การอ่านผลลัพธ์ของ Icingaweb2](#การอ่านผลลัพธ์-icingaweb2)


# การติดตั้ง Icinga
[top⬆️](#สารบัญ)

การติดตั้ง Icinga จะประกอบด้วย 3 ส่วนตาม สถาปัตยกรรม ของ Icinga คือ Icinga Core , IDODB และ Icinga Web
<p align="center"> <img src="Screenshots\Icinga_Architecture_v1.5.png"alt="Paris"></p>


## การ สร้าง Container ใน Proxmox เพื่อติดตั้ง Icinga
ในการติดตั้ง Container ใน Proxmox สามารถทำได้เหมือนที่อธิบายไปใน [SPCN-011](https://github.com/phisic1714/SPCN-011#%E0%B8%AA%E0%B8%A3%E0%B9%89%E0%B8%B2%E0%B8%87-container-template-%E0%B9%80%E0%B8%A5%E0%B8%B7%E0%B8%AD%E0%B8%81%E0%B8%88%E0%B8%B2%E0%B8%81-ct-list) แต่จะมีการปรับเปลี่ยนค่าในเรื่องของ Network และ DNS ดังนี้

* ส่วน Network
        
        IPv4/CIDR = 172.31.0.211/23 
        (หรือเลข IP อะไรก็ได้ที่ไม่ชนกับ IP อื่น)
        Gateway(IPv4) = 172.31.1.254
 <p align="center"> <img src="Screenshots\(75).png"alt="Paris"></p>

* ส่วน DNS 

        DNS domain = 202.28.187.202
        DNS servers = 172.31.1.254
<p align="center"> <img src="Screenshots\(76).png"alt="Paris"></p>



## ส่วน IDODB
จะเป็นส่วนที่เป็นฐานข้อมูลการ Monitor Network ต่างๆ โดยรับข้อมูลจาก Icinga Core และส่งข้อมูลไป Icinga Web การเก็บจะใช้รูปแบบ Mysql ใช้ฐานข้อมูล MariaDB ในการสร้างตาราง
### ขั้นตอนที่ 1: ติดตั้ง Apache, MariaDB และ PHP
1. เราเริ่มต้นด้วยการติดตั้ง Apache, MariaDB และ PHP ด้วยชุดคำสั่งนี้

        $ apt install apache2 mariadb-server mariadb-client mariadb-common php php-gd php-mbstring php-mysqlnd php-curl php-xml php-cli php-soap php-intl php-xmlrpc php-zip  php-common php-opcache php-gmp php-imagick php-pgsql -y

2. เมื่อติดตั้งแล้ว ตรวจสอบให้แน่ใจว่าบริการทั้งหมดกำลังทำงานอยู่ ถ้าใช่ ให้รันคำสั่งต่อไปนี้

        $ systemctl start {apache2,mariadb}
        $ systemctl enable {apache2,mariadb}
        $ systemctl status {apache2,mariadb}
3. ด้วยโมดูล PHP คุณต้องแก้ไขไฟล์ php.ini ซึ่งเป็นไฟล์กำหนดค่าเริ่มต้นสำหรับแอปพลิเคชันที่ทำงานบน PHP
เปิดไฟล์โดยใช้โปรแกรมแก้ไขที่คุณต้องการ ที่นี่. เรากำลังใช้ตัวแก้ไขบรรทัดคำสั่งนาโน

        $ nano /etc/php/8.1/apache2/php.ini
    
    จากนั้นทำการเปลี่ยนแปลงกับพารามิเตอร์

        date.timezone = "ทวีป/เมืองหลวงในพื้นที่ปัจจุบัน"

<p align="center"> <img src="Screenshots\224210.png"alt="Paris"></p>

4. Restart Apache โดยใช้คำสั่ง 

        $ systemctl restart apache2

## ส่วน Icinga Core
ส่วนที่จะทำการประมวณผลการ Monitor Network ที่รับมาจาก Icinga Web จึงต้องใช้ IDO Module ส่งข้อมูลไปเก็บใน IDODB 
### ขั้นตอนที่ 1: ติดตั้ง Icinga2 บน Ubuntu

1. อัพเดต apt และติดตั้ง Icinga2 และปลั๊กอินการตรวจสอบ

        $ apt update
        $ apt install icinga2 monitoring-plugins

2. เมื่อการติดตั้งเสร็จสิ้น ให้เปิดใช้งานและเริ่มบริการ Icinga2

        $ systemctl enable icinga2
        $ systemctl start icinga2

3. เพื่อยืนยันว่าบริการ Icinga2 กำลังทำงานอยู่ ให้ดำเนินการ

        $ systemctl status icinga2


<p align="center"> <img src="Screenshots\004346.png"alt="Paris"></p>

ผลลัพธ์บ่งชี้ว่า Icinga2 daemon กำลังทำงานและเราพร้อมดำเนินการ

### ขั้นตอนที่ 2: ติดตั้ง Icinga2 IDO Module
Icinga2 Data Output (IDO) ส่งออกข้อมูลการกำหนดค่าและสถานะทั้งหมดไปยังฐานข้อมูล ฐานข้อมูล IDO ถูกใช้โดย Icinga Web 2 เป็นแบ็กเอนด์ข้อมูล

1. ติดตั้ง IDO Module ให้รันคำสั่ง

        $ apt install icinga2-ido-mysql -y
    <p align="center"> <img src="Screenshots\Configure-ido-mysql-Module.png"alt="Paris"></p>

    ระหว่างทาง จะมีป๊อปอัปปรากฏขึ้นบนเครื่องเทอร์มินัล ในการเปิดใช้งานคุณสมบัติ ido-mysql ของ Icinga2 ให้เลือก 'ใช่' แล้วกด ENTER

2. เข้าสู่ระบบเซิร์ฟเวอร์ฐานข้อมูล MariaDB ของคุณด้วยคำสั่ง 

        $ mysql -u root -p
    
จากนั้น สร้างฐานข้อมูลและผู้ใช้ฐานข้อมูลสำหรับแพ็คเกจ icinga2-ido-mysql และให้สิทธิ์ทั้งหมดบนฐานข้อมูลแก่ผู้ใช้

    > CREATE DATABASE [ชื่อฐานข้อมูล];
    > GRANT ALL ON [ชื่อฐานข้อมูล].* TO '[ตั้งชื่อผู้ใช้]'@'localhost' IDENTIFIED BY '[ตั้งรหัส]';
    > FLUSH PRIVILEGES;
    > EXIT;

3. เมื่อมีฐานข้อมูลแล้ว ให้ดำเนินการและนำเข้า Icinga2 IDO schema โดยใช้คำสั่ง คุณจะต้องระบุรหัสผ่านรูทของเซิร์ฟเวอร์ฐานข้อมูล

        $ mysql -u root -p [ชื่อฐานข้อมูล] < /usr/share/icinga2-ido-mysql/schema/mysql.sql

### ขั้นตอนที่ 3: เปิดใช้งาน Icinga2 IDO Module

ในการเปิดใช้งานการสื่อสารฐานข้อมูล icinga2-ido-mysql กับ Icinga Web 2 เราจำเป็นต้องดำเนินการอีกขั้นและทำการเปลี่ยนแปลงไฟล์การกำหนดค่าเริ่มต้น

1. เปิดไฟล์การกำหนดค่า icinga2-ido-mysql

        $ nano /etc/icinga2/features-available/ido-mysql.conf
แก้ไขรายการต่อไปนี้และตั้งค่าให้ตรงกับรายละเอียดฐานข้อมูล icinga2-ido-mysql ตามที่ระบุไว้ใน [ขั้นตอนที่ 4](#ขั้นตอนที่-4-ติดตั้ง-icinga2-ido-module)

user ="[ชื่อผู้ใช้]",
password="[รหัส]",
host = "localhost",
database = "[ชื่อฐานข้อมูล]"

2. จากนั้นเปิดใช้งานคุณสมบัติ icinga2-ido-mysql

        $ icinga2 feature enable ido-mysql

3. Restart icinga โดยใช้คำสั่ง 
        
        $ systemctl restart icinga2 

## ส่วน Icinga Web
จะเป็นส่วนสุดท้ายในการติดตั้งเพื่อแสดงข้อมูลการ Monitor Network ผ่าน Web UI ก็จะต้องมีการตั้งค่า API เพื่อให้เครื่องหลักที่มีการติดตั้ง Icinga สามารถติดต่อกับ Icinga Web ได้
### ขั้นตอนที่ 1: ติดตั้งและตั้งค่า IcingaWeb2

องค์ประกอบสุดท้ายในการติดตั้งและกำหนดค่าคือ IcingaWeb 2 ซึ่งเป็นเฟรมเวิร์ก PHP ที่รวดเร็ว ทรงพลัง และขยายได้ ซึ่งทำหน้าที่เป็นส่วนหน้าของ Icinga2

1. ติดตั้ง IcingaWeb2 และ Icinga CLI รันคำสั่ง

        $ apt install icingaweb2 icingacli -y

2. สร้างฐานข้อมูลที่สองที่จะกำหนดสำหรับ Icinga Web 2 อีกครั้ง เข้าสู่ระบบเซิร์ฟเวอร์ฐานข้อมูลของคุณ

$ mysql -u root -p

จากนั้นสร้างฐานข้อมูลและผู้ใช้ฐานข้อมูลสำหรับ Icingaweb2 และให้สิทธิ์ทั้งหมดแก่ผู้ใช้ฐานข้อมูลบนฐานข้อมูล

        > CREATE DATABASE icingaweb2;
        > GRANT ALL ON icingaweb2.* TO 'icingaweb2user'@'localhost' IDENTIFIED BY '[ตั้งรหัส]';
        > FLUSH PRIVILEGES;
        > EXIT;

3. ให้สร้างโทเค็นการตั้งค่าโดยใช้คำสั่งต่อไปนี้ โทเค็นการตั้งค่าจะใช้ระหว่างการตรวจสอบสิทธิ์เมื่อตั้งค่า Icinga2 บนเบราว์เซอร์

        $ icingacli setup token create;

    ในกรณีที่คุณทำหายหรือลืมโทเค็น คุณสามารถดูได้โดยรันคำสั่ง

        $ icingacli setup token show;
<p align="center"> <img src="Screenshots\012450.png"alt="Paris"></p>

ผลที่ได้ Token ของผมจะเป็น "dd5c38558c9afa2f"


### ขั้นตอนที่ 2: เปิดใช้งาน VPN

เราจะเปิดใช้งาน VPN เพื่อให้เครื่อง Host เราสามารถเชื่อมต่อกับ Server ใน Promox เพื่อนำ IP ของ Container ที่เราสร้างมาเข้าหน้าเว็ป Icingaweb2 ได้ 
1. โหลดโปรแกรมที่ให้บริการ VPN ในที่นี้ผมจะใช้ [OpenVPN](https://openvpn.net/vpn-client/) 

2. นำเข้า File .ovpn ที่เป็นของ Server ใน Proxmox เข้ามาใน OpenVPN

<p align="center"> <img src="Screenshots\214619.png"alt="Paris"></p>

3. จากนั้นก็เปิดการทำงาน
<p align="center"> <img src="Screenshots\215325.png"alt="Paris"></p>
    

### ขั้นตอนที่ 3: ตั้งค่าในหน้าเว็ป Icingaweb2 และเปิดใช้งาน
1. เปิดหน้าตั้งค่าเว็ป Icingaweb2 จะใช้ 

        http://เลข IP ของ Container/icingaweb2/setup

สำหรับผมคือ http://172.31.0.211/icingaweb2/setup 

2. เมื่อเข้ามาแล้วนำ Token ที่ได้มาจาก [ขั้นตอนที่6](#ขั้นตอนที่-6-ติดตั้งและตั้งค่า-icingaweb2) มากรอกลงช่ิอง
<p align="center"> <img src="Screenshots\(82).png"alt="Paris"></p>

3. ต่อมาให้ติ๊กเลือก Module ไหนก็ได้ที่ต้องการใช้ แต่ที่ต้องใช้คือ Monitoring 
<p align="center"> <img src="Screenshots\(83).png"alt="Paris"></p>

4. ต่อมาหน้านี้ให้ตรวจสอบสิ่งต่างๆที่ต้องมีในการเปิดใช้ Icingaweb2 ว่ามีครบไหมถ้าครบจะเป็นสีเขียวทุกช่อง

<p align="center"> <img src="Screenshots\(84).png"alt="Paris"></p>

5. ต่อมาหน้านี้ให้เรากรอกข้อมูลของฐานข้อมูลที่เราสร้างใน [ขั้นตอนที่ 6](#ขั้นตอนที่-6-ติดตั้งและตั้งค่า-icingaweb2)

<p align="center"> <img src="Screenshots\(85).png"alt="Paris"></p>

6. ต่อมาหน้านี้ให้เราตั้ง Username และ Password เพื่อใช้ในการ Login เข้า Icingaweb2

<p align="center"> <img src="Screenshots\(86).png"alt="Paris"></p>

7. ต่อมาหน้านี้ให้เรากรอกข้อมูลของฐานข้อมูลที่เราสร้างใน [ขั้นตอนที่ 4](#ขั้นตอนที่-4-ติดตั้ง-icinga2-ido-module) 

<p align="center"> <img src="Screenshots\(87).png"alt="Paris"></p>

8. ต่อมาหน้านี้ให้เรากรอกข้อมูล API ของ  Container ซึ่งจะเข้าไปดูได้โดย 

        $ nano /etc/icinga2/conf.d/api-users.conf 

<p align="center"> <img src="Screenshots\011003.png"alt="Paris"></p>

จากนั้นนำข้อมูลมากรอกให้ตรง

<p align="center"> <img src="Screenshots\(88).png"alt="Paris"></p>

9. หน้านี้ถ้าทุกอย่างไม่มีอะไรผิดพลาดจะสามารถกดปุ่ม Login to Icinga Web2 ได้

<p align="center"> <img src="Screenshots\(89).png"alt="Paris"></p>

และจะได้สามารถเข้า Dashboard ของ Icingaweb2 ได้โดยเข้าจาก 
        http://เลข IP ของ Container/icingaweb2/setup

สำหรับผมคือ http://172.31.0.211/icingaweb2/ ได้เลย
<p align="center"> <img src="Screenshots\(81).png"alt="Paris"></p>

# การใช้งาน Icingaweb2
[top⬆️](#สารบัญ)

## การจัดการ Dashboard
1. จากหน้าหลักให้กดปุ่ม Drop Down ตามรูป เมื่อกดไปจะมีปุ่ม Add Dashlet
<p align="center"> <img src="Screenshots\().png"alt="Paris"></p>

2. จากนั้นจะได้หน้าแบบนี้ 

<p align="center"> <img src="Screenshots\2023-01-02 220744.png"alt="Paris"></p>

- ในช่อง Url ให้กรอก Url ของ Dashlet ที่เราต้องการเช่น หน้าแสดง Host ที่ได้ url จากในหน้านี้ 
<p align="center"> <img src="Screenshots\ภาพหน้าจอ (103).png"alt="Paris"></p> 

- ในช่อง Dashlet Title ให้ตั้งชื่อ Dashlet ที่จะให้แสดงใน Dashboard
- ในช่อง Dropdown ของ Dashboard จะให้เราเลือก หน้าแถบ Dashboard ที่เราต้องการจะให้ไปแสดง ถ้าอยากสร้างใหม่เองให้ ติ๊กปุ่ม New dashboard 

3. ผลที่ได้ก็จะได้ Dashboard ใหม่
<p align="center"> <img src="Screenshots\ภาพหน้าจอ (102).png"alt="Paris"></p> 

## การสร้าง Host
การสร้าง Host เพื่อจะ Monitor IP เป้าหมาย โดยวิธีดังนี้
1. เข้าไปแก้ไขใน File โดยใช้คำสั่งนี้
        
        $ nano /etc/icinga2/conf.d/hosts.conf
2. เมื่อเข้ามาแล้วให้เขียน Scripts เพื่อสร้าง Object ในการสร้าง Host ตามนี้
        
        object Host "ชื่อHostที่จะตั้ง"{
                import "Templateที่ต้องการ"
                address = "ที่อยู่IPที่ต้องการ"
                check_command = "คำสั่งในการตรวจสอบสถานะ"
        }
        
หรือในรูปตัวอย่างนี้
<p align="center"> <img src="Screenshots\2023-01-06 210257.png"alt="Paris"></p> 

3. Restart icinga โดยใช้คำสั่ง 
        
        $ systemctl restart icinga2 

4. ผลที่ได้ดังรูปนี้
<p align="center"> <img src="Screenshots\ภาพหน้าจอ (103).png"alt="Paris"></p> 

## การใช้ SNMP ในการ Monitor
1. สร้าง Service ใหม่ เข้าไปแก้ไขใน File โดยใช้คำสั่งนี้

        $ nano /etc/icinga2/conf.d/services.conf
    แล้วใช้ script นี้ในการสร้าง service

        apply Service "uptime" {
        import "generic-service"

        check_command = "snmp"
        vars.snmp_oid = "1.3.6.1.2.1.1.3.0"
        vars.snmp_miblist = "DISMAN-EVENT-MIB"

        assign where host.vars.agent_type == "snmp" && host.vars.snmp_community != ""
        }
        

2. สร้าง Template ใหม่ เข้าไปแก้ไขใน File โดยใช้คำสั่งนี้

        $ nano /etc/icinga2/conf.d/templates.conf
   แล้วใช้ script นี้ในการสร้าง Template

        template Host "snmp-agent" {
                check_command = "hostalive"

                vars.agent_type = "snmp"

                vars.snmp_community = "public-icinga"
                }        

3. เข้าไปใน Container ที่จะ Monitor ให้ติดตั้ง Package SNMP ใช้คำสั่งนี้

        $ apt-get install snmp
        
   จากนั้นให้เข้าไปตั้งค่าในไฟล์ snmpd ด้วยคำสั่งนี้
   
        $ nano /etc/snmp/snmpd.conf
        
   จากนั้นในไฟล์ให้แก้ Script 
   
        agentaddress เลขIPเครื่อง, [::1]  
        
        
    และ 
        
        
        rocommunity ชื่อCommunity 
        
        
    ดังรูปตัวอย่าง
        
    <p align="center"> <img src="Screenshots\สกรีนช็อต 2023-01-08 172418.png"alt="Paris"></p> 
        
        
    จากนั้น Restart snmpd ด้วยคำสั่งนี้
        
        $ systemctl restart snmpd

4. ต่อมาในเครื่องหลักให้สร้าง Host ที่จะใช้ SNMP ให้ import Template snmp-agent เช่น
        
        object Host "ชื่อHostที่จะตั้ง"{
                import "snmp-agent"
                address = "ที่อยู่IPที่ต้องการ"
                check_command = "คำสั่งในการตรวจสอบสถานะ"
        } 

5. ผลที่ได้ก็จะมีส่วน Custom Variables ที่มี Parameter agent_type กับ snmp_community ที่เพิ่มเข้ามาตามที่เราสร้าง Template 
<p align="center"> <img src="Screenshots\2023.png"alt="Paris"></p>

และมี service uptime ที่เพิ่มเข้ามาตามที่เราสร้าง Service ไว้และมีการตรวจสอบสถานะของ SNMP

<p align="center"> <img src="Screenshots\2023-01-06 235546.png"alt="Paris"></p>

# การอ่านผลลัพธ์ Icingaweb2
[top⬆️](#สารบัญ)

การอ่าน Monitor ของ Host หรือ Service จะอ่านสถานะของ Host หรือ Service ได้ในลักษณะนี้
<p align="center"> <img src="Screenshots\2023-01-06 235546.png"alt="Paris"></p>

ซึ่งแต่ล่ะสถานะก็จะมีความหมาย ดังนี้

<table style="width:100%">
<tr>
    <th colspan="3"><p align="center"> สถานะของ Host </p></th>
</tr>
  <tr>
    <th>สถานะ</th>
    <th>คำอธิบาย</th>
    <th>สี</th>
  </tr>
  <tr>
    <td>UP</td>
    <td>Host ยังทำงานอยู่</td>
        <td style="color:lightgreen;">เขียว</td>
  </tr>
  <tr>
    <td>DOWN</td>
    <td>Host ไม่ทำงาน</td>
    <td style="color:red;">แดง</td>
  </tr>
</table>
<table style="width:100%">
<tr>
    <th colspan="3"><p align="center"> สถานะของ Service </p></th>
</tr>
  <tr>
    <th>สถานะ</th>
    <th>คำอธิบาย</th>
        <th>สี</th>
  </tr>
  <tr>
    <td>OK</td>
    <td>Service ไม่มีปัญหา</td>
    <td style="color:lightgreen;">เขียว</td>
  </tr>
  <tr>
    <td>WARNING</td>
    <td>Service กำลังมีปัญหา</td>
    <td style="color:orange;">ส้ม</td>
  </tr>
  <tr>
    <td>CRITICAL</td>
    <td>Service เกิดความเสียหาย</td>
    <td style="color:red;">แดง</td>
  </tr>
  <tr>
    <td>UNKNOWN</td>
    <td>ไม่สามารถระบุ Service</td>
    <td style="color:#D462FF;">ม่วง</td>
  </tr>
</table>

ส่วนรายละเอียดการ Monitor ของ Host ก็จะมีส่วนที่สำคัญในการดู ดังนี้

-  Plugin Output และ Performance data

    เป็นส่วนที่แสดงเรื่องของการ Ping IP ของ Host ถ้าการ Ping มีปัญหาก็จะส่งผลถึงการแสดงสถานะของ Host หรือ Service

    <p align="center"> <img src="Screenshots\2025.png"alt="Paris"></p>

- Check execution

  เป็นส่วนแสดงการตรวจสถานะของ Host ที่จะมีการบอกเวลาในการตรวจสอบสถานะครั้งล่าสุด และปัจจุบัน ใน Parameter Last check และ Next check และการบอกถึงสถานะการเข้าถึงโดย Parameter Reachable ถ้า Host มีปัญหาจุดนี้ก็จะแจ้งเตือนด้วย
  
ref: https://icinga.com/docs/icinga-2/latest/doc/01-about/

----

ผู้จัดทำ : 
- นาย พีรพัฒน์ สาริมาน  https://github.com/phisic1714
        
- นาย กันต์ศักดิ์ ไชยมงคล https://github.com/NightingX9
        
- นางสาว พิชญ์กัญ วงศ์ธนะเชษฐ https://github.com/pitkann

- นางสาว ศรัณยา ทะลาสี https://github.com/ploysry




