---
title: SonarScanner CLI - คู่มือการติดตั้งบน Windows 11
tags:
  - sonarqube
  - sonarscanner
  - windows
  - devops
  - คู่มือ
created: 2026-06-27
updated: 2026-06-29
---

# SonarScanner CLI - คู่มือการติดตั้งบน Windows 11

> [!summary] ใช้เมื่อไร
> ใช้คู่มือนี้เมื่อต้องติดตั้ง SonarScanner CLI บน Windows 11 เพื่อสแกนซอร์สโค้ดจากเครื่องผู้พัฒนาหรือเครื่อง build agent แล้วส่งผลไปยัง SonarQube Server หรือ SonarQube Cloud

## วัตถุประสงค์

เอกสารนี้ใช้สำหรับติดตั้ง SonarScanner CLI บน Windows 11 เพื่อใช้วิเคราะห์คุณภาพซอร์สโค้ดและส่งผลลัพธ์ไปยัง SonarQube Server หรือ SonarQube Cloud

## ขอบเขต

| ครอบคลุม | ไม่ครอบคลุม |
| --- | --- |
| การติดตั้ง SonarScanner CLI บน Windows 11 | การติดตั้ง SonarQube Server |
| การตั้งค่า environment variable สำหรับ scan | การออกแบบ quality gate หรือ quality profile |
| การสร้าง `sonar-project.properties` เบื้องต้น | การตั้งค่า CI/CD pipeline เต็มรูปแบบ |

## ข้อกำหนดเบื้องต้น

- เครื่องใช้งานเป็น Windows 11 แบบ 64-bit
- มีสิทธิ์แก้ไข Environment Variables ของผู้ใช้หรือเครื่อง
- มี URL ของ SonarQube Server หรือ SonarQube Cloud
- มี Token สำหรับใช้สแกนโปรเจกต์
- หากใช้ไฟล์แบบ `Any` ต้องมี Java Runtime ตามเวอร์ชันที่ SonarSource กำหนด
- หากเป็นเครื่ององค์กร ต้องดาวน์โหลดจากแหล่งที่ได้รับอนุมัติ และตรวจสอบ checksum ตามนโยบายความปลอดภัย

## ข้อมูลที่ต้องเตรียม

| รายการ | ตัวอย่าง | หมายเหตุ |
| --- | --- | --- |
| SonarQube URL | `https://sonarqube.example.org` | หลีกเลี่ยง URL ตัวอย่างใน production |
| Token | `<your-token>` | เก็บใน secret manager หรือ environment variable |
| Project Key | `my-project` | ต้องตรงกับ project ใน SonarQube หากมีอยู่แล้ว |
| Source Folder | `.` | ระบุ root ของ source code |
| ตำแหน่งติดตั้ง | `C:\Tools\sonar-scanner` | ใช้ path ที่ผู้ดูแลระบบอนุมัติ |

## ขั้นตอนการติดตั้ง

1. เปิดหน้าเอกสารทางการของ SonarScanner CLI

   https://docs.sonarsource.com/sonarqube-server/analyzing-source-code/scanners/sonarscanner

2. ดาวน์โหลดไฟล์สำหรับ `Windows x64`

3. แตกไฟล์ `.zip` ไปยังโฟลเดอร์ที่ต้องการ เช่น

```text
C:\Tools\sonar-scanner
```

4. เพิ่ม path ของโฟลเดอร์ `bin` เข้า Environment Variable ชื่อ `Path`

```text
C:\Tools\sonar-scanner\bin
```

5. เปิด PowerShell หรือ Command Prompt ใหม่ แล้วตรวจสอบการติดตั้ง

```powershell
sonar-scanner.bat -h
sonar-scanner.bat -v
```

หากติดตั้งถูกต้อง ระบบจะแสดงคำสั่งการใช้งานและเวอร์ชันของ SonarScanner CLI

> [!tip] สำหรับเครื่ององค์กร
> หากติดตั้งให้หลายเครื่อง ควรจัดทำ package หรือ script ติดตั้งมาตรฐาน และกำหนด version ที่ผ่านการทดสอบแล้วแทนการให้แต่ละเครื่องดาวน์โหลดเอง

## การตั้งค่า Token และ Server URL

แนะนำให้เก็บ Token ใน Environment Variable แทนการบันทึกลงไฟล์โปรเจกต์

ตั้งค่าเฉพาะ session ปัจจุบัน

```powershell
$env:SONAR_HOST_URL="http://localhost:9000"
$env:SONAR_TOKEN="<your-token>"
```

ตั้งค่าแบบถาวรสำหรับผู้ใช้ปัจจุบัน

```powershell
setx SONAR_HOST_URL "http://localhost:9000"
setx SONAR_TOKEN "<your-token>"
```

หลังใช้ `setx` ให้ปิดและเปิด PowerShell ใหม่ก่อนใช้งาน

## ตัวอย่างไฟล์ sonar-project.properties

สร้างไฟล์ `sonar-project.properties` ไว้ที่ root folder ของโปรเจกต์

```properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.sources=.
```

ไม่ควรบันทึก Token ลงในไฟล์นี้

ตัวอย่างสำหรับโปรเจกต์ที่ต้องระบุ encoding:

```properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.sources=.
sonar.sourceEncoding=UTF-8
```

## การสแกนโปรเจกต์

เปิด PowerShell ที่ root folder ของโปรเจกต์ แล้วรันคำสั่ง

```powershell
sonar-scanner.bat
```

หากต้องการระบุค่าโดยตรงในคำสั่ง สามารถใช้รูปแบบดังนี้

```powershell
sonar-scanner.bat -D"sonar.host.url=http://localhost:9000" -D"sonar.token=<your-token>"
```

## คำสั่งตรวจสอบปัญหา

กรณีต้องการดูรายละเอียดเพิ่มเติมระหว่างสแกน ให้ใช้คำสั่ง

```powershell
sonar-scanner.bat -X
```

หรือ

```powershell
sonar-scanner.bat -D"sonar.verbose=true"
```

## การตรวจสอบหลังติดตั้ง

| รายการตรวจสอบ | ผลที่คาดหวัง |
| --- | --- |
| เรียก `sonar-scanner.bat -v` ได้ | แสดง version ของ scanner |
| Path ถูกต้อง | เปิด PowerShell ใหม่แล้วยังเรียกคำสั่งได้ |
| Token ไม่อยู่ในไฟล์โปรเจกต์ | ไม่พบ token ใน `sonar-project.properties` |
| Scan สำเร็จ | Log แสดงผลว่า `EXECUTION SUCCESS` |
| SonarQube แสดงผลล่าสุด | Project มี analysis timestamp ใหม่ |

## แนวทางแก้ปัญหาที่พบบ่อย

| อาการ | สาเหตุที่พบบ่อย | แนวทางตรวจสอบ |
| --- | --- | --- |
| `sonar-scanner.bat` ไม่ถูกพบ | ยังไม่ได้เพิ่ม path หรือยังไม่ได้เปิด terminal ใหม่ | ตรวจ `Path` และเปิด PowerShell ใหม่ |
| Authentication failed | Token ไม่ถูกต้องหรือหมดอายุ | สร้าง token ใหม่และตั้งค่า environment variable อีกครั้ง |
| เชื่อมต่อ server ไม่ได้ | URL, proxy, firewall หรือ certificate มีปัญหา | ทดสอบเปิด URL และตรวจ proxy/certificate |
| Scanner หา Java ไม่พบ | ใช้ package ที่ต้องพึ่ง Java Runtime | ติดตั้ง Java ตาม version ที่รองรับ หรือใช้ package ที่ bundled JRE |
| ผล scan ไม่พบ source | `sonar.sources` ไม่ตรงกับโครงสร้างโปรเจกต์ | ตรวจ path และรันจาก root folder ของโปรเจกต์ |

## Checklist สรุปผล

| รายการตรวจสอบ | สถานะ | หมายเหตุ |
| --- | --- | --- |
| ดาวน์โหลดจากแหล่งทางการหรือแหล่งที่องค์กรอนุมัติแล้ว |  |  |
| ติดตั้งและเพิ่ม `Path` แล้ว |  |  |
| ตรวจสอบ version สำเร็จ |  |  |
| ตั้งค่า URL และ Token แบบไม่เปิดเผยข้อมูลลับ |  |  |
| สร้าง `sonar-project.properties` แล้ว |  |  |
| ทดสอบ scan สำเร็จ |  |  |
| ตรวจผลบน SonarQube แล้ว |  |  |

## หมายเหตุสำคัญ

- SonarScanner CLI เหมาะสำหรับโปรเจกต์ที่ไม่มี scanner เฉพาะของ build system
- โปรเจกต์ .NET ควรพิจารณาใช้ SonarScanner for .NET
- ไม่ควรเปิดเผย Token ในเอกสาร รูปภาพ log หรือ repository
- หากใช้งานในองค์กร ควรดาวน์โหลดจากแหล่งทางการและตรวจสอบเวอร์ชันตามนโยบายความปลอดภัย
- หากเครื่องมี antivirus หรือ endpoint security อาจต้องตรวจสอบผลกระทบต่อการสแกนตามคำแนะนำของ SonarSource

## แหล่งอ้างอิง

- SonarSource Documentation: SonarScanner CLI  
  https://docs.sonarsource.com/sonarqube-server/analyzing-source-code/scanners/sonarscanner
- SonarSource GitHub: sonar-scanner-cli  
  https://github.com/SonarSource/sonar-scanner-cli
