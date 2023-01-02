# SPCN-NMS
<p align="center"><a href="https://pve22.ipv9.me/#v1:0:18:4:::::::"> <img src="Screenshots/logo.png"width=300 alt="Paris"></a><a href="http://172.31.0.211/icingaweb2"> <img src="Screenshots\Icinga_logo.svg"width=300 alt="Paris"></a></p>

# <p align="center">การใช้งาน Icinga ผ่าน Container ใน Proxmox</p>

|<p align="center"> สารบัญ </p>|
| ------------- |
|[การติดตั้ง Icinga](#สร้าง-master-vm-ubuntu-2204)|
|[สร้าง vm จาก os ตัวอื่นๆ](#สร้าง-vm-จาก-os-ตัวอื่นๆ)|
|[สร้าง container template (เลือกจาก CT list)](#สร้าง-container-template-เลือกจาก-ct-list)|
|<DL>[การตั้งค่าต่างๆใน vm ](#การตั้งค่าต่างๆใน-vm)<DT>[\|-การเปิดใช้ qemu guest agent](#การเปิดใช้-qemu-guest-agent)<DT><DT>[ \|-การตั้ง local time update](#การตั้ง-local-time-update)</DT><DT>[\|-การ reset machineid ไม่ให้ ip ชนกัน กับ vm ตัวอื่น](#การ-reset-machineid-ไม่ให้-ip-ชนกัน-กับ-vm-ตัวอื่น)<DT></DL>|
