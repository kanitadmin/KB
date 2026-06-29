---
title: PostgreSQL - คู่มือการสร้าง Database และ User
tags:
  - database
  - postgresql
  - คู่มือ
created: 2026-06-27
updated: 2026-06-29
---

# PostgreSQL - คู่มือการสร้าง Database และ User

> [!summary] ใช้เมื่อไร
> ใช้คู่มือนี้เมื่อต้องสร้างฐานข้อมูล PostgreSQL ใหม่ พร้อมสร้าง role สำหรับแอปพลิเคชันหรือบริการ และต้องการให้ role นั้นจัดการฐานข้อมูลของตัวเองได้อย่างถูกต้อง โดยไม่ต้องให้สิทธิ์ `superuser`

## เป้าหมาย

- สร้าง role สำหรับใช้งานฐานข้อมูล เช่น `app_user`
- สร้าง database ใหม่ เช่น `app_db`
- กำหนด owner และสิทธิ์บน schema ให้พร้อมใช้งาน
- ตรวจสอบว่า role ที่สร้างสามารถเชื่อมต่อและสร้าง object ได้
- ลดความเสี่ยงจากการใช้บัญชีผู้ดูแลระบบกับแอปพลิเคชัน

## ขอบเขต

| ครอบคลุม | ไม่ครอบคลุม |
| --- | --- |
| การสร้าง role, database, schema privilege และ default privilege | การออกแบบ HA, replication, backup policy, performance tuning |
| การตรวจสอบสิทธิ์หลังดำเนินการ | การ migrate ข้อมูลจากระบบเดิม |
| แนวทาง rollback กรณีเพิ่งสร้างผิด | การ hardening PostgreSQL ทั้งระบบ |

## ข้อกำหนดเบื้องต้น

- มีสิทธิ์ `superuser` หรือสิทธิ์เพียงพอในการ `CREATE ROLE` และ `CREATE DATABASE`
- ทราบชื่อ database, role, host, port และ schema ที่ต้องใช้งาน
- มี maintenance window หรือการอนุมัติ หากเป็นระบบ production
- ห้ามใช้รหัสผ่านตัวอย่างในระบบจริง และไม่ควรบันทึกรหัสผ่านลงในเอกสารหรือ repository

> [!warning] ข้อควรระวัง
> ผู้ใช้ของแอปพลิเคชันไม่ควรเป็น `superuser` ยกเว้นมีเหตุผลทางเทคนิคที่ชัดเจน ได้รับอนุมัติ และมีการบันทึกความเสี่ยงไว้แล้ว

## ข้อมูลที่ต้องเตรียม

| รายการ | ตัวอย่าง | หมายเหตุ |
| --- | --- | --- |
| Host | `<postgres-host>` | ใช้ชื่อ DNS หรือ IP ของ PostgreSQL Server |
| Port | `5432` | ค่าเริ่มต้นของ PostgreSQL |
| Database | `app_db` | ใช้ชื่อที่สื่อถึงระบบหรือบริการ |
| Role/User | `app_user` | ใช้เฉพาะระบบนี้ ไม่ใช้ร่วมหลายระบบ |
| Password | `<strong-password>` | เก็บใน password manager หรือ secret manager |
| Schema หลัก | `public` | เปลี่ยนได้ตามมาตรฐานของระบบ |
| ผู้ขอใช้บริการ |  | ระบุทีม/ระบบ/เจ้าของงาน |
| วันที่ดำเนินการ |  | ใช้สำหรับบันทึกหลักฐาน |

## แนวทางที่แนะนำ

กรณีต้องการให้ role จัดการฐานข้อมูลของตัวเองได้ครบถ้วน ให้กำหนด role นั้นเป็น `OWNER` ของ database ตั้งแต่ตอนสร้าง database

```sql
CREATE ROLE app_user
WITH
  LOGIN
  PASSWORD '<strong-password>'
  NOSUPERUSER
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

CREATE DATABASE app_db
WITH
  OWNER = app_user
  ENCODING = 'UTF8';
```

> [!tip] สำหรับ production
> หากต้องการแยกสิทธิ์ให้เข้มงวดกว่าเดิม ให้พิจารณาแยก `owner role` แบบ `NOLOGIN` ออกจาก `runtime role` ที่แอปพลิเคชันใช้เชื่อมต่อจริง

## ขั้นตอนดำเนินการ

### 1. เข้าใช้งาน PostgreSQL ด้วยบัญชีผู้ดูแล

บน Linux server ที่ติดตั้ง PostgreSQL:

```bash
sudo -iu postgres psql
```

หรือเชื่อมต่อจากเครื่องผู้ดูแลระบบ:

```bash
psql -h <postgres-host> -p 5432 -U postgres -d postgres
```

### 2. ตรวจสอบว่าชื่อยังไม่ถูกใช้งาน

```sql
\du app_user

SELECT datname
FROM pg_database
WHERE datname = 'app_db';
```

ถ้ามี role หรือ database ชื่อนี้อยู่แล้ว ให้หยุดและยืนยันกับเจ้าของระบบก่อนดำเนินการต่อ

### 3. สร้าง role สำหรับแอปพลิเคชัน

```sql
CREATE ROLE app_user
WITH
  LOGIN
  PASSWORD '<strong-password>'
  NOSUPERUSER
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;
```

### 4. สร้าง database และกำหนด owner

```sql
CREATE DATABASE app_db
WITH
  OWNER = app_user
  ENCODING = 'UTF8';
```

### 5. กำหนดสิทธิ์ภายใน database

เชื่อมต่อเข้า database ที่เพิ่งสร้าง:

```sql
\c app_db
```

กำหนด owner และสิทธิ์บน schema หลัก:

```sql
ALTER SCHEMA public OWNER TO app_user;
GRANT USAGE, CREATE ON SCHEMA public TO app_user;
GRANT ALL PRIVILEGES ON DATABASE app_db TO app_user;
```

### 6. กำหนดสิทธิ์ให้ object ที่มีอยู่แล้ว

หาก database ไม่ได้ว่าง หรือมีการสร้าง table, sequence, function ไว้ก่อนแล้ว ให้กำหนดสิทธิ์เพิ่มเติม:

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO app_user;
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO app_user;
```

### 7. กำหนดสิทธิ์เริ่มต้นสำหรับ object ที่จะสร้างในอนาคต

ถ้า object ใน schema นี้จะถูกสร้างโดย role อื่น เช่น migration user หรือ admin user ให้ระบุ role ผู้สร้าง object ให้ชัดเจน:

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE <object_creator_role> IN SCHEMA public
GRANT ALL PRIVILEGES ON TABLES TO app_user;

ALTER DEFAULT PRIVILEGES FOR ROLE <object_creator_role> IN SCHEMA public
GRANT ALL PRIVILEGES ON SEQUENCES TO app_user;

ALTER DEFAULT PRIVILEGES FOR ROLE <object_creator_role> IN SCHEMA public
GRANT ALL PRIVILEGES ON FUNCTIONS TO app_user;
```

> [!note] เรื่อง default privileges
> `ALTER DEFAULT PRIVILEGES` มีผลเฉพาะ object ที่จะถูกสร้างหลังจากรันคำสั่งเท่านั้น และผูกกับ role ผู้สร้าง object ไม่ได้ย้อนหลังไปแก้ object เก่า

## คำสั่งตรวจสอบ

ตรวจสอบ database:

```sql
\l app_db
```

ตรวจสอบ role:

```sql
\du app_user
```

ตรวจสอบ schema:

```sql
\dn+
```

ตรวจสอบ privilege ของ object:

```sql
\dp
```

ทดสอบการเชื่อมต่อด้วย role ที่สร้าง:

```bash
psql -h <postgres-host> -p 5432 -U app_user -d app_db
```

ทดสอบสร้างและลบ table:

```sql
CREATE TABLE public.permission_test (
  id integer PRIMARY KEY
);

DROP TABLE public.permission_test;
```

## Checklist หลังดำเนินการ

| รายการตรวจสอบ | สถานะ | หมายเหตุ |
| --- | --- | --- |
| Role ถูกสร้างแล้ว |  |  |
| Database ถูกสร้างแล้ว |  |  |
| Database owner ถูกต้อง |  |  |
| Schema owner หรือ privilege ถูกต้อง |  |  |
| Role เชื่อมต่อเข้า database ได้ |  |  |
| Role สร้างและลบ object ทดสอบได้ |  |  |
| ไม่มีการให้สิทธิ์ `superuser` โดยไม่จำเป็น |  |  |
| บันทึกหลักฐานหรือ ticket แล้ว |  |  |

## แนวทางแก้ปัญหาที่พบบ่อย

| อาการ | สาเหตุที่พบบ่อย | แนวทางตรวจสอบ |
| --- | --- | --- |
| `permission denied for schema public` | ยังไม่ได้ให้ `USAGE, CREATE` หรือ schema owner ไม่ถูกต้อง | ตรวจสอบด้วย `\dn+` แล้วรัน `GRANT USAGE, CREATE ON SCHEMA public TO app_user;` |
| `database "app_db" already exists` | มี database ชื่อนี้อยู่แล้ว | ตรวจสอบเจ้าของด้วย `\l app_db` ก่อนตัดสินใจใช้ชื่อใหม่หรือแก้ owner |
| `role "app_user" already exists` | มี role ชื่อนี้อยู่แล้ว | ตรวจสอบด้วย `\du app_user` และยืนยันว่าเป็น role ของระบบเดียวกันหรือไม่ |
| `password authentication failed` | รหัสผ่านไม่ถูกต้อง หรือ `pg_hba.conf` ไม่อนุญาตวิธี auth ที่ใช้ | ตรวจสอบ password, host, user, database และ policy ของ `pg_hba.conf` |
| ใช้คำสั่ง `ALTER DEFAULT PRIVILEGES` แล้ว object ใหม่ยังไม่มีสิทธิ์ | รันคำสั่งกับ role ผู้สร้าง object ไม่ถูกตัว | ใช้ `FOR ROLE <object_creator_role>` ให้ตรงกับ role ที่สร้าง object จริง |

## Rollback กรณีเพิ่งสร้างผิด

ใช้เฉพาะกรณีที่เพิ่งสร้าง database/role ผิดและยืนยันแล้วว่ายังไม่มีข้อมูลสำคัญ

```sql
\c postgres

REVOKE CONNECT ON DATABASE app_db FROM PUBLIC;

SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'app_db'
  AND pid <> pg_backend_pid();

DROP DATABASE app_db;
DROP ROLE app_user;
```

> [!danger] ห้ามใช้กับระบบที่มีข้อมูลจริงโดยไม่ตรวจสอบ
> ก่อนลบ database ใน production ต้องยืนยันเจ้าของระบบ, backup, ticket อนุมัติ และผลกระทบต่อแอปพลิเคชันทุกครั้ง

## หลักฐานที่ควรบันทึก

- Screenshot หรือผลลัพธ์ของ `\l app_db`
- Screenshot หรือผลลัพธ์ของ `\du app_user`
- Screenshot หรือผลลัพธ์ของ `\dn+`
- ผลการทดสอบเชื่อมต่อด้วย `app_user`
- Ticket หรือ change request ที่อนุมัติให้ดำเนินการ

## หมายเหตุสำคัญ

- การเป็น `OWNER` ของ database เป็นวิธีที่เหมาะสมเมื่อ role ต้องควบคุมฐานข้อมูลที่สร้างขึ้นเอง
- ไม่ควรใช้บัญชีผู้ดูแลระบบหรือ `postgres` เป็น connection user ของแอปพลิเคชัน
- ถ้ามีหลายทีมสร้าง object ใน database เดียวกัน ให้กำหนด owner, migration role และ runtime role ให้ชัดเจนตั้งแต่แรก
- หากต้องทำใน production ให้มี rollback plan และหลักฐานตรวจสอบหลังดำเนินการเสมอ

## แหล่งอ้างอิง

- PostgreSQL Documentation: CREATE ROLE  
  https://www.postgresql.org/docs/current/sql-createrole.html
- PostgreSQL Documentation: CREATE DATABASE  
  https://www.postgresql.org/docs/current/sql-createdatabase.html
- PostgreSQL Documentation: GRANT  
  https://www.postgresql.org/docs/current/sql-grant.html
- PostgreSQL Documentation: ALTER DEFAULT PRIVILEGES  
  https://www.postgresql.org/docs/current/sql-alterdefaultprivileges.html
