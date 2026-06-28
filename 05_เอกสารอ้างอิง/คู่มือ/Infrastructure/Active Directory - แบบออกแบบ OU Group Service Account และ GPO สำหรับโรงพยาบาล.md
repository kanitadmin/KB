---
title: Active Directory - แบบออกแบบ OU Group Service Account และ GPO สำหรับโรงพยาบาล
tags:
  - active-directory
  - group-policy
  - identity
  - hospital-it
  - infrastructure
  - คู่มือ
created: 2026-06-28
updated: 2026-06-28
---

# Active Directory - แบบออกแบบ OU Group Service Account และ GPO สำหรับโรงพยาบาล

## วัตถุประสงค์

เอกสารนี้ใช้เป็นแนวทางออกแบบโครงสร้าง Organizational Unit หรือ OU, กลุ่มผู้ใช้งาน, บัญชี Service Account และ Group Policy Object หรือ GPO สำหรับ Active Directory Domain Services ของโรงพยาบาล โดยเน้นความปลอดภัย การบริหารสิทธิ์ตามบทบาท และความง่ายต่อการดูแลระยะยาว

## หลักการออกแบบ

- แยก OU ตามวัตถุประสงค์การบริหารและการบังคับใช้นโยบาย ไม่ยึดตามแผนกเพียงอย่างเดียว
- ใช้ Role Level สำหรับผู้ใช้งาน เพื่อควบคุมสิทธิ์ตามหน้าที่งาน
- แยกบัญชีผู้ใช้ทั่วไป บัญชีผู้ดูแลระบบ และ Service Account ออกจากกันอย่างชัดเจน
- ใช้หลัก Least Privilege ให้สิทธิ์เท่าที่จำเป็นต่อการปฏิบัติงาน
- ใช้กลุ่มตามแนวทาง AGDLP ได้แก่ Account, Global, Domain Local, Permission
- ใช้ GPO แบบแยกหน้าที่ ชื่อชัดเจน และผูกกับ OU เฉพาะจุด
- หลีกเลี่ยงการแก้ไข Default Domain Policy โดยไม่จำเป็น
- ทดสอบ GPO ใน OU ทดสอบก่อนใช้งานจริงเสมอ

## ขอบเขต

คู่มือนี้ครอบคลุมโครงสร้าง Active Directory สำหรับโรงพยาบาลที่มีผู้ใช้งานหลายบทบาท เช่น แพทย์ พยาบาล เภสัชกร ห้องปฏิบัติการ รังสีวิทยา งานการเงิน งานเวชระเบียน งานบริหาร และฝ่าย IT

คู่มือนี้ไม่ครอบคลุมการออกแบบ Entra ID, Hybrid Join, Conditional Access หรือระบบ IAM ภายนอก

## Naming Convention

| ประเภท | รูปแบบชื่อ | ตัวอย่าง |
| --- | --- | --- |
| OU | `OU=<ลำดับ>_<ชื่อ>` | `OU=01_Users` |
| User | `<รหัสพนักงาน>` หรือ `<ชื่อย่อ>.<นามสกุล>` | `650123`, `somchai.k` |
| Admin User | `adm-<username>` | `adm-somchai.k` |
| Service Account | `svc-<system>-<purpose>` | `svc-his-api` |
| gMSA | `gmsa-<system>-<purpose>$` | `gmsa-his-web$` |
| Global Group | `GG_<ROLE/DEPT>_<NAME>` | `GG_RL03_DOCTOR_USERS` |
| Domain Local Group | `DL_<RESOURCE>_<PERMISSION>` | `DL_FS_HR_READ` |
| GPO | `GPO-<Scope>-<Purpose>` | `GPO-C-Workstation-Baseline` |

## Role Level สำหรับผู้ใช้งาน

| Role Level | กลุ่มผู้ใช้งาน | ตัวอย่างบทบาท | แนวทางสิทธิ์ |
| --- | --- | --- | --- |
| RL01 | Executive | ผู้อำนวยการ, รองผู้อำนวยการ | เข้าถึงข้อมูลบริหารตามอนุมัติ |
| RL02 | Clinical Leadership | หัวหน้าแพทย์, หัวหน้าพยาบาล, หัวหน้าหน่วยคลินิก | เข้าถึงข้อมูลหน่วยงานและรายงานทางคลินิก |
| RL03 | Clinical Practitioner | แพทย์, พยาบาล, ทันตแพทย์ | เข้าถึง HIS/EMR ตามบทบาทการรักษา |
| RL04 | Clinical Support | เภสัชกร, ห้อง Lab, รังสีวิทยา, กายภาพ | เข้าถึงระบบสนับสนุนการรักษา |
| RL05 | Back Office | การเงิน, บัญชี, HR, จัดซื้อ, เวชระเบียน | เข้าถึงระบบสำนักงานและระบบเฉพาะงาน |
| RL06 | IT Standard User | เจ้าหน้าที่ IT บัญชีใช้งานทั่วไป | ไม่มีสิทธิ์ผู้ดูแลระบบ |
| RL07 | Contractor / Temporary | Vendor, Outsource, นักศึกษาฝึกงาน | จำกัดเวลาและจำกัดสิทธิ์อย่างเข้มงวด |
| RL08 | Service / System | บัญชีระบบหรือบริการ | ห้าม Interactive Logon |

## โครงสร้าง OU ที่แนะนำ

```text
hospital.local
├── 00_Admin
│   ├── Tier0_Domain_Admins
│   ├── Tier1_Server_Admins
│   ├── Tier2_Workstation_Admins
│   ├── Helpdesk
│   └── Admin_Workstations
├── 01_Users
│   ├── RL01_Executive
│   ├── RL02_Clinical_Leadership
│   ├── RL03_Clinical_Practitioner
│   ├── RL04_Clinical_Support
│   ├── RL05_Back_Office
│   ├── RL06_IT_Standard_User
│   ├── RL07_Contractor_Temporary
│   └── Disabled_Users
├── 02_Groups
│   ├── Global_Role_Groups
│   ├── Global_Department_Groups
│   ├── Domain_Local_Resource_Groups
│   ├── GPO_Filtering_Groups
│   ├── Mail_Distribution_Groups
│   └── Service_Groups
├── 03_Computers
│   ├── Workstations
│   │   ├── Clinical_Stations
│   │   ├── Nursing_Stations
│   │   ├── Pharmacy
│   │   ├── Laboratory
│   │   ├── Radiology
│   │   ├── Back_Office
│   │   └── Shared_Kiosk
│   ├── Laptops
│   ├── IT_Workstations
│   └── Quarantine
├── 04_Servers
│   ├── Domain_Controllers
│   ├── Application_Servers
│   ├── Database_Servers
│   ├── File_Print_Servers
│   ├── Web_Servers
│   ├── Backup_Servers
│   ├── Monitoring_Servers
│   └── Test_Dev_Servers
├── 05_Service_Accounts
│   ├── gMSA
│   ├── Standard_Service_Accounts
│   ├── Vendor_Service_Accounts
│   └── Scheduled_Task_Accounts
├── 06_Medical_Devices
│   ├── Domain_Joined
│   ├── Vendor_Managed
│   └── Quarantine
├── 07_Terminal_Services
│   ├── RDS_Servers
│   ├── Published_App_Users
│   └── RDS_Profile_Groups
├── 08_Testing
│   ├── GPO_Test_Users
│   ├── GPO_Test_Computers
│   └── Pilot
└── 99_Disabled_Objects
    ├── Disabled_Users
    ├── Disabled_Computers
    └── Retired_Service_Accounts
```

## คำอธิบาย OU สำคัญ

| OU | วัตถุประสงค์ | หมายเหตุ |
| --- | --- | --- |
| `00_Admin` | เก็บบัญชีและเครื่องสำหรับงานผู้ดูแลระบบ | ต้องควบคุมเข้มงวดและแยกจากผู้ใช้ทั่วไป |
| `01_Users` | เก็บบัญชีผู้ใช้งานตาม Role Level | ใช้ผูก User GPO และกำหนดสิทธิ์ตามบทบาท |
| `02_Groups` | เก็บกลุ่มทั้งหมด | ลดความสับสนและง่ายต่อการ Audit |
| `03_Computers` | เก็บเครื่อง Client ตามลักษณะใช้งาน | ใช้ผูก Computer GPO |
| `04_Servers` | เก็บ Server ตามบทบาท | ใช้ GPO เฉพาะ Server Role |
| `05_Service_Accounts` | เก็บบัญชีระบบและบริการ | ต้องห้าม Interactive Logon |
| `06_Medical_Devices` | เก็บเครื่องมือแพทย์ที่ Join Domain | ใช้นโยบายเฉพาะและควบคุม Vendor |
| `08_Testing` | พื้นที่ทดสอบ GPO ก่อนใช้งานจริง | ต้องใช้ก่อน Deploy ทุกครั้ง |
| `99_Disabled_Objects` | เก็บ Object ที่ปิดใช้งานแล้ว | ช่วย Audit และลดความเสี่ยง |

## โครงสร้าง Group ที่แนะนำ

### กลุ่มตาม Role Level

| Group | สมาชิก | ใช้งาน |
| --- | --- | --- |
| `GG_RL01_EXECUTIVE_USERS` | ผู้บริหาร | สิทธิ์ระบบบริหารและรายงาน |
| `GG_RL02_CLINICAL_LEAD_USERS` | หัวหน้าหน่วยคลินิก | สิทธิ์รายงานและอนุมัติตามหน้าที่ |
| `GG_RL03_DOCTOR_USERS` | แพทย์ | สิทธิ์ HIS/EMR สำหรับแพทย์ |
| `GG_RL03_NURSE_USERS` | พยาบาล | สิทธิ์ HIS/EMR สำหรับพยาบาล |
| `GG_RL04_PHARMACY_USERS` | เภสัชกร | สิทธิ์ระบบยา |
| `GG_RL04_LAB_USERS` | ห้องปฏิบัติการ | สิทธิ์ระบบ LIS |
| `GG_RL04_RADIOLOGY_USERS` | รังสีวิทยา | สิทธิ์ระบบ RIS/PACS |
| `GG_RL05_FINANCE_USERS` | การเงิน | สิทธิ์ระบบการเงิน |
| `GG_RL05_HR_USERS` | HR | สิทธิ์ระบบบุคลากร |
| `GG_RL05_MEDICAL_RECORD_USERS` | เวชระเบียน | สิทธิ์ระบบเวชระเบียน |
| `GG_RL07_CONTRACTOR_USERS` | Vendor หรือ Outsource | สิทธิ์จำกัดเวลา |

### กลุ่มตามแผนก

| Group | ตัวอย่างสมาชิก | ใช้งาน |
| --- | --- | --- |
| `GG_DEPT_ER_USERS` | ER | สิทธิ์ Share หรือ Application ของ ER |
| `GG_DEPT_OPD_USERS` | OPD | สิทธิ์ Share หรือ Application ของ OPD |
| `GG_DEPT_IPD_USERS` | IPD | สิทธิ์ Share หรือ Application ของ IPD |
| `GG_DEPT_OR_USERS` | ห้องผ่าตัด | สิทธิ์ Share หรือ Application ของ OR |
| `GG_DEPT_ICU_USERS` | ICU | สิทธิ์ Share หรือ Application ของ ICU |
| `GG_DEPT_ADMIN_USERS` | ธุรการ | สิทธิ์ Share ส่วนกลาง |

### กลุ่มสิทธิ์ Resource

| Group | สิทธิ์ | ตัวอย่างการใช้งาน |
| --- | --- | --- |
| `DL_FS_HR_READ` | Read | อ่านไฟล์ HR |
| `DL_FS_HR_MODIFY` | Modify | แก้ไขไฟล์ HR |
| `DL_FS_FINANCE_READ` | Read | อ่านไฟล์การเงิน |
| `DL_FS_FINANCE_MODIFY` | Modify | แก้ไขไฟล์การเงิน |
| `DL_APP_HIS_DOCTOR` | Application Role | สิทธิ์แพทย์ใน HIS |
| `DL_APP_HIS_NURSE` | Application Role | สิทธิ์พยาบาลใน HIS |
| `DL_APP_LIS_USERS` | Application Role | ใช้งาน LIS |
| `DL_APP_RIS_PACS_USERS` | Application Role | ใช้งาน RIS/PACS |

### กลุ่มผู้ดูแลระบบ

| Group | Scope | แนวทางใช้งาน |
| --- | --- | --- |
| `GG_AD_TIER0_DOMAIN_ADMINS` | Tier 0 | บริหาร Domain Controller, AD, DNS, PKI |
| `GG_AD_TIER1_SERVER_ADMINS` | Tier 1 | บริหาร Member Server |
| `GG_AD_TIER2_WORKSTATION_ADMINS` | Tier 2 | บริหารเครื่อง Client |
| `GG_AD_HELPDESK_PASSWORD_RESET` | Helpdesk | Reset Password เฉพาะ OU ที่มอบหมาย |
| `GG_AD_GPO_ADMINS` | GPO Admin | จัดการ GPO ตามสิทธิ์ที่อนุมัติ |
| `GG_AD_AUDIT_READONLY` | Audit | อ่าน Log และตรวจสอบการตั้งค่า |

## แนวทางใช้ AGDLP

ให้เพิ่มผู้ใช้เข้า `Global Group` ตามบทบาทหรือแผนก จากนั้นนำ `Global Group` ไปเป็นสมาชิกของ `Domain Local Group` ที่ผูกกับสิทธิ์ของ Resource

ตัวอย่าง:

```text
User: somchai.k
เข้าเป็นสมาชิก: GG_RL03_DOCTOR_USERS
GG_RL03_DOCTOR_USERS เข้าเป็นสมาชิก: DL_APP_HIS_DOCTOR
DL_APP_HIS_DOCTOR ได้รับสิทธิ์ในระบบ HIS
```

## Service Account Design

### ประเภท Service Account

| ประเภท | OU | แนวทางใช้งาน |
| --- | --- | --- |
| gMSA | `05_Service_Accounts/gMSA` | แนะนำสำหรับ Windows Service ที่รองรับ |
| Standard Service Account | `05_Service_Accounts/Standard_Service_Accounts` | ใช้เฉพาะระบบที่ไม่รองรับ gMSA |
| Vendor Service Account | `05_Service_Accounts/Vendor_Service_Accounts` | ใช้กับระบบ Vendor และต้องมีเจ้าของชัดเจน |
| Scheduled Task Account | `05_Service_Accounts/Scheduled_Task_Accounts` | ใช้สำหรับ Task ที่ต้องรันตามรอบเวลา |

### มาตรฐาน Service Account

- ใช้ gMSA เป็นค่าเริ่มต้นสำหรับ Windows Service ที่รองรับ
- Service Account ต้องมี Owner, Purpose, System, Expiry หรือ Review Date
- ห้ามใช้ Service Account สำหรับ Login แบบ Interactive
- ห้ามใช้ Domain Admin หรือบัญชีผู้ดูแลระบบเป็น Service Account
- กำหนดสิทธิ์เฉพาะ Server และ Application ที่จำเป็น
- บันทึกการใช้งาน Service Account ลงทะเบียนกลาง
- ทบทวน Service Account อย่างน้อยทุก 6 เดือน

### กลุ่มสำหรับ Service Account

| Group | ใช้งาน |
| --- | --- |
| `GG_SVC_HIS_ACCOUNTS` | รวม Service Account ของระบบ HIS |
| `GG_SVC_LIS_ACCOUNTS` | รวม Service Account ของระบบ LIS |
| `GG_SVC_RIS_PACS_ACCOUNTS` | รวม Service Account ของระบบ RIS/PACS |
| `GG_SVC_BACKUP_ACCOUNTS` | รวม Service Account ของระบบ Backup |
| `GG_SVC_MONITORING_ACCOUNTS` | รวม Service Account ของระบบ Monitoring |
| `GG_SVC_DENY_INTERACTIVE_LOGON` | ใช้ Deny Interactive Logon ผ่าน GPO |

## แนวทางออกแบบ GPO

### หลักการจัดการ GPO

- ใช้ Microsoft Security Baseline จาก Security Compliance Toolkit เป็นจุดตั้งต้น แล้วปรับให้เหมาะกับระบบโรงพยาบาล
- แยก GPO ตามหน้าที่ เช่น Security Baseline, Firewall, Update, User Experience, Application Control
- หลีกเลี่ยง GPO ขนาดใหญ่ที่รวมทุกอย่างไว้ใน Policy เดียว
- ใช้ Security Filtering เฉพาะกรณีจำเป็น และต้องมีชื่อกลุ่มชัดเจน
- ใช้ WMI Filter เฉพาะกรณีจำเป็น เช่น แยก Windows Server และ Windows Client
- ทุก GPO ต้องมี Owner, Purpose, Linked OU, Change Ticket และวันที่ทบทวน

### รายการ GPO ที่จำเป็น

| GPO | Scope | วัตถุประสงค์หลัก |
| --- | --- | --- |
| `GPO-D-Domain-Password-AccountLockout` | Domain | Password Policy และ Account Lockout |
| `GPO-D-Domain-Kerberos` | Domain | Kerberos Policy |
| `GPO-DC-DomainController-Baseline` | Domain Controllers | Security Baseline สำหรับ Domain Controller |
| `GPO-DC-DomainController-Audit` | Domain Controllers | Advanced Audit Policy |
| `GPO-C-Workstation-Baseline` | Workstations | Hardening เครื่อง Client |
| `GPO-C-Workstation-Defender` | Workstations | Microsoft Defender Antivirus |
| `GPO-C-Workstation-Firewall` | Workstations | Windows Defender Firewall |
| `GPO-C-Workstation-BitLocker` | Workstations / Laptops | เข้ารหัส Disk และเก็บ Recovery Key ใน AD |
| `GPO-C-Workstation-USB-Control` | Workstations | จำกัด Removable Storage |
| `GPO-C-Workstation-LocalAdmin-LAPS` | Workstations | จัดการ Local Administrator ด้วย Windows LAPS |
| `GPO-C-Workstation-WSUS` | Workstations | กำหนด Windows Update / WSUS |
| `GPO-C-Clinical-SharedStation` | Shared Clinical PCs | ตั้งค่าเครื่องใช้งานร่วมกัน |
| `GPO-C-Clinical-Kiosk-Lockdown` | Kiosk | จำกัดการใช้งานเครื่อง Kiosk |
| `GPO-C-MedicalDevice-Exception` | Medical Devices | นโยบายเฉพาะเครื่องมือแพทย์ที่มีข้อจำกัด |
| `GPO-S-Server-Baseline` | Member Servers | Hardening Server ทั่วไป |
| `GPO-S-Server-Defender` | Member Servers | Defender สำหรับ Server |
| `GPO-S-Server-Firewall` | Member Servers | Firewall สำหรับ Server |
| `GPO-S-Server-WSUS` | Member Servers | Windows Update / WSUS สำหรับ Server |
| `GPO-S-Server-AdminAccess` | Member Servers | จำกัด Local Admin และ RDP |
| `GPO-S-Database-Server` | Database Servers | นโยบายเฉพาะ Database Server |
| `GPO-S-Backup-Server` | Backup Servers | นโยบายเฉพาะ Backup Server |
| `GPO-U-User-Baseline` | Users | ตั้งค่าผู้ใช้พื้นฐาน |
| `GPO-U-User-ScreenLock` | Users | Auto Lock และ Screen Saver |
| `GPO-U-User-Browser-Security` | Users | Browser Security |
| `GPO-U-Clinical-Users` | Clinical Users | Policy สำหรับผู้ใช้คลินิก |
| `GPO-U-BackOffice-Users` | Back Office Users | Policy สำหรับผู้ใช้สำนักงาน |
| `GPO-U-Contractor-Restricted` | Contractor Users | จำกัดสิทธิ์ Vendor/Outsource |
| `GPO-U-Admin-Account-Restriction` | Admin Users | จำกัดการใช้งานบัญชี Admin |
| `GPO-SVC-ServiceAccount-DenyLogon` | Service Accounts | ห้าม Service Account Login แบบ Interactive |

### Mapping GPO เข้ากับ OU

| OU | GPO ที่ควรผูก | หมายเหตุ |
| --- | --- | --- |
| Domain Root | `GPO-D-Domain-Password-AccountLockout`, `GPO-D-Domain-Kerberos` | ใช้เฉพาะ Policy ระดับ Domain |
| `00_Admin` | `GPO-U-Admin-Account-Restriction` | จำกัดการใช้งานบัญชีผู้ดูแลระบบ |
| `00_Admin/Admin_Workstations` | `GPO-C-Workstation-Baseline`, `GPO-C-Workstation-Defender`, `GPO-C-Workstation-Firewall`, `GPO-C-Workstation-BitLocker`, `GPO-C-Workstation-LocalAdmin-LAPS` | ควรมีนโยบายเข้มกว่าผู้ใช้ทั่วไป |
| `01_Users` | `GPO-U-User-Baseline`, `GPO-U-User-ScreenLock`, `GPO-U-User-Browser-Security` | Policy ผู้ใช้พื้นฐาน |
| `01_Users/RL01_Executive` | `GPO-U-User-Baseline`, `GPO-U-User-ScreenLock`, `GPO-U-User-Browser-Security` | เพิ่ม Policy เฉพาะผู้บริหารตามความจำเป็น |
| `01_Users/RL02_Clinical_Leadership` | `GPO-U-Clinical-Users` | ใช้กับหัวหน้าหน่วยคลินิก |
| `01_Users/RL03_Clinical_Practitioner` | `GPO-U-Clinical-Users` | ใช้กับแพทย์และพยาบาล |
| `01_Users/RL04_Clinical_Support` | `GPO-U-Clinical-Users` | ใช้กับเภสัช Lab รังสี |
| `01_Users/RL05_Back_Office` | `GPO-U-BackOffice-Users` | ใช้กับงานสำนักงาน |
| `01_Users/RL07_Contractor_Temporary` | `GPO-U-Contractor-Restricted` | ต้องจำกัดสิทธิ์และระยะเวลา |
| `03_Computers/Workstations` | `GPO-C-Workstation-Baseline`, `GPO-C-Workstation-Defender`, `GPO-C-Workstation-Firewall`, `GPO-C-Workstation-WSUS`, `GPO-C-Workstation-LocalAdmin-LAPS` | เครื่อง Client ทั่วไป |
| `03_Computers/Laptops` | เพิ่ม `GPO-C-Workstation-BitLocker` | บังคับเข้ารหัส Disk |
| `03_Computers/Workstations/Shared_Kiosk` | `GPO-C-Clinical-SharedStation`, `GPO-C-Clinical-Kiosk-Lockdown` | อาจใช้ Loopback Processing |
| `03_Computers/Workstations/Nursing_Stations` | `GPO-C-Clinical-SharedStation` | เครื่องใช้งานร่วมกันในพื้นที่คลินิก |
| `04_Servers` | `GPO-S-Server-Baseline`, `GPO-S-Server-Defender`, `GPO-S-Server-Firewall`, `GPO-S-Server-WSUS`, `GPO-S-Server-AdminAccess` | Member Server ทั่วไป |
| `04_Servers/Domain_Controllers` | `GPO-DC-DomainController-Baseline`, `GPO-DC-DomainController-Audit` | ควรอยู่ภายใต้ Domain Controllers OU มาตรฐาน |
| `04_Servers/Database_Servers` | `GPO-S-Database-Server` | ปรับ Firewall และสิทธิ์เฉพาะ Database |
| `04_Servers/Backup_Servers` | `GPO-S-Backup-Server` | จำกัดการเข้าถึงสูงเป็นพิเศษ |
| `05_Service_Accounts` | `GPO-SVC-ServiceAccount-DenyLogon` | ใช้ Deny Interactive Logon |
| `06_Medical_Devices/Domain_Joined` | `GPO-C-MedicalDevice-Exception` | ต้องทดสอบกับ Vendor ก่อน |
| `08_Testing` | GPO Pilot ทุกชุด | ทดสอบก่อน Deploy จริง |
| `99_Disabled_Objects` | ไม่มี GPO เฉพาะ ยกเว้นจำเป็น | ใช้เก็บ Object ที่ปิดใช้งาน |

### ค่าหลักที่ควรกำหนดใน GPO

| หมวด | ค่าที่แนะนำ |
| --- | --- |
| Password / Lockout | ใช้ Password Policy กลาง และพิจารณา Fine-Grained Password Policy สำหรับ Admin |
| Audit | เปิด Advanced Audit Policy สำหรับ Logon, Account Management, Directory Service Changes, Policy Change, Object Access ตามความจำเป็น |
| Local Admin | ใช้ Windows LAPS และจำกัดสมาชิก Local Administrators |
| RDP | อนุญาตเฉพาะกลุ่มผู้ดูแลที่ได้รับอนุมัติ |
| Firewall | เปิด Windows Defender Firewall ทุก Profile |
| Defender | เปิด Real-time Protection และ Cloud-delivered Protection ตามนโยบายองค์กร |
| BitLocker | บังคับใช้กับ Laptop และเครื่องที่เก็บข้อมูลสำคัญ |
| USB | Block หรือ Read-only โดยมี Exception ผ่านกลุ่มที่อนุมัติ |
| Screen Lock | Auto Lock ภายในระยะเวลาที่องค์กรกำหนด |
| Windows Update | ชี้ไป WSUS หรือระบบบริหาร Patch ขององค์กร |
| Logging | Forward Log สำคัญไป SIEM หรือ Log Server |
| Service Account | Deny log on locally, Deny log on through Remote Desktop Services |

## แนวทาง Delegation

| หน้าที่ | สิทธิ์ที่ควรมอบหมาย | OU |
| --- | --- | --- |
| Helpdesk | Reset Password, Unlock Account | `01_Users` เฉพาะ OU ที่อนุมัติ |
| HR IT Coordinator | Request Create/Disable User เท่านั้น | ไม่มีสิทธิ์แก้ AD โดยตรง |
| Server Admin | Join Server, Manage Server OU | `04_Servers` เฉพาะกลุ่ม |
| Workstation Admin | Join Computer, Move Computer | `03_Computers` |
| GPO Admin | Create/Edit/Link GPO ตามอนุมัติ | เฉพาะ OU ที่รับผิดชอบ |
| Audit | Read-only | ทุก OU ที่เกี่ยวข้อง |

## ขั้นตอนนำไปใช้งาน

1. สำรองและ Export โครงสร้าง AD/GPO ปัจจุบันก่อนเปลี่ยนแปลง

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-01-export-current-state.svg]]

2. สร้าง OU หลักและ OU ย่อยในช่วง Maintenance Window

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-02-create-ou.svg]]

3. สร้างกลุ่มตาม Naming Convention

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-03-create-groups.svg]]

4. กำหนด Owner ของทุกกลุ่มและทุก Service Account

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-04-assign-owners.svg]]

5. สร้าง GPO ในสภาพแวดล้อมทดสอบหรือ OU `08_Testing`

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-05-create-gpo-testing.svg]]

6. ทดสอบกับ Pilot User และ Pilot Computer

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-06-pilot.svg]]

7. ตรวจสอบผลด้วย `gpresult`, Event Log และการใช้งาน Application สำคัญ

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-07-validate.svg]]

8. Deploy ทีละกลุ่มงาน โดยเริ่มจาก Back Office ก่อนพื้นที่คลินิก

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-08-phased-deploy.svg]]

9. บันทึกผลและปรับปรุง Exception List

![[07_ไฟล์แนบ/รูปภาพ/Active Directory OU GPO/ad-ou-gpo-step-09-record-exceptions.svg]]

## Checklist สำหรับตรวจรับ

| รายการตรวจสอบ | สถานะ | หมายเหตุ |
| --- | --- | --- |
| OU หลักถูกสร้างครบถ้วน |  |  |
| Role Level ของผู้ใช้ถูกกำหนดครบ |  |  |
| กลุ่ม Role และ Department ถูกสร้างครบ |  |  |
| กลุ่ม Resource ใช้ AGDLP ถูกต้อง |  |  |
| Service Account มี Owner และ Purpose |  |  |
| Service Account ถูกห้าม Interactive Logon |  |  |
| Admin Account แยกจากบัญชีใช้งานประจำวัน |  |  |
| GPO ถูกทดสอบใน OU Testing แล้ว |  |  |
| GPO Mapping ตรงตาม OU |  |  |
| มี Exception สำหรับ Medical Devices ที่ได้รับอนุมัติ |  |  |
| มี Audit และ Log Forwarding |  |  |
| มีเอกสาร Owner และรอบ Review |  |  |

## ข้อควรระวัง

- ห้ามย้าย Domain Controllers ออกจาก OU มาตรฐานโดยไม่มีแผนและการทดสอบ
- ห้ามผูก GPO ที่กระทบวงกว้างกับ Domain Root โดยไม่จำเป็น
- ห้ามใช้ Security Filtering ซับซ้อนเกินไปจนตรวจสอบยาก
- ห้ามให้ผู้ใช้ทั่วไปเป็น Local Administrator
- ห้ามใช้บัญชีเดียวกันเป็นทั้งบัญชีผู้ใช้ประจำวันและบัญชีผู้ดูแลระบบ
- Medical Devices ต้องทดสอบร่วมกับ Vendor ก่อนบังคับใช้นโยบายเข้มงวด

## แหล่งอ้างอิง

- Microsoft Learn: Group Policy overview for Windows Server  
  https://learn.microsoft.com/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview
- Microsoft Learn: Microsoft Security Compliance Toolkit  
  https://learn.microsoft.com/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10
- Microsoft Download Center: Security Compliance Toolkit  
  https://www.microsoft.com/download/details.aspx?id=55319
