---
title: SonarLint - คู่มือการติดตั้งบน VS Code
tags:
  - sonarlint
  - sonarqube
  - vscode
  - devops
  - คู่มือ
created: 2026-06-27
updated: 2026-06-29
---

# SonarLint - คู่มือการติดตั้งบน VS Code

> [!summary] ใช้เมื่อไร
> ใช้คู่มือนี้เมื่อต้องติดตั้ง SonarQube for IDE บน VS Code ให้ทีมพัฒนา เพื่อช่วยตรวจคุณภาพโค้ดตั้งแต่บนเครื่องผู้พัฒนา และเชื่อมกับ SonarQube Server หรือ SonarQube Cloud ขององค์กร

## วัตถุประสงค์

เอกสารนี้ใช้สำหรับติดตั้ง SonarLint บน Visual Studio Code เพื่อช่วยตรวจสอบคุณภาพโค้ด ความปลอดภัย และปัญหาที่ควรแก้ไขระหว่างการพัฒนา

> หมายเหตุ: SonarLint for VS Code ถูกเปลี่ยนชื่อเป็น SonarQube for IDE / SonarQube for VS Code ในเอกสารรุ่นใหม่ แต่ extension id ยังใช้ `SonarSource.sonarlint-vscode`

## ขอบเขต

| ครอบคลุม | ไม่ครอบคลุม |
| --- | --- |
| การติดตั้ง extension บน VS Code | การติดตั้งหรือดูแล SonarQube Server |
| การใช้งานเบื้องต้นและ Connected Mode | การออกแบบ quality gate ระดับองค์กร |
| การตรวจสอบปัญหาหลังติดตั้ง | การปรับ pipeline CI/CD |

## ข้อกำหนดเบื้องต้น

- ติดตั้ง Visual Studio Code แล้ว
- เครื่องสามารถเชื่อมต่ออินเทอร์เน็ตเพื่อดาวน์โหลด extension ได้
- หากต้องการเชื่อมกับ SonarQube Server หรือ SonarQube Cloud ต้องมี URL และ Token ที่ถูกต้อง
- หากเป็นเครื่ององค์กร ต้องอนุญาตให้ติดตั้ง extension จาก Visual Studio Marketplace หรือแหล่งภายในที่องค์กรกำหนด

## ข้อมูลที่ต้องเตรียม

| รายการ | ตัวอย่าง | หมายเหตุ |
| --- | --- | --- |
| VS Code Version |  | ตรวจจาก `Help > About` |
| Extension ID | `SonarSource.sonarlint-vscode` | ใช้ติดตั้งผ่าน command line |
| SonarQube URL | `https://sonarqube.example.org` | ใช้เฉพาะ Connected Mode |
| Token | `<token>` | ห้ามบันทึกลง repository หรือเอกสารสาธารณะ |
| Project Key | `my-project` | ใช้ตอน bind project |

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

> [!warning] การจัดการ Token
> ให้สร้าง Token เฉพาะผู้ใช้หรือเฉพาะวัตถุประสงค์ และยกเลิก Token ทันทีเมื่อผู้ใช้ย้ายทีม ออกจากองค์กร หรือไม่ต้องใช้งานแล้ว

## การตรวจสอบหลังติดตั้ง

| รายการตรวจสอบ | ผลที่คาดหวัง |
| --- | --- |
| Extension แสดงในรายการติดตั้งแล้ว | พบ `SonarSource.sonarlint-vscode` |
| เปิด workspace ของโปรเจกต์ได้ | VS Code วิเคราะห์ไฟล์ในโฟลเดอร์โปรเจกต์ |
| Problems panel แสดง issue ได้ | เห็นปัญหาหรือข้อความว่าไม่พบปัญหา |
| Connected Mode ใช้งานได้ | Project ถูก bind กับ SonarQube Server/Cloud |
| Log ไม่มี error สำคัญ | Output panel ของ SonarQube ไม่มี authentication หรือ network error |

## การตรวจสอบปัญหาเบื้องต้น

หาก extension ไม่ทำงาน ให้ตรวจสอบรายการต่อไปนี้

- ตรวจสอบว่า extension ติดตั้งและเปิดใช้งานแล้ว
- เปิดโฟลเดอร์โปรเจกต์ ไม่ใช่เปิดเฉพาะไฟล์เดี่ยว
- ตรวจสอบภาษาโปรแกรมที่ extension รองรับ
- ตรวจสอบ internet connection หากต้องดาวน์โหลด analyzer เพิ่มเติม
- ตรวจสอบ Token และ URL กรณีใช้ Connected Mode
- เปิด Output panel แล้วเลือก SonarQube เพื่อตรวจสอบ log

## แนวทางแก้ปัญหาที่พบบ่อย

| อาการ | สาเหตุที่พบบ่อย | แนวทางตรวจสอบ |
| --- | --- | --- |
| ติดตั้ง extension ไม่ได้ | Marketplace ถูก block หรือ proxy ไม่อนุญาต | ตรวจสอบ proxy, firewall และ policy ขององค์กร |
| ไม่เห็น issue ในไฟล์ | ยังไม่ได้เปิด workspace หรือภาษาไม่รองรับ | เปิดทั้งโฟลเดอร์โปรเจกต์และตรวจ extension output |
| Connected Mode ล้มเหลว | URL หรือ Token ไม่ถูกต้อง | ตรวจ Token, สิทธิ์ project และ certificate ของ server |
| ผลตรวจไม่ตรงกับ SonarQube Server | ยังไม่ได้ bind project หรือใช้ quality profile คนละชุด | ตรวจ Connected Mode และ project key |
| Extension ทำงานช้า | Workspace ใหญ่หรือ analyzer ดาวน์โหลดไม่ครบ | ตรวจ network, exclude folder ที่ไม่จำเป็น และ output log |

## Checklist สรุปผล

| รายการตรวจสอบ | สถานะ | หมายเหตุ |
| --- | --- | --- |
| ติดตั้ง extension จาก SonarSource แล้ว |  |  |
| เปิดใช้งาน extension แล้ว |  |  |
| ตรวจสอบไฟล์ source code ได้ |  |  |
| ตั้งค่า Connected Mode แล้ว หากต้องใช้ |  |  |
| ไม่บันทึก Token ลงใน repository |  |  |
| ผู้ใช้งานทราบวิธีดู Problems panel และ Output log |  |  |

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
