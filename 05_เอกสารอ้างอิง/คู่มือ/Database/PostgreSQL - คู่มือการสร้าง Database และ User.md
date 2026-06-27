---
title: PostgreSQL - คู่มือการสร้าง Database และ User
tags:
  - database
  - postgresql
  - คู่มือ
created: 2026-06-27
updated: 2026-06-27
---

# PostgreSQL - คู่มือการสร้าง Database และ User

## วัตถุประสงค์

เอกสารนี้ใช้สำหรับสร้างฐานข้อมูล PostgreSQL พร้อมสร้างผู้ใช้งาน และกำหนดสิทธิ์ให้ผู้ใช้งานสามารถจัดการฐานข้อมูลที่สร้างได้อย่างครบถ้วน

## ข้อกำหนดเบื้องต้น

- ต้องเข้าใช้งานด้วยผู้ใช้ที่มีสิทธิ์ `superuser` หรือมีสิทธิ์เพียงพอในการสร้าง role และ database
- ต้องกำหนดชื่อฐานข้อมูล ชื่อผู้ใช้ และรหัสผ่านให้เรียบร้อยก่อนดำเนินการ
- หลีกเลี่ยงการใช้รหัสผ่านตัวอย่างในระบบจริง

## ตัวอย่างค่าที่ใช้

| รายการ | ตัวอย่าง |
| --- | --- |
| ชื่อฐานข้อมูล | `app_db` |
| ชื่อผู้ใช้ | `app_user` |
| รหัสผ่าน | `ChangeThisStrongPassword` |
| Schema หลัก | `public` |

พื้นที่สำหรับรูปภาพ:



## วิธีที่แนะนำ

กรณีต้องการให้ผู้ใช้สามารถจัดการฐานข้อมูลที่สร้างได้ครบถ้วน ให้กำหนดผู้ใช้นั้นเป็น `OWNER` ของฐานข้อมูลตั้งแต่ขั้นตอนสร้างฐานข้อมูล

```sql
CREATE ROLE app_user
WITH
  LOGIN
  PASSWORD 'ChangeThisStrongPassword';

CREATE DATABASE app_db
WITH
  OWNER = app_user
  ENCODING = 'UTF8';
```

พื้นที่สำหรับรูปภาพ:



## กำหนดสิทธิ์ภายในฐานข้อมูล

หลังจากสร้างฐานข้อมูลแล้ว ให้เชื่อมต่อเข้าไปยังฐานข้อมูลที่สร้าง

```sql
\c app_db
```

จากนั้นกำหนดสิทธิ์บน database และ schema

```sql
GRANT ALL PRIVILEGES ON DATABASE app_db TO app_user;
GRANT USAGE, CREATE ON SCHEMA public TO app_user;
ALTER SCHEMA public OWNER TO app_user;
```

พื้นที่สำหรับรูปภาพ:



กำหนดสิทธิ์สำหรับ object ที่มีอยู่แล้วใน schema

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO app_user;
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO app_user;
```

กำหนดสิทธิ์สำหรับ object ที่จะถูกสร้างในอนาคต

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT ALL PRIVILEGES ON TABLES TO app_user;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT ALL PRIVILEGES ON SEQUENCES TO app_user;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT ALL PRIVILEGES ON FUNCTIONS TO app_user;
```

พื้นที่สำหรับรูปภาพ:



## คำสั่งตรวจสอบ

ตรวจสอบรายการฐานข้อมูล

```sql
\l
```

ตรวจสอบ schema

```sql
\dn+
```

ตรวจสอบสิทธิ์ของตาราง

```sql
\dp
```

ตรวจสอบการเชื่อมต่อด้วยผู้ใช้ที่สร้าง

```bash
psql -h <host> -U app_user -d app_db
```

พื้นที่สำหรับรูปภาพ:



## หมายเหตุสำคัญ

- การเป็น `OWNER` ของ database เป็นวิธีที่เหมาะสมที่สุดเมื่อต้องการให้ผู้ใช้ควบคุมฐานข้อมูลที่สร้างขึ้น
- คำสั่ง `ALTER DEFAULT PRIVILEGES` มีผลกับ object ที่จะสร้างหลังจากรันคำสั่งเท่านั้น
- หาก object ถูกสร้างโดยผู้ใช้อื่น ต้องกำหนด default privileges จากผู้ใช้นั้นเพิ่มเติม หรือกำหนดสิทธิ์ให้ภายหลัง
- ไม่ควรใช้สิทธิ์ `superuser` กับผู้ใช้ของแอปพลิเคชัน ยกเว้นมีเหตุผลและได้รับอนุมัติอย่างชัดเจน

## แหล่งอ้างอิง

- PostgreSQL Documentation: CREATE ROLE  
  https://www.postgresql.org/docs/current/sql-createrole.html
- PostgreSQL Documentation: CREATE DATABASE  
  https://www.postgresql.org/docs/current/sql-createdatabase.html
- PostgreSQL Documentation: GRANT  
  https://www.postgresql.org/docs/current/sql-grant.html
- PostgreSQL Documentation: ALTER DEFAULT PRIVILEGES  
  https://www.postgresql.org/docs/current/sql-alterdefaultprivileges.html
