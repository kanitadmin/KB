---
title: Wazuh - คู่มือการ Upgrade แบบ Standalone
tags:
  - wazuh
  - siem
  - security
  - upgrade
  - คู่มือ
created: 2026-06-28
updated: 2026-06-29
---

# Wazuh - คู่มือการ Upgrade แบบ Standalone

> [!summary] ใช้เมื่อไร
> ใช้คู่มือนี้เมื่อต้อง Upgrade Wazuh แบบ Standalone หรือ All-in-one และต้องควบคุมความเสี่ยงของ SIEM, log ingestion, dashboard, agent visibility และหลักฐาน audit หลังดำเนินการ

## วัตถุประสงค์

เอกสารนี้ใช้เป็นแนวทางการ Upgrade Wazuh แบบ Standalone หรือ All-in-one ที่ติดตั้ง Wazuh Indexer, Wazuh Manager, Filebeat และ Wazuh Dashboard อยู่บนเครื่องเดียวกัน เพื่อให้สามารถดำเนินการได้อย่างเป็นลำดับ ลดความเสี่ยงจาก Downtime และมีหลักฐานตรวจสอบหลังดำเนินการ

## ขอบเขต

คู่มือนี้ครอบคลุมการ Upgrade Wazuh Central Components บน Linux ที่ติดตั้งด้วย Package Repository ได้แก่

- Wazuh Indexer
- Wazuh Manager
- Filebeat
- Wazuh Dashboard

คู่มือนี้ไม่ครอบคลุม Wazuh แบบ Docker, Kubernetes, Multi-node Cluster หรือการ Upgrade Wazuh Agent บนเครื่องปลายทาง

## ข้อกำหนดเบื้องต้น

- มีสิทธิ์ `root` หรือสิทธิ์ `sudo`
- มี Maintenance Window ที่ได้รับอนุมัติ
- มี Snapshot หรือ Backup ที่สามารถกู้คืนได้
- ทราบบัญชีผู้ดูแลระบบ Wazuh Indexer
- เครื่องสามารถเข้าถึง Wazuh Package Repository ได้
- ตรวจสอบ Release Notes และเอกสาร Upgrade Guide ของ Wazuh รุ่นเป้าหมายก่อนดำเนินการ

## ข้อมูลก่อนดำเนินการ

| รายการ | รายละเอียด |
| --- | --- |
| ชื่อเครื่อง |  |
| IP Address |  |
| Operating System |  |
| Wazuh Version ปัจจุบัน |  |
| Wazuh Version เป้าหมาย |  |
| ผู้ดำเนินการ |  |
| วันที่และเวลาเริ่มดำเนินการ |  |
| วันที่และเวลาสิ้นสุด |  |

พื้นที่สำหรับรูปภาพ:



## ข้อควรระวังสำคัญ

- Wazuh Central Components ต้องเป็น Version เดียวกัน รวมถึง Patch Level
- Wazuh Manager ต้องมี Version เท่ากับหรือใหม่กว่า Wazuh Agent
- เมื่อ Upgrade ไปยัง Wazuh 4.12.0 หรือใหม่กว่า จะไม่สามารถ Rollback ไปยัง 4.11 หรือเก่ากว่าได้โดยตรง เนื่องจากข้อจำกัดของ Apache Lucene
- ตั้งแต่ Wazuh 4.14.1 เป็นต้นไป Field ชื่อ `user` ใน JSON จะถูก Map เป็น `dstuser` อาจกระทบ Dashboard, Search, Script หรือ Integration ที่อ้างถึง `data.user`
- ห้ามเปิดเผยรหัสผ่าน, Token หรือข้อมูล Sensitive ในรูปภาพหรือเอกสารประกอบ

> [!warning] SIEM Visibility
> ระหว่าง upgrade อาจเกิดช่วงที่ alert, dashboard หรือ log ingestion หยุดชั่วคราว ต้องแจ้งทีม Security/IT Operations และระบุช่องทาง monitoring สำรองก่อนเริ่มงาน

## ความเสี่ยงและการควบคุม

| ความเสี่ยง | ผลกระทบ | การควบคุม |
| --- | --- | --- |
| Indexer health ไม่พร้อมก่อน upgrade | Upgrade ล้มเหลวหรือข้อมูลค้นหาไม่ได้ | ตรวจ `_cluster/health` และแก้เหตุ `yellow/red` ก่อนเริ่ม |
| Component version ไม่ตรงกัน | Dashboard หรือ agent data ผิดปกติ | Upgrade ตามลำดับและตรวจ version หลังทุก component |
| Saved objects หรือ dashboard หาย | สูญเสียมุมมองตรวจสอบสำคัญ | Export saved objects ก่อน upgrade |
| Repository เปิดค้าง | รอบ patch ถัดไปอาจ upgrade โดยไม่ตั้งใจ | ปิด repository หลังงานเสร็จ |
| Rollback ข้าม major/ข้อจำกัด Lucene ทำไม่ได้ | ต้อง restore snapshot เท่านั้น | เตรียม snapshot ก่อน upgrade และกำหนด decision point |

## จุดตัดสินใจก่อนดำเนินการต่อ

| จุดตรวจ | เงื่อนไขผ่าน | หากไม่ผ่าน |
| --- | --- | --- |
| ก่อน stop service | Backup, saved objects และ indexer health พร้อม | หยุดงานและแก้ปัญหาก่อน |
| หลัง upgrade indexer | Cluster health กลับมาใช้งานได้ | กู้ config หรือ restore snapshot ตามแผน |
| หลัง upgrade manager/filebeat | Agent data และ index ใหม่เข้าได้ | ตรวจ config, certificate และ filebeat output |
| ก่อนปิดงาน | Dashboard, alert และ agent visibility ปกติ | เปิด incident/change follow-up |

## ลำดับการ Upgrade

ให้ดำเนินการตามลำดับดังนี้

1. เตรียมระบบและสำรองข้อมูล
2. Stop Filebeat และ Wazuh Dashboard
3. Upgrade Wazuh Indexer
4. Upgrade Wazuh Manager
5. Upgrade และ Configure Filebeat
6. Upgrade Wazuh Dashboard
7. ตรวจสอบผลหลัง Upgrade
8. ปิด Wazuh Repository เพื่อป้องกันการ Upgrade โดยไม่ตั้งใจ

## ขั้นตอนที่ 1 เตรียมระบบและสำรองข้อมูล

### 1.1 ตรวจสอบสถานะ Service

```bash
systemctl status wazuh-indexer --no-pager
systemctl status wazuh-manager --no-pager
systemctl status filebeat --no-pager
systemctl status wazuh-dashboard --no-pager
```

### 1.2 ตรวจสอบ Version ปัจจุบัน

Debian หรือ Ubuntu:

```bash
apt list --installed wazuh-indexer wazuh-manager filebeat wazuh-dashboard
```

RHEL-compatible:

```bash
yum list installed wazuh-indexer wazuh-manager filebeat wazuh-dashboard
```

### 1.3 ตรวจสอบ Health ของ Wazuh Indexer

แทนค่า `<WAZUH_INDEXER_IP_ADDRESS>` และ `<USERNAME>` ให้ตรงกับระบบจริง

```bash
curl -k -u <USERNAME> https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cluster/health?pretty
curl -k -u <USERNAME> https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cat/nodes?v
```

ผลลัพธ์ควรเป็น `green` หรืออย่างน้อยต้องทราบสาเหตุหากเป็น `yellow` ก่อนเริ่ม Upgrade

พื้นที่สำหรับรูปภาพ:



### 1.4 สำรอง Configuration สำคัญ

สร้างโฟลเดอร์สำหรับเก็บไฟล์สำรอง

```bash
mkdir -p /root/wazuh-upgrade-backup-$(date +%F)
BACKUP_DIR=/root/wazuh-upgrade-backup-$(date +%F)
```

สำรอง Configuration

```bash
cp -a /etc/wazuh-indexer "$BACKUP_DIR/"
cp -a /var/ossec/etc "$BACKUP_DIR/ossec-etc"
cp -a /etc/filebeat "$BACKUP_DIR/"
cp -a /etc/wazuh-dashboard "$BACKUP_DIR/"
```

สำรอง Wazuh Indexer Security Configuration

```bash
/usr/share/wazuh-indexer/bin/indexer-security-init.sh --options "-backup /etc/wazuh-indexer/opensearch-security -icl -nhnv"
```

### 1.5 Export Dashboard Saved Objects

ดำเนินการผ่าน Wazuh Dashboard

1. เข้าเมนู `Dashboard management`
2. เลือก `Dashboards Management`
3. เลือก `Saved objects`
4. เลือก `Export all objects` หรือเลือกเฉพาะรายการที่ต้องการ
5. เก็บไฟล์ `.ndjson` ไว้ในพื้นที่ Backup

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 2 เปิดใช้งาน Wazuh Repository

หากเครื่องมี Wazuh Repository อยู่แล้ว ให้ตรวจสอบว่า Repository ยังใช้งานได้ก่อนดำเนินการ

Debian หรือ Ubuntu:

```bash
apt-get install gnupg apt-transport-https
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
apt-get update
```

RHEL-compatible:

```bash
rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
cat > /etc/yum.repos.d/wazuh.repo << 'EOF'
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
priority=1
EOF
```

## ขั้นตอนที่ 3 Stop Filebeat และ Wazuh Dashboard

```bash
systemctl stop filebeat
systemctl stop wazuh-dashboard
```

ตรวจสอบว่า Service หยุดแล้ว

```bash
systemctl is-active filebeat
systemctl is-active wazuh-dashboard
```

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 4 Upgrade Wazuh Indexer

### 4.1 ปรับ Shard Allocation และ Flush ข้อมูล

แทนค่า `<WAZUH_INDEXER_IP_ADDRESS>` และ `<USERNAME>` ให้ตรงกับระบบจริง

```bash
curl -X PUT "https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cluster/settings" -u <USERNAME> -k -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
'

curl -X POST "https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_flush" -u <USERNAME> -k
```

### 4.2 Stop Wazuh Manager และ Wazuh Indexer

```bash
systemctl stop wazuh-manager
systemctl stop wazuh-indexer
```

### 4.3 สำรอง JVM Options

```bash
cp /etc/wazuh-indexer/jvm.options /etc/wazuh-indexer/jvm.options.old
```

### 4.4 Upgrade Package

Debian หรือ Ubuntu:

```bash
apt-get install wazuh-indexer
```

RHEL-compatible:

```bash
yum upgrade wazuh-indexer
```

หากระบบถามการแทนที่ไฟล์ `/etc/wazuh-indexer/jvm.options` ให้เลือกใช้ไฟล์ใหม่ แล้วนำค่าที่ปรับแต่งไว้กลับมาใส่ภายหลังจากไฟล์ `.old`

### 4.5 Start Wazuh Indexer

```bash
systemctl daemon-reload
systemctl enable wazuh-indexer
systemctl start wazuh-indexer
```

### 4.6 Apply Security Configuration และเปิด Shard Allocation กลับ

```bash
/usr/share/wazuh-indexer/bin/indexer-security-init.sh

curl -X PUT "https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cluster/settings" -u <USERNAME> -k -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}
'
```

ตรวจสอบสถานะ

```bash
curl -k -u <USERNAME> https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cat/nodes?v
curl -k -u <USERNAME> https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cluster/health?pretty
```

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 5 Upgrade Wazuh Manager

Debian หรือ Ubuntu:

```bash
apt-get install wazuh-manager
```

RHEL-compatible:

```bash
yum upgrade wazuh-manager
```

Start Wazuh Manager

```bash
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
```

ตรวจสอบว่าค่า `<indexer>` ในไฟล์ `/var/ossec/etc/ossec.conf` ชี้ไปยัง Wazuh Indexer ที่ถูกต้อง ตัวอย่างเช่น

```xml
<indexer>
  <enabled>yes</enabled>
  <hosts>
    <host>https://127.0.0.1:9200</host>
  </hosts>
  <ssl>
    <certificate_authorities>
      <ca>/etc/filebeat/certs/root-ca.pem</ca>
    </certificate_authorities>
    <certificate>/etc/filebeat/certs/filebeat.pem</certificate>
    <key>/etc/filebeat/certs/filebeat-key.pem</key>
  </ssl>
</indexer>
```

หากมีการแก้ไขค่า ให้ Restart Wazuh Manager

```bash
systemctl restart wazuh-manager
```

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 6 Upgrade และ Configure Filebeat

กำหนด Version เป้าหมายให้ตรงกับ Wazuh รุ่นที่ต้องการ Upgrade

```bash
TARGET_VERSION="4.14.5"
```

Download Wazuh Filebeat Module และ Alerts Template

```bash
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.5.tar.gz | tar -xvz -C /usr/share/filebeat/module
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v${TARGET_VERSION}/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
```

สำรอง Filebeat Configuration

```bash
cp /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.old
```

Upgrade Filebeat

Debian หรือ Ubuntu:

```bash
apt-get install filebeat
```

RHEL-compatible:

```bash
yum upgrade filebeat
```

กู้คืน Filebeat Configuration ที่ใช้งานอยู่

```bash
cp /etc/filebeat/filebeat.yml.old /etc/filebeat/filebeat.yml
```

Start Filebeat และ Upload Template

```bash
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat

filebeat setup --pipelines
filebeat setup --index-management -E output.logstash.enabled=false
```

หมายเหตุ: หาก Upgrade จาก Wazuh 4.8.x หรือ 4.9.x ให้ตรวจสอบขั้นตอนการ Update Mapping ของ `wazuh-states-vulnerabilities-*` เพิ่มเติมจากเอกสารทางการของ Wazuh

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 7 Upgrade Wazuh Dashboard

สำรอง Configuration

```bash
cp /etc/wazuh-dashboard/opensearch_dashboards.yml /etc/wazuh-dashboard/opensearch_dashboards.yml.old
```

Upgrade Package

Debian หรือ Ubuntu:

```bash
apt-get install wazuh-dashboard
```

RHEL-compatible:

```bash
yum upgrade wazuh-dashboard
```

หากระบบถามการแทนที่ไฟล์ `/etc/wazuh-dashboard/opensearch_dashboards.yml` ให้เลือกใช้ไฟล์ใหม่ แล้วนำค่าที่ปรับแต่งไว้กลับมาใส่ภายหลังจากไฟล์ `.old`

Start Wazuh Dashboard

```bash
systemctl daemon-reload
systemctl enable wazuh-dashboard
systemctl start wazuh-dashboard
```

เข้าใช้งาน Dashboard

```text
https://<DASHBOARD_IP_ADDRESS>/app/wz-home
```

หากมีการ Export Saved Objects ไว้ ให้ Import กลับผ่านเมนู `Dashboard management > Dashboards Management > Saved objects`

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 8 ตรวจสอบหลัง Upgrade

### 8.1 ตรวจสอบ Service

```bash
systemctl status wazuh-indexer --no-pager
systemctl status wazuh-manager --no-pager
systemctl status filebeat --no-pager
systemctl status wazuh-dashboard --no-pager
```

### 8.2 ตรวจสอบ Version

Debian หรือ Ubuntu:

```bash
apt list --installed wazuh-indexer wazuh-manager filebeat wazuh-dashboard
```

RHEL-compatible:

```bash
yum list installed wazuh-indexer wazuh-manager filebeat wazuh-dashboard
```

### 8.3 ตรวจสอบ Wazuh Manager

```bash
/var/ossec/bin/wazuh-control info
/var/ossec/bin/agent_control -l
```

### 8.4 ตรวจสอบ Wazuh Indexer และ Filebeat

```bash
curl -k -u <USERNAME> https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cluster/health?pretty
curl -k -u <USERNAME> https://<WAZUH_INDEXER_IP_ADDRESS>:9200/_cat/indices/wazuh-*?v
filebeat test output
```

### 8.5 ตรวจสอบผ่าน Dashboard

- Login เข้า Wazuh Dashboard ได้
- เห็น Agent และสถานะล่าสุด
- เห็น Alert ใหม่หลัง Upgrade
- Dashboard หรือ Saved Objects สำคัญทำงานได้ตามปกติ

พื้นที่สำหรับรูปภาพ:



## ขั้นตอนที่ 9 ปิด Wazuh Repository หลัง Upgrade

แนะนำให้ปิด Wazuh Repository หลังดำเนินการเสร็จ เพื่อป้องกันการ Upgrade โดยไม่ตั้งใจในรอบ Patch ระบบครั้งถัดไป

Debian หรือ Ubuntu:

```bash
sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list
apt update
```

RHEL-compatible:

```bash
sed -i "s/^enabled=1/enabled=0/" /etc/yum.repos.d/wazuh.repo
```

## Checklist สรุปผล

| รายการตรวจสอบ | สถานะ | หมายเหตุ |
| --- | --- | --- |
| สำรองข้อมูลและ Configuration แล้ว |  |  |
| Export Dashboard Saved Objects แล้ว |  |  |
| Upgrade Wazuh Indexer สำเร็จ |  |  |
| Upgrade Wazuh Manager สำเร็จ |  |  |
| Upgrade Filebeat และ Upload Template สำเร็จ |  |  |
| Upgrade Wazuh Dashboard สำเร็จ |  |  |
| Service ทั้งหมดทำงานปกติ |  |  |
| Version ของ Central Components ตรงกัน |  |  |
| Dashboard ใช้งานได้ |  |  |
| Agent ส่งข้อมูลเข้า Wazuh ได้ |  |  |
| ปิด Wazuh Repository แล้ว |  |  |

## แนวทาง Rollback

หาก Upgrade ไม่สำเร็จ ให้ประเมินตามระดับผลกระทบดังนี้

1. หากเป็นปัญหา Configuration ให้กู้คืนไฟล์จาก Backup แล้ว Restart Service ที่เกี่ยวข้อง
2. หากเป็นปัญหาหลัง Upgrade Package และยังไม่กระทบข้อมูล Index ให้พิจารณา Restore จาก VM Snapshot หรือ Backup ระดับระบบ
3. หาก Upgrade ไปยัง Wazuh 4.12.0 หรือใหม่กว่าแล้ว ห้ามคาดหวังว่าจะ Downgrade Indexer กลับไปยัง 4.11 หรือเก่ากว่าได้โดยตรง ต้องใช้ Snapshot หรือ Fresh Installation ตามแนวทางของ Wazuh

## แนวทางแก้ปัญหาที่พบบ่อย

| อาการ | สาเหตุที่พบบ่อย | แนวทางตรวจสอบ |
| --- | --- | --- |
| Dashboard เข้าไม่ได้ | Service ไม่ขึ้น, certificate หรือ config ไม่ตรง | ตรวจ `systemctl status wazuh-dashboard` และ log ของ dashboard |
| Agent ไม่แสดงข้อมูลใหม่ | Wazuh Manager หรือ Filebeat มีปัญหา | ตรวจ `/var/ossec/bin/wazuh-control info`, `filebeat test output` |
| Indexer health เป็น `red` | Shard หรือ security config มีปัญหา | ตรวจ `_cluster/health`, `_cat/indices` และ log indexer |
| Login dashboard ไม่ได้ | Security config หรือ credential เปลี่ยน | ตรวจ saved config และ user/role ของ Wazuh Indexer |
| Field ใน dashboard หายหลัง upgrade | Mapping หรือ field name เปลี่ยน | ตรวจ release notes และ saved object ที่อ้าง field เดิม |

## หลักฐานที่ควรบันทึก

- Version ของ Wazuh components ก่อนและหลัง upgrade
- ผล `_cluster/health` ก่อนและหลัง upgrade
- ผล `systemctl status` ของทุก service หลัง upgrade
- ไฟล์ export saved objects หรือหลักฐานว่า export แล้ว
- Screenshot dashboard แสดง agent และ alert หลัง upgrade
- Ticket หรือ change request ที่อนุมัติ maintenance window

## แหล่งอ้างอิง

- Wazuh Documentation: Upgrade guide  
  https://documentation.wazuh.com/current/upgrade-guide/index.html
- Wazuh Documentation: Wazuh central components  
  https://documentation.wazuh.com/current/upgrade-guide/upgrading-central-components.html
