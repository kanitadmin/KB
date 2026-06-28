---
title: Windows Server - แผนและขั้นตอนการอัปเดตระบบ
tags:
  - windows-server
  - patch-management
  - update
  - infrastructure
  - คู่มือ
created: 2026-06-28
updated: 2026-06-28
---

# Windows Server - แผนและขั้นตอนการอัปเดตระบบ

## วัตถุประสงค์

เอกสารนี้ใช้เป็นแนวทางการวางแผนและดำเนินการอัปเดต Windows Server เพื่อปิดช่องโหว่ด้านความปลอดภัย แก้ไขปัญหาของระบบ และลดความเสี่ยงจากการหยุดชะงักของบริการ

## ขอบเขต

คู่มือนี้ครอบคลุมการติดตั้ง Windows Update, Security Update, Cumulative Update และ Update ที่ได้รับอนุมัติสำหรับ Windows Server

คู่มือนี้ไม่ครอบคลุมการอัปเกรดข้ามเวอร์ชันระบบปฏิบัติการ เช่น Windows Server 2019 เป็น Windows Server 2022 หรือการย้ายระบบไปยังเครื่องใหม่

## ข้อมูลระบบ

| รายการ | รายละเอียด |
| --- | --- |
| ชื่อเครื่อง |  |
| IP Address |  |
| Operating System |  |
| บทบาทของ Server |  |
| ระบบหรือ Application ที่เกี่ยวข้อง |  |
| วิธีอัปเดต | Windows Update / WSUS / Manual / อื่น ๆ |
| ผู้ดำเนินการ |  |
| วันที่และเวลาเริ่มดำเนินการ |  |
| วันที่และเวลาสิ้นสุด |  |

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## แผนการดำเนินงาน

| ลำดับ | กิจกรรม | ผู้รับผิดชอบ | ช่วงเวลา | ผลลัพธ์ที่ต้องการ |
| --- | --- | --- | --- | --- |
| 1 | ตรวจสอบรายการ Server ที่ต้องอัปเดต |  |  | ได้รายการ Server ครบถ้วน |
| 2 | ตรวจสอบผลกระทบต่อระบบงาน |  |  | ระบุ Downtime และผู้เกี่ยวข้อง |
| 3 | ขออนุมัติ Maintenance Window |  |  | ได้รับอนุมัติก่อนดำเนินการ |
| 4 | สำรองข้อมูลหรือทำ Snapshot |  |  | สามารถกู้คืนระบบได้ |
| 5 | ติดตั้ง Update |  |  | Update สำเร็จ |
| 6 | Restart Server หากจำเป็น |  |  | Server กลับมาทำงานปกติ |
| 7 | ตรวจสอบบริการหลังอัปเดต |  |  | Service และ Application ใช้งานได้ |
| 8 | บันทึกผลและปิดงาน |  |  | มีหลักฐานประกอบครบถ้วน |

## ข้อกำหนดก่อนดำเนินการ

- ได้รับอนุมัติ Maintenance Window จากผู้มีอำนาจ
- แจ้งผู้ใช้งานหรือเจ้าของระบบล่วงหน้าตามระยะเวลาที่กำหนด
- ตรวจสอบว่า Backup หรือ Snapshot สามารถใช้งานได้จริง
- มีบัญชีผู้ดูแลระบบที่มีสิทธิ์เพียงพอ
- ตรวจสอบพื้นที่ว่างของ Disk โดยเฉพาะ Drive `C:`
- ตรวจสอบว่า Server ไม่มีงานสำคัญกำลังประมวลผล
- เตรียมแผน Rollback และผู้ประสานงานกรณีเกิดเหตุขัดข้อง

## ขั้นตอนที่ 1 ตรวจสอบสถานะก่อนอัปเดต

### 1.1 ตรวจสอบข้อมูลระบบ

เปิด PowerShell ด้วยสิทธิ์ Administrator แล้วรันคำสั่ง

```powershell
hostname
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber, OsLastBootUpTime
```

### 1.2 ตรวจสอบพื้นที่ว่างของ Disk

```powershell
Get-PSDrive -PSProvider FileSystem
```

แนะนำให้ Drive `C:` มีพื้นที่ว่างเพียงพอก่อนดำเนินการ หากพื้นที่เหลือน้อยควรแก้ไขก่อนเริ่มติดตั้ง Update

### 1.3 ตรวจสอบ Update ที่ติดตั้งล่าสุด

```powershell
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
```

### 1.4 ตรวจสอบ Service สำคัญ

ตัวอย่างคำสั่งตรวจสอบ Service

```powershell
Get-Service | Where-Object {$_.Status -eq "Running"} | Sort-Object DisplayName
```

หากเป็น Server เฉพาะทาง เช่น Domain Controller, DNS, DHCP, File Server, Web Server หรือ Database Server ให้ตรวจสอบ Service ของระบบนั้นเพิ่มเติม

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 2 สำรองข้อมูลและเตรียม Rollback

### 2.1 สำรองข้อมูล

ให้เลือกวิธีสำรองข้อมูลตามมาตรฐานขององค์กร เช่น

- VM Snapshot
- Backup ระดับระบบปฏิบัติการ
- Backup ระดับ Application
- Backup Database
- Export Configuration ของระบบสำคัญ

### 2.2 ตรวจสอบผล Backup

ต้องตรวจสอบให้แน่ใจว่า Backup สำเร็จและสามารถนำมาใช้กู้คืนได้

### 2.3 บันทึกข้อมูลก่อนอัปเดต

บันทึกข้อมูลอย่างน้อยดังนี้

- Version และ Build ของ Windows Server
- รายการ Update ล่าสุด
- สถานะ Service สำคัญ
- สถานะ Application
- ขนาดพื้นที่ว่างของ Disk

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 3 ตรวจสอบและติดตั้ง Update

เลือกวิธีดำเนินการตามรูปแบบที่องค์กรใช้งาน

### วิธีที่ 1 อัปเดตผ่าน Windows Update

เหมาะสำหรับ Server ที่อนุญาตให้เชื่อมต่อ Windows Update โดยตรง

1. เปิด `Settings`
2. ไปที่ `Windows Update`
3. เลือก `Check for updates`
4. ตรวจสอบรายการ Update ที่พบ
5. เลือกติดตั้ง Update ตามที่ได้รับอนุมัติ
6. Restart Server หากระบบร้องขอ

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



### วิธีที่ 2 อัปเดตผ่าน WSUS หรือระบบบริหารจัดการ Patch

เหมาะสำหรับองค์กรที่ควบคุม Patch ผ่านศูนย์กลาง

1. ตรวจสอบว่า Server อยู่ในกลุ่มเป้าหมายที่ถูกต้อง
2. ตรวจสอบว่า Update ได้รับการอนุมัติแล้ว
3. สั่ง Scan หรือรอให้ Server ตรวจสอบ Update ตามรอบเวลา
4. ติดตั้ง Update ตาม Maintenance Window
5. Restart Server หากระบบร้องขอ
6. ตรวจสอบสถานะใน Console ของระบบบริหารจัดการ Patch

### วิธีที่ 3 อัปเดตผ่าน Server Core หรือ SConfig

เหมาะสำหรับ Windows Server Core

1. เข้าสู่ระบบด้วยบัญชีผู้ดูแลระบบ
2. เปิด `sconfig`
3. เลือกเมนู `Windows Update`
4. เลือกค้นหา Update
5. เลือกติดตั้ง Update ที่ได้รับอนุมัติ
6. Restart Server หากระบบร้องขอ

### วิธีที่ 4 ติดตั้ง Update แบบ Manual

เหมาะสำหรับกรณีที่ต้องติดตั้งไฟล์ Update เฉพาะรายการ เช่น `.msu` หรือ `.cab`

ตัวอย่างการติดตั้งไฟล์ `.msu`

```powershell
wusa.exe C:\Patch\WindowsUpdate.msu /quiet /norestart
```

ตัวอย่างการติดตั้งไฟล์ `.cab`

```powershell
dism.exe /Online /Add-Package /PackagePath:C:\Patch\WindowsUpdate.cab
```

หลังติดตั้งเสร็จให้ Restart Server หากจำเป็น

```powershell
Restart-Computer
```

## ขั้นตอนที่ 4 Restart Server

หากมีการร้องขอ Restart ให้ดำเนินการภายใน Maintenance Window เท่านั้น

ก่อน Restart ให้ตรวจสอบว่าไม่มีผู้ใช้งานหรือ Process สำคัญกำลังใช้งานอยู่

```powershell
shutdown /r /t 60 /c "Windows Server update maintenance"
```

หลัง Restart ให้รอจนระบบกลับมา Online และสามารถ Remote เข้าใช้งานได้

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 5 ตรวจสอบหลังอัปเดต

### 5.1 ตรวจสอบ Version และ Update ที่ติดตั้ง

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber, OsLastBootUpTime
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
```

### 5.2 ตรวจสอบ Event Log

```powershell
Get-WinEvent -LogName System -MaxEvents 50 | Select-Object TimeCreated, LevelDisplayName, ProviderName, Id, Message
Get-WinEvent -LogName Application -MaxEvents 50 | Select-Object TimeCreated, LevelDisplayName, ProviderName, Id, Message
```

### 5.3 ตรวจสอบ Service สำคัญ

```powershell
Get-Service | Where-Object {$_.StartType -eq "Automatic" -and $_.Status -ne "Running"}
```

หากพบ Service ที่ควรทำงานแต่ไม่ทำงาน ให้ตรวจสอบ Log และ Start Service ตามขั้นตอนของระบบนั้น

### 5.4 ตรวจสอบ Application

ตรวจสอบร่วมกับเจ้าของระบบหรือผู้ดูแล Application อย่างน้อยดังนี้

- Login เข้า Application ได้
- Function สำคัญทำงานได้
- ระบบเชื่อมต่อ Database หรือระบบปลายทางได้
- ไม่มี Error ใหม่ใน Log
- Monitoring แสดงสถานะปกติ

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 6 บันทึกผลและปิดงาน

บันทึกข้อมูลหลังดำเนินการอย่างน้อยดังนี้

- วันที่และเวลาที่เริ่มและสิ้นสุดงาน
- รายการ Update หรือ KB ที่ติดตั้ง
- ผลการ Restart
- ผลการตรวจสอบ Service และ Application
- ปัญหาที่พบและวิธีแก้ไข
- ผู้ตรวจสอบและผู้อนุมัติปิดงาน

## Checklist สรุปผล

| รายการตรวจสอบ | สถานะ | หมายเหตุ |
| --- | --- | --- |
| ได้รับอนุมัติ Maintenance Window แล้ว |  |  |
| แจ้งผู้เกี่ยวข้องแล้ว |  |  |
| ตรวจสอบสถานะก่อนอัปเดตแล้ว |  |  |
| สำรองข้อมูลหรือทำ Snapshot แล้ว |  |  |
| ตรวจสอบพื้นที่ว่างของ Disk แล้ว |  |  |
| ติดตั้ง Update สำเร็จ |  |  |
| Restart Server สำเร็จ |  |  |
| ตรวจสอบ Update ที่ติดตั้งแล้ว |  |  |
| Service สำคัญทำงานปกติ |  |  |
| Application ใช้งานได้ปกติ |  |  |
| Monitoring แสดงสถานะปกติ |  |  |
| บันทึกหลักฐานและปิดงานแล้ว |  |  |

## แนวทาง Rollback

หากพบปัญหาหลังอัปเดต ให้ดำเนินการตามลำดับดังนี้

1. ประเมินผลกระทบและแจ้งผู้เกี่ยวข้องทันที
2. ตรวจสอบ Event Log และสถานะ Service ที่เกี่ยวข้อง
3. หากเป็นปัญหา Service ให้พยายาม Start หรือแก้ไข Configuration ตามสาเหตุ
4. หากเกิดจาก Update และ Update นั้นรองรับการถอนการติดตั้ง ให้พิจารณาถอน Update
5. หากระบบไม่สามารถให้บริการได้ตามเวลาที่กำหนด ให้กู้คืนจาก Backup หรือ Snapshot ที่เตรียมไว้

ตัวอย่างการถอน Update เฉพาะกรณีที่ Update รองรับการถอนการติดตั้ง

```powershell
wusa.exe /uninstall /kb:<KB_NUMBER> /quiet /norestart
Restart-Computer
```

หมายเหตุ: Update บางประเภทอาจไม่สามารถถอนการติดตั้งได้ หรืออาจต้องใช้วิธีกู้คืนระบบจาก Backup หรือ Snapshot แทน

## แหล่งอ้างอิง

- Microsoft Learn: Manage Windows Server updates  
  https://learn.microsoft.com/windows-server/get-started/manage-windows-server-updates
- Microsoft Learn: Server configuration tool  
  https://learn.microsoft.com/windows-server/administration/server-core/server-core-sconfig
- Microsoft Learn: DISM Operating System Package Servicing Command-Line Options  
  https://learn.microsoft.com/windows-hardware/manufacture/desktop/dism-operating-system-package-servicing-command-line-options
- Microsoft Learn: Windows Update Standalone Installer  
  https://learn.microsoft.com/windows/win32/wua_sdk/windows-update-standalone-installer
