---
title: MinIO - คู่มือการ Upgrade แบบ Standalone Native
tags:
  - minio
  - object-storage
  - storage
  - upgrade
  - คู่มือ
created: 2026-06-28
updated: 2026-06-28
---

# MinIO - คู่มือการ Upgrade แบบ Standalone Native

## วัตถุประสงค์

เอกสารนี้ใช้เป็นแนวทางการ Upgrade MinIO แบบ Standalone ที่ติดตั้งแบบ Native บน Linux โดยใช้ Binary และควบคุม Service ผ่าน `systemd` เพื่อให้ดำเนินการได้อย่างเป็นลำดับ ลดความเสี่ยงจาก Downtime และมีหลักฐานตรวจสอบหลังดำเนินการ

## ขอบเขต

คู่มือนี้ครอบคลุม MinIO แบบเครื่องเดียวที่ติดตั้งด้วย Binary เช่น `/usr/local/bin/minio` และทำงานเป็น Linux Service

คู่มือนี้ไม่ครอบคลุม MinIO แบบ Docker, Kubernetes, Distributed Cluster, Operator หรือการ Migration ข้ามเครื่อง

## ข้อกำหนดเบื้องต้น

- มีสิทธิ์ `root` หรือสิทธิ์ `sudo`
- มี Maintenance Window ที่ได้รับอนุมัติ
- มี Backup หรือ Snapshot ที่สามารถกู้คืนได้
- มี MinIO Client หรือ `mc` สำหรับตรวจสอบสถานะระบบ
- ทราบ Alias ของ MinIO ใน `mc` เช่น `local`
- เครื่องสามารถ Download MinIO Binary จากแหล่งทางการได้
- ตรวจสอบ Release Notes ของ MinIO รุ่นเป้าหมายก่อนดำเนินการ

## ข้อมูลก่อนดำเนินการ

| รายการ | รายละเอียด |
| --- | --- |
| ชื่อเครื่อง |  |
| IP Address |  |
| Operating System |  |
| MinIO Version ปัจจุบัน |  |
| MinIO Version เป้าหมาย |  |
| Path ของ MinIO Binary |  |
| Path ของ Data Directory |  |
| Path ของ Configuration File |  |
| ผู้ดำเนินการ |  |
| วันที่และเวลาเริ่มดำเนินการ |  |
| วันที่และเวลาสิ้นสุด |  |

พื้นที่สำหรับรูปภาพ:



## ข้อควรระวังสำคัญ

- การ Upgrade แบบ Standalone จะมี Downtime เนื่องจากมี MinIO Server เพียงเครื่องเดียว
- ต้องสำรองข้อมูลและ Configuration ก่อนดำเนินการทุกครั้ง
- ไม่ควรเปิดเผย `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, Access Key หรือ Secret Key ในรูปภาพหรือเอกสารประกอบ
- หลีกเลี่ยงการ Downgrade หลังเปิดใช้งาน Version ใหม่แล้ว หากต้อง Rollback ควรใช้ VM Snapshot หรือ Storage Snapshot ที่ทำไว้ก่อน Upgrade
- หากมี Application เชื่อมต่อ MinIO อยู่ ให้แจ้งผู้เกี่ยวข้องและหยุดการเขียนข้อมูลก่อนเริ่มดำเนินการ

## ลำดับการ Upgrade

ให้ดำเนินการตามลำดับดังนี้

1. ตรวจสอบสถานะและ Version ปัจจุบัน
2. สำรอง Configuration, Binary และข้อมูลสำคัญ
3. Download MinIO Binary รุ่นใหม่
4. หยุด Service
5. แทนที่ Binary
6. Start Service
7. ตรวจสอบผลหลัง Upgrade
8. บันทึกหลักฐานและปิดงาน

## ขั้นตอนที่ 1 ตรวจสอบสถานะและ Version ปัจจุบัน

### 1.1 ตรวจสอบ Service

```bash
systemctl status minio --no-pager
systemctl is-enabled minio
```

### 1.2 ตรวจสอบ Version ของ MinIO Server

```bash
minio --version
```

หากไม่พบคำสั่ง `minio` ให้ตรวจสอบ Path ของ Binary

```bash
command -v minio
readlink -f $(command -v minio)
```

### 1.3 ตรวจสอบตำแหน่งไฟล์ Service และ Configuration

```bash
systemctl cat minio
```

ตรวจสอบไฟล์ Environment ที่ใช้กับ MinIO โดยไม่แสดงรหัสผ่าน

```bash
grep -E '^MINIO_' /etc/default/minio /etc/sysconfig/minio /etc/minio/minio.conf 2>/dev/null | sed -E 's/(PASSWORD|SECRET|KEY)=.*/\1=********/g'
```

### 1.4 ตรวจสอบสถานะผ่าน MinIO Client

แทนค่า `local` ด้วย Alias ที่ใช้งานจริง

```bash
mc admin info local
mc admin user list local
mc ls local
```

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 2 สำรองข้อมูลก่อน Upgrade

### 2.1 สร้างโฟลเดอร์ Backup

```bash
mkdir -p /root/minio-upgrade-backup-$(date +%F)
BACKUP_DIR=/root/minio-upgrade-backup-$(date +%F)
```

### 2.2 สำรอง Binary เดิม

```bash
MINIO_BIN=$(readlink -f $(command -v minio))
cp -a "$MINIO_BIN" "$BACKUP_DIR/minio.old"
"$BACKUP_DIR/minio.old" --version
```

### 2.3 สำรอง Service และ Configuration

```bash
systemctl cat minio > "$BACKUP_DIR/minio.service.txt"
cp -a /etc/default/minio "$BACKUP_DIR/" 2>/dev/null || true
cp -a /etc/sysconfig/minio "$BACKUP_DIR/" 2>/dev/null || true
cp -a /etc/minio "$BACKUP_DIR/" 2>/dev/null || true
```

หากใช้ TLS Certificate ให้สำรองไฟล์ Certificate และ Key ที่เกี่ยวข้องด้วย

### 2.4 Export Configuration ผ่าน `mc`

แทนค่า `local` ด้วย Alias ที่ใช้งานจริง

```bash
mc admin config export local > "$BACKUP_DIR/minio-config-export.txt"
cd "$BACKUP_DIR"
mc admin cluster bucket export local
mv cluster-metadata.zip minio-bucket-export.zip
mc admin cluster iam export local --output "$BACKUP_DIR/minio-iam-export.zip"
```

### 2.5 สำรองข้อมูล Object

แนะนำให้ใช้ VM Snapshot, Storage Snapshot หรือ Backup Solution ขององค์กร ก่อนเริ่ม Upgrade โดยเฉพาะระบบ Production

หากต้องหยุดการเขียนข้อมูลก่อนทำ Snapshot ให้ประสานงานกับเจ้าของระบบและ Application ที่เกี่ยวข้อง

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 3 Download MinIO Binary รุ่นใหม่

Download Binary จากแหล่งทางการ

```bash
mkdir -p /tmp/minio-upgrade
cd /tmp/minio-upgrade
curl -LO https://dl.min.io/server/minio/release/linux-amd64/minio
curl -LO https://dl.min.io/server/minio/release/linux-amd64/minio.sha256sum
chmod +x minio
./minio --version
```

ตรวจสอบค่า Checksum ตามนโยบายขององค์กร

```bash
sha256sum minio
cat minio.sha256sum
```

ค่าที่ได้ควรตรงกับ Checksum จากแหล่งทางการของ MinIO

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 4 หยุดการใช้งานและ Stop Service

แจ้งผู้เกี่ยวข้องให้หยุดการใช้งานหรือหยุดการเขียนข้อมูลลง MinIO ก่อนดำเนินการ

Stop Service

```bash
systemctl stop minio
systemctl is-active minio
```

ตรวจสอบว่าไม่มี Process ของ MinIO ทำงานอยู่

```bash
ps -ef | grep '[m]inio'
```

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 5 แทนที่ Binary

ตรวจสอบ Path ของ Binary เดิมอีกครั้ง

```bash
MINIO_BIN=$(readlink -f $(command -v minio))
echo "$MINIO_BIN"
```

แทนที่ Binary เดิมด้วย Binary รุ่นใหม่

```bash
install -o root -g root -m 0755 /tmp/minio-upgrade/minio "$MINIO_BIN"
```

ตรวจสอบ Version หลังแทนที่ Binary

```bash
minio --version
```

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 6 Start Service

Reload `systemd` และ Start Service

```bash
systemctl daemon-reload
systemctl start minio
systemctl status minio --no-pager
```

ตรวจสอบ Log หลัง Start Service

```bash
journalctl -u minio -n 100 --no-pager
```

หากพบ Error ให้หยุดดำเนินการต่อ และตรวจสอบ Log ก่อนเปิดระบบให้ผู้ใช้งาน

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 7 ตรวจสอบหลัง Upgrade

### 7.1 ตรวจสอบ Version

```bash
minio --version
```

### 7.2 ตรวจสอบสถานะผ่าน Service

```bash
systemctl is-active minio
systemctl status minio --no-pager
```

### 7.3 ตรวจสอบสถานะผ่าน MinIO Client

แทนค่า `local` ด้วย Alias ที่ใช้งานจริง

```bash
mc admin info local
mc ls local
```

### 7.4 ทดสอบ Upload และ Download

สร้างไฟล์ทดสอบ

```bash
echo "minio-upgrade-test $(date)" > /tmp/minio-upgrade-test.txt
```

Upload ไฟล์ไปยัง Bucket ทดสอบหรือ Bucket ที่ได้รับอนุญาต

```bash
mc cp /tmp/minio-upgrade-test.txt local/<bucket-name>/minio-upgrade-test.txt
mc ls local/<bucket-name>/minio-upgrade-test.txt
mc cat local/<bucket-name>/minio-upgrade-test.txt
```

ลบไฟล์ทดสอบเมื่อยืนยันผลแล้ว

```bash
mc rm local/<bucket-name>/minio-upgrade-test.txt
rm -f /tmp/minio-upgrade-test.txt
```

### 7.5 ตรวจสอบ Console และ Application

- เข้าหน้า MinIO Console ได้ตามปกติ
- เห็น Bucket และ Object สำคัญครบถ้วน
- Application ที่เชื่อมต่อ MinIO สามารถอ่านและเขียนข้อมูลได้
- ไม่มี Error ใหม่ใน Log หลังเปิดใช้งาน

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 8 Upgrade MinIO Client

หากเครื่องมี `mc` ติดตั้งแบบ Binary แนะนำให้ Upgrade ให้เป็นรุ่นล่าสุดด้วย

```bash
mkdir -p /tmp/mc-upgrade
cd /tmp/mc-upgrade
curl -LO https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
./mc --version
```

ตรวจสอบ Path ของ `mc` เดิม และแทนที่ Binary

```bash
MC_BIN=$(readlink -f $(command -v mc))
cp -a "$MC_BIN" "$BACKUP_DIR/mc.old" 2>/dev/null || true
install -o root -g root -m 0755 /tmp/mc-upgrade/mc "$MC_BIN"
mc --version
```

พื้นที่สำหรับรูปภาพ:



## Checklist สรุปผล

| รายการตรวจสอบ | สถานะ | หมายเหตุ |
| --- | --- | --- |
| แจ้ง Maintenance Window แล้ว |  |  |
| ตรวจสอบ Version เดิมแล้ว |  |  |
| สำรอง Binary เดิมแล้ว |  |  |
| สำรอง Service และ Configuration แล้ว |  |  |
| Export Configuration ผ่าน `mc` แล้ว |  |  |
| สำรองข้อมูล Object หรือทำ Snapshot แล้ว |  |  |
| Download Binary รุ่นใหม่จากแหล่งทางการแล้ว |  |  |
| Stop Service สำเร็จ |  |  |
| แทนที่ Binary สำเร็จ |  |  |
| Start Service สำเร็จ |  |  |
| ตรวจสอบ Version ใหม่แล้ว |  |  |
| ทดสอบ Upload และ Download สำเร็จ |  |  |
| Application ใช้งานได้ตามปกติ |  |  |
| บันทึกหลักฐานหลัง Upgrade แล้ว |  |  |

## แนวทาง Rollback

หากพบปัญหาหลัง Upgrade ให้พิจารณาตามลำดับดังนี้

1. หาก Service Start ไม่ขึ้นทันทีหลังแทนที่ Binary และยังไม่มีการเปิดใช้งานระบบ ให้ Stop Service แล้วนำ Binary เดิมจาก Backup กลับมาใช้งาน
2. หากระบบเปิดใช้งาน Version ใหม่แล้วและมีการเขียนข้อมูลแล้ว ไม่ควร Downgrade โดยตรง ให้ใช้ VM Snapshot หรือ Storage Snapshot ที่ทำไว้ก่อน Upgrade
3. หากปัญหาเกิดจาก Configuration ให้กู้คืนไฟล์ Configuration จาก Backup แล้ว Restart Service

ตัวอย่างการนำ Binary เดิมกลับมาใช้

```bash
systemctl stop minio
MINIO_BIN=$(readlink -f $(command -v minio))
cp -a "$BACKUP_DIR/minio.old" "$MINIO_BIN"
chmod 0755 "$MINIO_BIN"
systemctl start minio
systemctl status minio --no-pager
```

## แหล่งอ้างอิง

- MinIO Server Download  
  https://dl.min.io/server/minio/release/linux-amd64/
- MinIO Client Download  
  https://dl.min.io/client/mc/release/linux-amd64/
- MinIO Documentation  
  https://docs.min.io/
