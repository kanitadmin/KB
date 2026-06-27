---
title: SonarLint - คู่มือการติดตั้งบน VS Code
tags:
  - sonarlint
  - sonarqube
  - vscode
  - devops
  - คู่มือ
created: 2026-06-27
updated: 2026-06-27
---

# SonarLint - คู่มือการติดตั้งบน VS Code

## วัตถุประสงค์

เอกสารนี้ใช้สำหรับติดตั้ง SonarLint บน Visual Studio Code เพื่อช่วยตรวจสอบคุณภาพโค้ด ความปลอดภัย และปัญหาที่ควรแก้ไขระหว่างการพัฒนา

> หมายเหตุ: SonarLint for VS Code ถูกเปลี่ยนชื่อเป็น SonarQube for IDE / SonarQube for VS Code ในเอกสารรุ่นใหม่ แต่ extension id ยังใช้ `SonarSource.sonarlint-vscode`

## ข้อกำหนดเบื้องต้น

- ติดตั้ง Visual Studio Code แล้ว
- เครื่องสามารถเชื่อมต่ออินเทอร์เน็ตเพื่อดาวน์โหลด extension ได้
- หากต้องการเชื่อมกับ SonarQube Server หรือ SonarQube Cloud ต้องมี URL และ Token ที่ถูกต้อง

## วิธีติดตั้งผ่าน VS Code

1. เปิด Visual Studio Code
2. ไปที่เมนู Extensions
3. ค้นหา `SonarQube for IDE` หรือ `SonarLint`
4. เลือก extension จาก SonarSource
5. กด Install
6. ปิดและเปิด VS Code ใหม่ หากระบบร้องขอ

## วิธีติดตั้งผ่าน Command Line

เปิด PowerShell หรือ Command Prompt แล้วรันคำสั่ง

```powershell
code --install-extension SonarSource.sonarlint-vscode
```

ตรวจสอบ extension ที่ติดตั้งแล้ว

```powershell
code --list-extensions | findstr sonarlint
```

## การใช้งานเบื้องต้น

1. เปิดโฟลเดอร์โปรเจกต์ใน VS Code
2. เปิดไฟล์ source code ที่ต้องการตรวจสอบ
3. รอให้ extension วิเคราะห์โค้ดอัตโนมัติ
4. ตรวจสอบปัญหาที่แสดงใน Editor, Problems panel หรือ SonarQube panel
5. แก้ไขปัญหาตามคำแนะนำที่แสดงใน VS Code

## การเชื่อมต่อกับ SonarQube Server หรือ SonarQube Cloud

ใช้ Connected Mode เมื่อต้องการให้ VS Code ใช้ rule และ quality profile เดียวกับระบบ SonarQube ขององค์กร

1. เปิดแถบ SonarQube Setup ใน VS Code
2. เลือก Connected Mode
3. เลือก Add SonarQube Server Connection หรือ Add SonarQube Cloud Connection
4. ระบุ URL และ Token
5. เลือก project ที่ต้องการเชื่อมต่อ
6. ตรวจสอบว่า VS Code แสดงสถานะเชื่อมต่อสำเร็จ

## การตรวจสอบปัญหาเบื้องต้น

หาก extension ไม่ทำงาน ให้ตรวจสอบรายการต่อไปนี้

- ตรวจสอบว่า extension ติดตั้งและเปิดใช้งานแล้ว
- เปิดโฟลเดอร์โปรเจกต์ ไม่ใช่เปิดเฉพาะไฟล์เดี่ยว
- ตรวจสอบภาษาโปรแกรมที่ extension รองรับ
- ตรวจสอบ internet connection หากต้องดาวน์โหลด analyzer เพิ่มเติม
- ตรวจสอบ Token และ URL กรณีใช้ Connected Mode
- เปิด Output panel แล้วเลือก SonarQube เพื่อตรวจสอบ log

## หมายเหตุสำคัญ

- ไม่ควรบันทึก Token ลงใน repository หรือเอกสารที่แชร์สาธารณะ
- ควรติดตั้ง extension จาก SonarSource เท่านั้น
- หากองค์กรมี SonarQube Server ควรใช้ Connected Mode เพื่อให้ผลตรวจสอดคล้องกับนโยบายขององค์กร
- หากใช้โปรเจกต์ .NET ควรตรวจสอบแนวทางของ SonarQube ร่วมกับ build pipeline เพิ่มเติม

## แหล่งอ้างอิง

- SonarSource Documentation: Installation  
  https://docs.sonarsource.com/sonarqube-for-vs-code/getting-started/installation
- SonarSource Documentation: Connected Mode Setup  
  https://docs.sonarsource.com/sonarqube-for-vs-code/connect-your-ide/setup
- Visual Studio Marketplace: SonarQube for IDE  
  https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode
