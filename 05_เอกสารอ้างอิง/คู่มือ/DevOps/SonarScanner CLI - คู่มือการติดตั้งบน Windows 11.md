---
title: SonarScanner CLI - คู่มือการติดตั้งบน Windows 11
tags:
  - sonarqube
  - sonarscanner
  - windows
  - devops
  - คู่มือ
created: 2026-06-27
updated: 2026-06-27
---

# SonarScanner CLI - คู่มือการติดตั้งบน Windows 11

## วัตถุประสงค์

เอกสารนี้ใช้สำหรับติดตั้ง SonarScanner CLI บน Windows 11 เพื่อใช้วิเคราะห์คุณภาพซอร์สโค้ดและส่งผลลัพธ์ไปยัง SonarQube Server หรือ SonarQube Cloud

## ข้อกำหนดเบื้องต้น

- เครื่องใช้งานเป็น Windows 11 แบบ 64-bit
- มีสิทธิ์แก้ไข Environment Variables ของผู้ใช้หรือเครื่อง
- มี URL ของ SonarQube Server หรือ SonarQube Cloud
- มี Token สำหรับใช้สแกนโปรเจกต์
- หากใช้ไฟล์แบบ `Any` ต้องมี Java Runtime ตามเวอร์ชันที่ SonarSource กำหนด

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
