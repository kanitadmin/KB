---
title: Ubuntu Server - แผนและขั้นตอนการอัปเดตระบบ
tags:
  - ubuntu-server
  - patch-management
  - update
  - linux
  - infrastructure
  - คู่มือ
created: 2026-06-28
updated: 2026-06-28
---

# Ubuntu Server - แผนและขั้นตอนการอัปเดตระบบ

## วัตถุประสงค์

เอกสารนี้ใช้เป็นแนวทางการวางแผนและดำเนินการอัปเดต Ubuntu Server เพื่อปรับปรุงความปลอดภัย แก้ไขปัญหาของระบบ และลดความเสี่ยงจากช่องโหว่หรือข้อผิดพลาดของแพ็กเกจที่ติดตั้งอยู่

## ขอบเขต

คู่มือนี้ครอบคลุมการอัปเดตแพ็กเกจบน Ubuntu Server ด้วย `apt` เช่น Security Update, Bug Fix, Kernel Update และแพ็กเกจที่ได้รับอนุมัติให้ติดตั้ง

คู่มือนี้ไม่ครอบคลุมการอัปเกรดข้ามรุ่นระบบปฏิบัติการ เช่น Ubuntu 22.04 LTS เป็น Ubuntu 24.04 LTS ซึ่งควรจัดทำแผนแยกต่างหาก

## ข้อมูลระบบ

| รายการ | รายละเอียด |
| --- | --- |
| ชื่อเครื่อง |  |
| IP Address |  |
| Ubuntu Version |  |
| Kernel Version |  |
| บทบาทของ Server |  |
| ระบบหรือ Application ที่เกี่ยวข้อง |  |
| วิธีอัปเดต | Manual / Repository ภายใน / Automation / อื่น ๆ |
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
| 5 | ตรวจสอบแพ็กเกจที่รออัปเดต |  |  | ทราบรายการเปลี่ยนแปลง |
| 6 | ติดตั้ง Update |  |  | Update สำเร็จ |
| 7 | Restart Server หากจำเป็น |  |  | Server กลับมาทำงานปกติ |
| 8 | ตรวจสอบบริการหลังอัปเดต |  |  | Service และ Application ใช้งานได้ |
| 9 | บันทึกผลและปิดงาน |  |  | มีหลักฐานประกอบครบถ้วน |

## ข้อกำหนดก่อนดำเนินการ

- ได้รับอนุมัติ Maintenance Window จากผู้มีอำนาจ
- แจ้งผู้ใช้งานหรือเจ้าของระบบล่วงหน้าตามระยะเวลาที่กำหนด
- มีสิทธิ์ `root` หรือสิทธิ์ `sudo`
- ตรวจสอบว่า Backup หรือ Snapshot สามารถใช้งานได้จริง
- ตรวจสอบพื้นที่ว่างของ Disk โดยเฉพาะ `/`, `/boot` และ `/var`
- ตรวจสอบว่า Repository ที่ใช้งานถูกต้องและเข้าถึงได้
- ตรวจสอบว่า Server ไม่มีงานสำคัญกำลังประมวลผล
- เตรียมแผน Rollback และผู้ประสานงานกรณีเกิดเหตุขัดข้อง

## ขั้นตอนที่ 1 ตรวจสอบสถานะก่อนอัปเดต

### 1.1 ตรวจสอบข้อมูลระบบ

```bash
hostnamectl
lsb_release -a
uname -r
uptime
```

### 1.2 ตรวจสอบพื้นที่ว่างของ Disk

```bash
df -h
df -h /boot
du -sh /var/cache/apt 2>/dev/null
```

หากพื้นที่ `/`, `/boot` หรือ `/var` เหลือน้อย ควรแก้ไขก่อนดำเนินการอัปเดต

### 1.3 ตรวจสอบสถานะ Service สำคัญ

```bash
systemctl --failed
systemctl list-units --type=service --state=running
```

หากเป็น Server เฉพาะทาง เช่น Web Server, Database Server, Application Server, DNS, DHCP หรือ File Server ให้ตรวจสอบ Service ของระบบนั้นเพิ่มเติม

### 1.4 ตรวจสอบแพ็กเกจที่ถูก Hold

```bash
apt-mark showhold
```

หากมีแพ็กเกจที่ถูก Hold ให้ตรวจสอบเหตุผลก่อนอัปเดต เพื่อป้องกันผลกระทบต่อ Application

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 2 สำรองข้อมูลและเตรียม Rollback

### 2.1 สำรองข้อมูล

ให้เลือกวิธีสำรองข้อมูลตามมาตรฐานขององค์กร เช่น

- VM Snapshot
- Backup ระดับระบบปฏิบัติการ
- Backup ระดับ Application
- Backup Database
- Export Configuration ของ Service สำคัญ

### 2.2 สำรองรายการแพ็กเกจและ Repository

```bash
mkdir -p /root/ubuntu-update-backup-$(date +%F)
BACKUP_DIR=/root/ubuntu-update-backup-$(date +%F)

dpkg -l > "$BACKUP_DIR/dpkg-list-before.txt"
apt-mark showmanual > "$BACKUP_DIR/apt-manual-before.txt"
apt-mark showhold > "$BACKUP_DIR/apt-hold-before.txt"
cp -a /etc/apt "$BACKUP_DIR/"
```

### 2.3 บันทึกสถานะก่อนอัปเดต

```bash
hostnamectl > "$BACKUP_DIR/hostnamectl-before.txt"
uname -a > "$BACKUP_DIR/kernel-before.txt"
df -h > "$BACKUP_DIR/df-before.txt"
systemctl --failed > "$BACKUP_DIR/systemctl-failed-before.txt"
```

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 3 ตรวจสอบรายการ Update

### 3.1 อัปเดตรายการแพ็กเกจจาก Repository

```bash
sudo apt update
```

### 3.2 ตรวจสอบรายการแพ็กเกจที่สามารถอัปเดตได้

```bash
apt list --upgradable
```

### 3.3 ตรวจสอบรายการ Security Update

```bash
apt list --upgradable | grep -i security
```

หากองค์กรใช้ Ubuntu Pro หรือ ESM ให้ตรวจสอบสถานะเพิ่มเติม

```bash
pro status
```

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 4 ติดตั้ง Update

### วิธีที่ 1 อัปเดตแบบมาตรฐาน

เหมาะสำหรับการอัปเดตแพ็กเกจทั่วไป โดยไม่ลบแพ็กเกจเดิมออกโดยอัตโนมัติ

```bash
sudo apt upgrade
```

ตรวจสอบรายการที่จะติดตั้งก่อนยืนยัน หากพบแพ็กเกจสำคัญ เช่น Kernel, Database, Web Server หรือ Runtime ของ Application ให้พิจารณาผลกระทบก่อนกดดำเนินการ

### วิธีที่ 2 อัปเดตแบบ Full Upgrade

ใช้เมื่อจำเป็นต้องติดตั้งแพ็กเกจใหม่หรือลบแพ็กเกจบางรายการเพื่อแก้ Dependency

```bash
sudo apt full-upgrade
```

ควรใช้วิธีนี้ภายใต้ Maintenance Window เท่านั้น และต้องตรวจสอบรายการแพ็กเกจที่จะถูกลบก่อนยืนยันเสมอ

### วิธีที่ 3 อัปเดตเฉพาะแพ็กเกจ

เหมาะสำหรับกรณีที่ต้องการอัปเดตเฉพาะแพ็กเกจที่ได้รับอนุมัติ

```bash
sudo apt install --only-upgrade <package-name>
```

### วิธีที่ 4 อัปเดตแบบไม่ถามยืนยัน

ใช้เฉพาะกรณีที่ผ่านการทดสอบและได้รับอนุมัติแล้ว

```bash
sudo apt upgrade -y
```

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 5 ตรวจสอบความจำเป็นในการ Restart

ตรวจสอบว่าระบบต้อง Restart หรือไม่

```bash
test -f /var/run/reboot-required && cat /var/run/reboot-required
test -f /var/run/reboot-required.pkgs && cat /var/run/reboot-required.pkgs
```

หากมีการอัปเดต Kernel, libc หรือแพ็กเกจสำคัญระดับระบบ มักจำเป็นต้อง Restart Server

ตรวจสอบ Service ที่อาจต้อง Restart

```bash
sudo systemctl --failed
```

หากมีเครื่องมือ `needrestart` สามารถใช้ตรวจสอบ Process หรือ Service ที่ควร Restart ได้

```bash
sudo needrestart
```

## ขั้นตอนที่ 6 Restart Server

หากต้อง Restart ให้ดำเนินการภายใน Maintenance Window เท่านั้น

```bash
sudo reboot
```

หลัง Restart ให้รอจนระบบกลับมา Online และสามารถ SSH เข้าใช้งานได้

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 7 ตรวจสอบหลังอัปเดต

### 7.1 ตรวจสอบข้อมูลระบบและ Kernel

```bash
hostnamectl
uname -r
uptime
```

### 7.2 ตรวจสอบแพ็กเกจที่ติดตั้งล่าสุด

```bash
grep " install " /var/log/dpkg.log | tail -n 30
grep " upgrade " /var/log/dpkg.log | tail -n 30
```

### 7.3 ตรวจสอบสถานะ Service

```bash
systemctl --failed
systemctl status <service-name> --no-pager
```

### 7.4 ตรวจสอบ Log สำคัญ

```bash
journalctl -p warning -n 100 --no-pager
tail -n 100 /var/log/apt/history.log
tail -n 100 /var/log/apt/term.log
```

### 7.5 ตรวจสอบ Application

ตรวจสอบร่วมกับเจ้าของระบบหรือผู้ดูแล Application อย่างน้อยดังนี้

- Login เข้า Application ได้
- Function สำคัญทำงานได้
- ระบบเชื่อมต่อ Database หรือระบบปลายทางได้
- ไม่มี Error ใหม่ใน Log
- Monitoring แสดงสถานะปกติ

พื้นที่สำหรับรูปภาพหรือหลักฐาน:



## ขั้นตอนที่ 8 ล้างแพ็กเกจที่ไม่จำเป็น

ดำเนินการหลังยืนยันว่าระบบทำงานปกติแล้วเท่านั้น

ตรวจสอบรายการแพ็กเกจที่ไม่จำเป็น

```bash
sudo apt autoremove --dry-run
```

หากรายการถูกต้องและไม่มีผลกระทบ ให้ดำเนินการ

```bash
sudo apt autoremove
sudo apt autoclean
```

## ขั้นตอนที่ 9 บันทึกผลและปิดงาน

บันทึกข้อมูลหลังดำเนินการอย่างน้อยดังนี้

- วันที่และเวลาที่เริ่มและสิ้นสุดงาน
- รายการแพ็กเกจที่อัปเดต
- Kernel Version ก่อนและหลังอัปเดต
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
| สำรองรายการแพ็กเกจและ Repository แล้ว |  |  |
| ตรวจสอบพื้นที่ว่างของ Disk แล้ว |  |  |
| ตรวจสอบแพ็กเกจที่ถูก Hold แล้ว |  |  |
| ตรวจสอบรายการ Update แล้ว |  |  |
| ติดตั้ง Update สำเร็จ |  |  |
| Restart Server สำเร็จ หากจำเป็น |  |  |
| ตรวจสอบ Kernel และแพ็กเกจหลังอัปเดตแล้ว |  |  |
| Service สำคัญทำงานปกติ |  |  |
| Application ใช้งานได้ปกติ |  |  |
| Monitoring แสดงสถานะปกติ |  |  |
| บันทึกหลักฐานและปิดงานแล้ว |  |  |

## แนวทาง Rollback

หากพบปัญหาหลังอัปเดต ให้ดำเนินการตามลำดับดังนี้

1. ประเมินผลกระทบและแจ้งผู้เกี่ยวข้องทันที
2. ตรวจสอบ `/var/log/apt/history.log`, `/var/log/apt/term.log` และ `journalctl`
3. หากเป็นปัญหา Service ให้ตรวจสอบ Configuration และ Restart Service ที่เกี่ยวข้อง
4. หากเป็นปัญหาเฉพาะแพ็กเกจ ให้พิจารณาติดตั้ง Version เดิมจาก Repository หรือ Package Cache เฉพาะกรณีที่ผ่านการประเมินแล้ว
5. หากระบบไม่สามารถให้บริการได้ตามเวลาที่กำหนด ให้กู้คืนจาก Backup หรือ Snapshot ที่เตรียมไว้

ตัวอย่างการตรวจสอบรายการ Update ล่าสุด

```bash
less /var/log/apt/history.log
less /var/log/dpkg.log
```

ตัวอย่างการติดตั้งแพ็กเกจ Version เฉพาะ

```bash
apt-cache policy <package-name>
sudo apt install <package-name>=<version>
```

หมายเหตุ: การ Downgrade แพ็กเกจอาจกระทบ Dependency และไม่ควรดำเนินการในระบบ Production โดยไม่มีการทดสอบหรือแผนกู้คืนที่ชัดเจน

## หมายเหตุเกี่ยวกับ Automatic Updates

Ubuntu Server อาจมี `unattended-upgrades` สำหรับติดตั้ง Security Update อัตโนมัติ ให้ตรวจสอบนโยบายขององค์กรก่อนเปิดหรือปิดการใช้งาน

ตรวจสอบสถานะ

```bash
systemctl status unattended-upgrades --no-pager
cat /etc/apt/apt.conf.d/20auto-upgrades
ls -lh /var/log/unattended-upgrades/
```

หากใช้ Production Server ควรกำหนดนโยบายเรื่องเวลาการอัปเดต การ Restart อัตโนมัติ และการแจ้งเตือนให้ชัดเจน

## แหล่งอ้างอิง

- Ubuntu Server Documentation: Package management  
  https://ubuntu.com/server/docs/how-to/software/package-management/
- Ubuntu Server Documentation: Automatic updates  
  https://ubuntu.com/server/docs/how-to/software/automatic-updates/
- Ubuntu Server Documentation: Upgrade your release  
  https://ubuntu.com/server/docs/how-to/software/upgrade-your-release/
