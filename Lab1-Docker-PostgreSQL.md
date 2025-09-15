<img width="579" height="515" alt="image" src="https://github.com/user-attachments/assets/7c71882a-3bac-4f50-85d2-7a892214fb26" /><img width="939" height="374" alt="image" src="https://github.com/user-attachments/assets/5be870ba-6539-4749-b1d4-09e90ae92b67" /># Lab 01: PostgreSQL Docker Setup and Basic Operations

## วัตถุประสงค์
1. ติดตั้งและใช้งาน PostgreSQL ผ่าน Docker
2. อธิบายโครงสร้างพื้นฐานของ PostgreSQL Database Cluster
3. สร้าง และจัดการ Database และ User
4. ใช้ Docker Volume เพื่อความคงทนของข้อมูล


## ทฤษฎีก่อนการทดลอง

### 1. Database Cluster Architecture

PostgreSQL Database Cluster คือกลุ่มของฐานข้อมูลที่ถูกบริหารจัดการโดย PostgreSQL server หนึ่งตัว โครงสร้างประกอบด้วย:

#### Logical Structure
- **Database Cluster**: กลุ่มของฐานข้อมูลที่ถูกบริหารโดย PostgreSQL server หนึ่งตัว
- **Database**: การแยกแยะข้อมูลเป็นกลุ่มที่เกี่ยวข้องกัน แต่ละฐานข้อมูลมีชื่อที่ไม่ซ้ำกัน
- **Schema**: คือ namespace ที่ประกอบด้วย tables, views, indexes, functions, stored procedures
- **Tables/Objects**: วัตถุต่างๆ ในฐานข้อมูลที่เก็บข้อมูลจริง

#### Physical Structure
- **Tablespace**: ตำแหน่งบนดิสก์ที่ใช้เก็บไฟล์ข้อมูล (default: pg_default และ pg_global)
- **Data Directory**: โฟลเดอร์หลักที่เก็บไฟล์ทั้งหมดของ database cluster
- **WAL (Write-Ahead Log)**: ไฟล์บันทึกการเปลี่ยนแปลงเพื่อการ recovery

### 2. PostgreSQL Memory Architecture

#### Shared Memory Components
- **Shared Buffers**: หน่วยความจำส่วนกลางสำหรับแคชข้อมูล (ควรเป็น 25% ของ RAM)
- **WAL Buffers**: พื้นที่เก็บ Write-Ahead Log ก่อนเขียนลงดิสก์
- **CLog Buffer**: เก็บรายการ transaction ที่ทำงานอยู่

#### Process Memory Components
- **Work Memory**: หน่วยความจำสำหรับการ sort และ hash operations
- **Maintenance Work Memory**: ใช้สำหรับงาน maintenance เช่น VACUUM, REINDEX

### 3. Docker และ Volume Management

#### Docker Benefits
- **Isolation**: แยกสภาพแวดล้อมจากระบบหลัก
- **Consistency**: สภาพแวดล้อมเหมือนกันทุกเครื่อง
- **Easy Management**: ง่ายต่อการติดตั้ง backup และ restore
- **Version Control**: จัดการหลายเวอร์ชันได้ง่าย

#### Docker Volume Types
- **Named Volumes**: จัดการโดย Docker, เหมาะสำหรับ production
- **Bind Mounts**: เชื่อมโฟลเดอร์จากระบบโดยตรง
- **tmpfs Mounts**: เก็บข้อมูลใน memory (ไม่คงทน)

### 4. User และ Role Management

#### PostgreSQL Security Model
- **Roles**: ผู้ใช้งานหรือกลุ่มผู้ใช้งาน
- **Privileges**: สิทธิ์ในการเข้าถึงวัตถุต่างๆ
- **Authentication**: การยืนยันตัวตน
- **Authorization**: การควบคุมสิทธิ์การเข้าถึง

## การเตรียมความพร้อม

### ติดตั้ง Docker
```bash
# สำหรับ Windows/Mac: ดาวน์โหลด Docker Desktop
# สำหรับ Linux (Ubuntu):
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker

# เพิ่ม user ปัจจุบันเข้า docker group (Linux)
sudo usermod -aG docker $USER
newgrp docker
```

### ตรวจสอบการติดตั้ง Docker
```bash
docker --version
docker run hello-world
```

**บันทึกผลการทดลอง - การเตรียมความพร้อม:**
```
<img width="573" height="327" alt="image" src="https://github.com/user-attachments/assets/08656b1d-ea04-4d95-8e4e-ca2deb8f6c54" />
<img width="279" height="49" alt="image" src="https://github.com/user-attachments/assets/d97acea0-027f-4557-8fcf-335678f47039" />

```

## ขั้นตอนการทดลอง

### Step 1: Pull PostgreSQL Docker Image

```bash
# ดาวน์โหลด PostgreSQL Image (Latest เวอร์ชัน )
docker pull postgres
-- กรณีระบุเวอร์ชั่นที่ต้องการ เช่น 16.3 ให้ระบุดังนี้ docker pull postgres:16.3

# ตรวจสอบ Images ที่มี
docker images

# ตรวจสอบรายละเอียดของ Image
docker inspect postgres

-- กรณีระบุเวอร์ชั่น ใช้คำสั่ง docker inspect postgres:16.3
```


**บันทึกผลการทดลอง - Step 1:**
```
<img width="457" height="141" alt="image" src="https://github.com/user-attachments/assets/dd371b85-4dc4-4c82-9e8e-d92217379a1d" />
```

### Step 2: Create Docker Volume for Data Persistence

```bash
# สร้าง Named Volume สำหรับเก็บข้อมูล
docker volume create postgres-data

# ตรวจสอบ Volume ที่สร้าง
docker volume ls

# ดูรายละเอียดของ Volume
docker volume inspect postgres-data

# สร้าง Volume สำหรับ configuration files
docker volume create postgres-config
```

**คำอธิบาย**: Docker Volume จะทำให้ข้อมูลคงอยู่แม้ Container จะถูกลบ

**บันทึกผลการทดลอง - Step 2:**
```
<img width="554" height="319" alt="image" src="https://github.com/user-attachments/assets/e163959e-b7c4-4203-8ad3-da476c561cb7" />

```

### Step 3: Create PostgreSQL Container with Volume

```bash
# สร้างและรัน PostgreSQL Container พร้อม Volume
docker run --name postgres-lab \
  -e POSTGRES_PASSWORD=admin123 \
  -e POSTGRES_DB=testdb \
  -e POSTGRES_USER=postgres \
  -v postgres-data:/var/lib/postgresql/data \
  -v postgres-config:/etc/postgresql \
  -p 5432:5432 \
  --memory="1g" \
  --cpus="1.0" \
  -d postgres \
  -c shared_buffers=256MB \
  -c work_mem=16MB \
  -c maintenance_work_mem=128MB
```

**คำอธิบายพารามิเตอร์**:
- `--name postgres-lab`: ตั้งชื่อ Container
- `-e POSTGRES_PASSWORD=admin123`: กำหนดรหัสผ่าน postgres user
- `-e POSTGRES_DB=testdb`: สร้างฐานข้อมูล testdb อัตโนมัติ
- `-v postgres-data:/var/lib/postgresql/data`: Mount Volume สำหรับข้อมูล
- `-p 5432:5432`: Map port 5432 ของ Container กับ Host
- `--memory="1g"`: จำกัดการใช้ RAM
- `--cpus="1.0"`: จำกัดการใช้ CPU
- `-c shared_buffers=256MB`: กำหนด shared buffers

**บันทึกผลการทดลอง - Step 3:**
```
<img width="920" height="125" alt="image" src="https://github.com/user-attachments/assets/65a3ad92-9afa-4a98-a5c0-b177e280b8e7" />

```

### Step 4: Verify Container Status and Resource Usage

```bash
# ตรวจสอบสถานะ Container
docker ps

# ดู logs ของ Container
docker logs postgres-lab

# ตรวจสอบการใช้ resources
docker stats postgres-lab --no-stream

# ตรวจสอบข้อมูลใน Volume
docker volume inspect postgres-data
```

**บันทึกผลการทดลอง - Step 4:**
```
ใส่ Screenshot ของ:
1. <img width="916" height="125" alt="image" src="https://github.com/user-attachments/assets/80e94d7d-a483-45e9-9669-8a5688dd2e25" />
2. <img width="853" height="316" alt="image" src="https://github.com/user-attachments/assets/e1c1b207-cb4b-487c-bbac-7053d27fa087" />
    <img width="886" height="322" alt="image" src="https://github.com/user-attachments/assets/7fa37778-0e34-4a6a-9db7-746c9614c079" />
<img width="910" height="318" alt="image" src="https://github.com/user-attachments/assets/bcd9b788-e5e2-49b7-aae6-b506d2ebe6c4" />
<img width="919" height="315" alt="image" src="https://github.com/user-attachments/assets/822ab157-4a40-4d94-a07f-3131fb948611" />
3. <img width="616" height="247" alt="image" src="https://github.com/user-attachments/assets/b6e1f6d0-e8f3-4aeb-984e-c4c192b2ec31" />
```

### Step 5: Connect to PostgreSQL และตรวจสอบ Configuration

```bash
# เชื่อมต่อผ่าน psql ใน Container
docker exec -it postgres-lab psql -U postgres
```
**คำอธิบายพารามิเตอร์**:
docker exec คำสั่งนี้ใช้เพื่อ รันคำสั่งภายในคอนเทนเนอร์ Docker ที่กำลังทำงานอยู่ 
โดยสั่งให้คอนเทนเนอร์ที่ชื่อ postgres-lab เปิดโปรแกรม psql เพื่อเชื่อมต่อฐานข้อมูล PostgreSQL ในฐานะผู้ใช้ postgres
-i (ย่อมาจาก --interactive): ทำให้ input (STDIN) เปิดอยู่ตลอดเวลา
-t (ย่อมาจาก --tty): จัดสรรเทอร์มินัลเสมือน (pseudo-TTY) ทำให้เราสามารถโต้ตอบกับคอนเทนเนอร์ได้เหมือนกับการใช้เทอร์มินัลปกติบนคอมพิวเตอร์

```sql
-- ตรวจสอบเวอร์ชัน PostgreSQL
SELECT version();

-- ตรวจสอบ Memory Configuration
SHOW shared_buffers;
SHOW work_mem;
SHOW maintenance_work_mem;
SHOW effective_cache_size;

-- ตรวจสอบการตั้งค่าทั้งหมด
SELECT name, setting, unit, short_desc 
FROM pg_settings 
WHERE name IN ('shared_buffers', 'work_mem', 'maintenance_work_mem', 'effective_cache_size');

-- ตรวจสอบข้อมูลระบบ
\conninfo

-- ดูรายการฐานข้อมูลทั้งหมด
\l

-- ดูรายการ User/Role
\du
```

**บันทึกผลการทดลอง - Step 5:**
```
ใส่ Screenshot ของ:
1. <img width="429" height="232" alt="image" src="https://github.com/user-attachments/assets/0732c334-2ff9-4ca9-afef-ead041417db5" />
;
2.<img width="285" height="105" alt="image" src="https://github.com/user-attachments/assets/0054df8d-b0c6-498f-bd9c-b8239409cf7c" />
maintenance_work_mem;SHOW effective_cache_size;
3. <img width="424" height="302" alt="image" src="https://github.com/user-attachments/assets/2351533a-96fd-46e8-a41b-55e3d1d16a24" />
<img width="418" height="322" alt="image" src="https://github.com/user-attachments/assets/7af30ed5-c319-4d14-a3fc-810af4580971" />
<img width="440" height="209" alt="image" src="https://github.com/user-attachments/assets/4ba9be12-d0de-4271-9200-e80cbabbb692" />


```

### Step 6: Database Management Operations

```sql
-- สร้างฐานข้อมูลใหม่พร้อม configuration
CREATE DATABASE lab_db
    WITH OWNER = postgres
         ENCODING = 'UTF8'
         TABLESPACE = pg_default
         CONNECTION LIMIT = -1
         TEMPLATE template1;

-- ตรวจสอบฐานข้อมูลที่สร้าง
\l

-- เชื่อมต่อไปยังฐานข้อมูลใหม่
\c lab_db

-- ตรวจสอบ OID และข้อมูลของฐานข้อมูล
SELECT 
    datname,
    oid,
    datdba,
    encoding,
    datcollate,
    datctype,
    datconnlimit
FROM pg_database 
WHERE datname = 'lab_db';

-- กลับไปยัง postgres database
\c postgres

-- ลบฐานข้อมูล (แสดงตัวอย่าง - ไม่ต้องรัน)
-- DROP DATABASE lab_db;
```

**บันทึกผลการทดลอง - Step 6:**
```
ใส่ Screenshot ของ:
1. <img width="405" height="313" alt="image" src="https://github.com/user-attachments/assets/4ff828fe-0783-486b-ac22-80be8499c09e" />
2. <img width="371" height="99" alt="image" src="https://github.com/user-attachments/assets/9d09ca74-ff82-433c-a22f-ede1834e14c4" />
3. <img width="394" height="163" alt="image" src="https://github.com/user-attachments/assets/6511c41b-2fbe-4b59-a958-863e2d2c6440" />

```

### Step 7: User และ Role Management

```sql
-- สร้าง Role พื้นฐาน
CREATE ROLE lab_role;

-- สร้าง User ธรรมดา
CREATE USER lab_user WITH 
    PASSWORD 'user123'
    LOGIN
    NOSUPERUSER
    NOCREATEDB
    NOCREATEROLE
    NOINHERIT
    NOREPLICATION
    CONNECTION LIMIT 10;

-- สร้าง User ที่มีสิทธิ์มากขึ้น
CREATE USER admin_user WITH 
    PASSWORD 'admin123' 
    SUPERUSER 
    CREATEDB 
    CREATEROLE
    LOGIN;

-- สร้าง Database Admin User
CREATE USER db_admin WITH
    PASSWORD 'dbadmin123'
    CREATEDB
    CREATEROLE
    LOGIN
    NOSUPERUSER;

-- ตรวจสอบ Users ที่สร้าง
\du+

-- ดู Role membership
SELECT 
    r.rolname,
    r.rolsuper,
    r.rolinherit,
    r.rolcreaterole,
    r.rolcreatedb,
    r.rolcanlogin,
    r.rolreplication,
    r.rolconnlimit
FROM pg_roles r
WHERE r.rolname NOT LIKE 'pg_%';
```

**บันทึกผลการทดลอง - Step 7:**
```
ใส่ Screenshot ของ:
1. <img width="385" height="327" alt="image" src="https://github.com/user-attachments/assets/ae7410a2-ba62-434d-8ba7-768999cf472a" />
2. <img width="432" height="318" alt="image" src="https://github.com/user-attachments/assets/4adad5ab-45e8-409d-9042-77a22a1aa6b5" />
3. <img width="386" height="314" alt="image" src="https://github.com/user-attachments/assets/e698e401-3188-4417-b5ba-7a915e385d34" />

```

### Step 8: การจัดการสิทธิ์ User

```sql
-- เปลี่ยนสิทธิ์ของ User
ALTER USER lab_user CREATEDB;
ALTER USER lab_user NOCREATEROLE;

-- ให้สิทธิ์ในระดับฐานข้อมูล
GRANT CONNECT ON DATABASE lab_db TO lab_user;
GRANT CONNECT ON DATABASE testdb TO lab_user;

-- สร้างตารางทดสอบใน lab_db
\c lab_db

CREATE TABLE test_permissions (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ใส่ข้อมูลทดสอบ
INSERT INTO test_permissions (name) VALUES 
    ('Test User 1'),
    ('Test User 2'),
    ('Test User 3');

-- ให้สิทธิ์ในระดับตาราง
GRANT SELECT ON test_permissions TO lab_user;
GRANT INSERT, UPDATE ON test_permissions TO db_admin;

-- ตรวจสอบสิทธิ์ของตาราง
\dp test_permissions

-- ให้สิทธิ์ในระดับ Schema
GRANT USAGE ON SCHEMA public TO lab_user;
GRANT ALL ON SCHEMA public TO db_admin;

-- สร้างตารางทดสอบเพิ่มเติมใน postgres database
\c postgres

CREATE TABLE postgres_test_table (
    id SERIAL PRIMARY KEY,
    description VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO postgres_test_table (description) VALUES 
    ('Test in postgres database'),
    ('Another test record');

-- ให้สิทธิ์สำหรับตารางใน postgres database
GRANT SELECT ON postgres_test_table TO lab_user;
```

**บันทึกผลการทดลอง - Step 8:**
```
ใส่ Screenshot ของ:
1. <img width="361" height="160" alt="image" src="https://github.com/user-attachments/assets/22bed16d-fe79-4928-b526-fc499132dac1" />
2.<img width="410" height="213" alt="image" src="https://github.com/user-attachments/assets/ef69e3bf-8651-4a54-9981-5bdae341289a" />
3. <img width="469" height="160" alt="image" src="https://github.com/user-attachments/assets/c77851d9-b188-4da3-8c02-bc1c15b9dd70" />
```
**คำถาม
 ```
postgres=arwdDxtm/postgres หมายความว่า user postgres ได้รับสิทธิ์เต็มทุกอย่างที่ทำได้กับตาราง โดยสิทธิ์นี้ถูก มอบโดย user postgres


 ```
### Step 9: Schema Management และ Namespace

```sql
-- เชื่อมต่อไปยัง lab_db
\c lab_db

-- สร้าง Schemas ต่างๆ
CREATE SCHEMA sales AUTHORIZATION postgres;
CREATE SCHEMA hr AUTHORIZATION postgres;  
CREATE SCHEMA inventory AUTHORIZATION postgres;
CREATE SCHEMA finance AUTHORIZATION db_admin;
-- สร้าง SCHEMA และกำหนดเจ้าของ SCHEMA 
-- ตรวจสอบ Schemas
\dn+

-- สร้างตารางใน Schema ต่างๆ
CREATE TABLE sales.customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE sales.orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES sales.customers(customer_id),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10,2)
);

CREATE TABLE hr.employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE DEFAULT CURRENT_DATE
);

CREATE TABLE hr.departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) UNIQUE NOT NULL,
    manager_id INTEGER
);

CREATE TABLE inventory.products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price DECIMAL(8,2),
    stock_quantity INTEGER DEFAULT 0
);

-- ดูตารางในแต่ละ Schema
\dt sales.*
\dt hr.*
\dt inventory.*

-- ใส่ข้อมูลทดสอบ
INSERT INTO sales.customers (name, email, phone) VALUES
    ('John Doe', 'john@example.com', '123-456-7890'),
    ('Jane Smith', 'jane@example.com', '098-765-4321'),
    ('Bob Johnson', 'bob@example.com', '555-123-4567');

-- เพิ่มข้อมูลในตาราง orders เพื่อให้สามารถทดสอบ JOIN ได้
INSERT INTO sales.orders (customer_id, order_date, total_amount) VALUES
    (1, '2024-01-15 10:30:00', 1299.98),
    (1, '2024-02-20 14:45:00', 89.97),
    (2, '2024-01-28 09:15:00', 79.99),
    (3, '2024-02-10 16:20:00', 1049.99);

INSERT INTO hr.employees (name, department, salary, hire_date) VALUES
    ('Alice Brown', 'IT', 75000, '2023-01-15'),
    ('Charlie Wilson', 'HR', 65000, '2023-02-01'),
    ('Diana Lee', 'Finance', 80000, '2023-03-10');

INSERT INTO inventory.products (product_name, category, price, stock_quantity) VALUES
    ('Laptop', 'Electronics', 999.99, 50),
    ('Mouse', 'Electronics', 29.99, 200),
    ('Keyboard', 'Electronics', 79.99, 100);

-- สร้างตารางเพิ่มเติมสำหรับทดสอบ JOIN ข้าม Schema
CREATE TABLE hr.employee_orders (
    emp_order_id SERIAL PRIMARY KEY,
    employee_id INTEGER REFERENCES hr.employees(employee_id),
    customer_id INTEGER, -- จะ reference ไปยัง sales.customers
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    commission DECIMAL(8,2)
);

-- ใส่ข้อมูลสำหรับทดสอบ JOIN ข้าม Schema
INSERT INTO hr.employee_orders (employee_id, customer_id, order_date, commission) VALUES
    (1, 1, '2024-01-15 10:30:00', 129.99),
    (1, 2, '2024-01-28 09:15:00', 7.99),
    (2, 3, '2024-02-10 16:20:00', 104.99),
    (3, 1, '2024-02-20 14:45:00', 8.99);
```

**บันทึกผลการทดลอง - Step 9:**
```
ใส่ Screenshot ของ:
1. <img width="417" height="299" alt="image" src="https://github.com/user-attachments/assets/59803b98-9a02-47ef-bdfd-1bbd49465649" />
2. <img width="290" height="137" alt="image" src="https://github.com/user-attachments/assets/1ad539ea-2cd2-4174-8945-d923db2e7339" />
<img width="324" height="138" alt="image" src="https://github.com/user-attachments/assets/f9aa0827-4484-4b3d-b5b6-b38329b3610f" />
<img width="373" height="215" alt="image" src="https://github.com/user-attachments/assets/79bf2564-1044-4643-be9d-c9d040c2254a" />
3. <img width="803" height="261" alt="image" src="https://github.com/user-attachments/assets/d9ebf53a-1b8e-46e8-b55b-e6f4fc042a81" />

4. <img width="714" height="107" alt="image" src="https://github.com/user-attachments/assets/924afcd6-de3a-4ebf-9358-9a976aba32e9" />
```

### Step 10: ทดสอบการเข้าถึง Schema และ Search Path

```sql
-- ตรวจสอบ Search Path ปัจจุบัน
SHOW search_path;

-- เข้าถึงตารางโดยระบุ Schema เต็ม
SELECT * FROM sales.customers;
SELECT * FROM hr.employees;
SELECT * FROM inventory.products;

-- ตั้งค่า Search Path
SET search_path TO sales, hr, public;
SHOW search_path;

-- ทดสอบการเข้าถึงโดยไม่ต้องระบุ Schema
SELECT * FROM customers; -- จะหาจาก sales schema
SELECT * FROM employees; -- จะหาจาก hr schema

-- ทดสอบ JOIN ภายใน Schema เดียวกัน
SELECT 
    c.name as customer_name,
    o.order_date,
    o.total_amount
FROM sales.customers c
JOIN sales.orders o ON c.customer_id = o.customer_id
ORDER BY o.order_date;

-- ทดสอบ JOIN ข้าม Schema (sales และ hr)
SELECT 
    c.name as customer_name,
    e.name as employee_name,
    e.department,
    eo.order_date,
    eo.commission
FROM sales.customers c
JOIN hr.employee_orders eo ON c.customer_id = eo.customer_id
JOIN hr.employees e ON eo.employee_id = e.employee_id
ORDER BY eo.order_date;

-- ทดสอบ Complex JOIN ข้าม 3 Schema
SELECT 
    c.name as customer_name,
    e.name as sales_rep,
    e.department,
    p.product_name,
    p.price,
    eo.commission
FROM sales.customers c
JOIN hr.employee_orders eo ON c.customer_id = eo.customer_id
JOIN hr.employees e ON eo.employee_id = e.employee_id
JOIN inventory.products p ON p.price < (eo.commission * 10) -- สมมติว่า commission เป็น 10% ของ product price
ORDER BY c.name, eo.order_date;

-- Reset Search Path
SET search_path TO public;
```

**บันทึกผลการทดลอง - Step 10:**
```
ใส่ Screenshot ของ:
1. <img width="207" height="97" alt="image" src="https://github.com/user-attachments/assets/753b1082-3018-4bcb-a57f-f003c365eec7" />

2. <img width="501" height="298" alt="image" src="https://github.com/user-attachments/assets/21f40bf1-fd3d-4f30-92da-e0eac4d9d6ab" />
<img width="390" height="95" alt="image" src="https://github.com/user-attachments/assets/2dcee14b-4b0c-472e-b2ee-b02cc8ee41c1" />
3.<img width="844" height="303" alt="image" src="https://github.com/user-attachments/assets/07ed0c91-5000-428f-bf6a-b3ae7d5f171d" />

4. <img width="844" height="303" alt="image" src="https://github.com/user-attachments/assets/145930bd-3741-4066-b9c7-d5487654c225" />

```

### Step 11: ทดสอบการเชื่อมต่อจาก User อื่น

```bash
# เปิด Terminal ใหม่และทดสอบเชื่อมต่อด้วย lab_user
docker exec -it postgres-lab psql -U lab_user -d lab_db
```

```sql
-- ทดสอบสิทธิ์ของ lab_user
\conninfo

-- ทดสอบการเข้าถึงข้อมูล
SELECT * FROM test_permissions; -- ควรทำงานได้
SELECT * FROM sales.customers; -- ไม่มีสิทธิ์

-- ทดสอบการ INSERT
INSERT INTO test_permissions (name) VALUES ('Test by lab_user'); -- ทำไม่ได้


-- ออกจาก psql
\q
```

**บันทึกผลการทดลอง - Step 11:**
```
ใส่ Screenshot ของ:
1.<img width="586" height="68" alt="image" src="https://github.com/user-attachments/assets/a925f804-7287-4250-b5ff-624fe09007bd" />
2. <img width="884" height="55" alt="image" src="https://github.com/user-attachments/assets/b273dcdf-6c94-43e2-8e5d-c2cf318c64ac" />
3. ข้อความ error (ถ้ามี) เมื่อไม่มีสิทธิ์
<img width="584" height="134" alt="image" src="https://github.com/user-attachments/assets/12351ea1-c566-4735-af47-66cd2be5d516" />

```

### Step 12: การจัดการ Volume และ Data Persistence

```bash
# ทดสอบความคงทนของข้อมูล - หยุด Container
docker stop postgres-lab

# ตรวจสอบสถานะ Container
docker ps -a

# เริ่ม Container อีกครั้ง
docker start postgres-lab

# ตรวจสอบว่าข้อมูลยังอยู่
docker exec -it postgres-lab psql -U postgres -d lab_db -c "SELECT COUNT(*) FROM sales.customers;"

# สร้าง Bind Mount สำหรับ backup
mkdir -p ~/postgres-backup

# สร้าง Container ใหม่พร้อม Bind Mount
docker run --name postgres-backup-test \
  -e POSTGRES_PASSWORD=backup123 \
  -v ~/postgres-backup:/backup \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5433:5432 \
  -d postgres
```

**บันทึกผลการทดลอง - Step 12:**
```
ใส่ Screenshot ของ:
1. ผลการหยุดและเริ่ม Container
<img width="292" height="45" alt="image" src="https://github.com/user-attachments/assets/cf7e4809-1327-4c3b-bf77-f24305f97fab" />

2. ยืนยันว่าข้อมูลยังอยู่หลังจาก restart
<img width="763" height="104" alt="image" src="https://github.com/user-attachments/assets/042e9cb7-fcb6-4b4d-8350-f03f1ba62955" />

3. ผลการสร้าง container พร้อม bind mount
<img width="286" height="37" alt="image" src="https://github.com/user-attachments/assets/1a187be2-b144-4dc5-93a7-933ffe09135c" />
<img width="622" height="161" alt="image" src="https://github.com/user-attachments/assets/3699d98d-00be-4657-8370-f3c1249bb0e9" />
```

## การตรวจสอบผลงานและ Performance

### Checkpoint 1: Container และ Resource Management
```bash
# ตรวจสอบสถานะทุก Container
docker ps -a

# ตรวจสอบการใช้ resources
docker stats postgres-lab --no-stream

# ตรวจสอบ Volume usage
docker system df

# ตรวจสอบ Volume details
docker volume inspect postgres-data
```

**บันทึกผล Checkpoint 1:**
```
ใส่ Screenshot ของ resource usage และ volume information ที่นี่
<img width="591" height="46" alt="image" src="https://github.com/user-attachments/assets/d28bfc40-bb0e-468a-9b65-263eaf6e6456" />
<img width="356" height="85" alt="image" src="https://github.com/user-attachments/assets/36869644-d904-4eef-bfda-34d0349aad18" />
<img width="387" height="156" alt="image" src="https://github.com/user-attachments/assets/5859ae5b-5ce7-43a5-ad25-c06bc9423546" />

```

### Checkpoint 2: Database Performance และ Configuration
```sql
-- เชื่อมต่อไปยัง postgres
docker exec -it postgres-lab psql -U postgres

-- ตรวจสอบ Database Statistics
SELECT 
    datname,
    numbackends,
    xact_commit,
    xact_rollback,
    blks_read,
    blks_hit,
    temp_files,
    temp_bytes
FROM pg_stat_database 
WHERE datname IN ('postgres', 'testdb', 'lab_db');

-- ตรวจสอบ Memory Usage
SELECT 
    name,
    setting,
    unit
FROM pg_settings 
WHERE name IN (
    'shared_buffers', 
    'work_mem', 
    'maintenance_work_mem',
    'effective_cache_size',
    'max_connections'
);

-- ตรวจสอบ Active Connections
SELECT 
    datname,
    usename,
    client_addr,
    state,
    query_start
FROM pg_stat_activity 
WHERE state = 'active';
```

**บันทึกผล Checkpoint 2:**
```
ใส่ Screenshot ของ:
1. Database statistics
<img width="939" height="374" alt="image" src="https://github.com/user-attachments/assets/3dc7e7ad-2a67-4bbf-8141-6146b35822aa" />
2. Memory configuration
<img width="369" height="436" alt="image" src="https://github.com/user-attachments/assets/47cd60e3-4d69-49d0-a489-7789005837ae" />
3. Active connections
<img width="677" height="263" alt="image" src="https://github.com/user-attachments/assets/5a415bda-f454-4ec2-bb66-924c0594fa10" />
```

## การแก้ไขปัญหาเบื้องต้น

### ปัญหา Port ซ้ำ
```bash
# ตรวจสอบ Port ที่ใช้งาน
netstat -tulpn | grep 5432
# หรือใน Windows
netstat -an | findstr 5432

# ใช้ Port อื่น
docker run --name postgres-alt -p 5433:5432 -d postgres:15
```

### ปัญหา Memory และ Performance
```bash
# ตรวจสอบการใช้ Memory
docker stats --no-stream

# ปรับแต่ง Memory settings
docker run --name postgres-optimized \
  --memory="2g" \
  --memory-swap="4g" \
  -e POSTGRES_PASSWORD=admin123 \
  -d postgres \
  -c shared_buffers=512MB \
  -c work_mem=32MB \
  -c maintenance_work_mem=256MB
```

### ปัญหา Volume และ Permissions
```bash
# ตรวจสอบ Volume
docker volume inspect postgres-data

# ล้าง Volume (ระวัง: จะลบข้อมูลทั้งหมด)
docker volume rm postgres-data

# สร้าง Volume ใหม่
docker volume create postgres-data
```

## แบบฝึกหัด

### แบบฝึกหัด 1: การสร้าง Multi-Database Environment
**คำสั่ง**: สร้าง PostgreSQL Container ใหม่ที่มี:
- ชื่อ: `multi-postgres`
- รหัสผ่าน: `multipass123`  
- Port: 5434
- Memory limit: 1.5GB
- CPU limit: 1.5 cores
- Volume: `multi-postgres-data`

```bash
# พื้นที่สำหรับคำตอบ - เขียน command ที่ใช้
docker run -d \
  --name multi-postgres \
  -e POSTGRES_PASSWORD=multipass123 \
  -p 5434:5432 \
  --memory="1.5g" \
  --cpus="1.5" \
  -v multi-postgres-data:/var/lib/postgresql/data \
  postgres
```

**ผลการทำแบบฝึกหัด 1:**
```
ใส่ Screenshot ของ:
1. คำสั่งที่ใช้สร้าง container
<img width="627" height="198" alt="image" src="https://github.com/user-attachments/assets/f1460c2d-6c44-497e-b06e-b598a651ea92" />
2. docker ps แสดง container ใหม่
<img width="1112" height="107" alt="image" src="https://github.com/user-attachments/assets/62aa4c2d-43f3-4ba2-8690-d9707219a3a1" />
3. docker stats แสดงการใช้ resources
<img width="1066" height="87" alt="image" src="https://github.com/user-attachments/assets/86acd4bd-69e7-4db0-a75b-506f2dddf3cd" />

```

### แบบฝึกหัด 2: User Management และ Security
**คำสั่ง**: สร้างระบบผู้ใช้ที่สมบูรณ์:

1. สร้าง Role Groups:
   - `app_developers` - สำหรับนักพัฒนา
   - `data_analysts` - สำหรับนักวิเคราะห์ข้อมูล  
   - `db_admins` - สำหรับผู้ดูแลฐานข้อมูล

2. สร้าง Users:
   - `dev_user` (รหัสผ่าน: `dev123`) - เป็นสมาชิกของ app_developers
   - `analyst_user` (รหัสผ่าน: `analyst123`) - เป็นสมาชิกของ data_analysts
   - `admin_user` (รหัสผ่าน: `admin123`) - เป็นสมาชิกของ db_admins

```sql
-- พื้นที่สำหรับคำตอบ - เขียน SQL commands ที่ใช้
docker exec -it multi-postgres psql -U postgres

CREATE ROLE app_developers NOLOGIN;
CREATE ROLE data_analysts NOLOGIN;
CREATE ROLE db_admins NOLOGIN;

CREATE ROLE dev_user LOGIN PASSWORD 'dev123';
GRANT app_developers TO dev_user;

CREATE ROLE analyst_user LOGIN PASSWORD 'analyst123';
GRANT data_analysts TO analyst_user;

CREATE ROLE admin_user LOGIN PASSWORD 'admin123';
GRANT db_admins TO admin_user;

docker exec -e PGPASSWORD=dev123 -it multi-postgres \
  psql -U dev_user -d postgres -c "SELECT current_user, session_user;"

docker exec -e PGPASSWORD=analyst123 -it multi-postgres \
  psql -U analyst_user -d postgres -c "SELECT current_user, session_user;"

docker exec -e PGPASSWORD=admin123 -it multi-postgres \
  psql -U admin_user -d postgres -c "SELECT current_user, session_user;"
```

**ผลการทำแบบฝึกหัด 2:**
```
ใส่ Screenshot ของ:
1. การสร้าง roles และ users
<img width="567" height="387" alt="image" src="https://github.com/user-attachments/assets/297a1150-6e7b-418f-8f5b-b29713fc1ceb" />

2. ผลการรัน \du แสดงผู้ใช้ทั้งหมด
<img width="721" height="242" alt="image" src="https://github.com/user-attachments/assets/ce3b5e8f-4b39-474b-8940-54527824ae08" />

3. ผลการทดสอบเชื่อมต่อด้วย user ต่างๆ
<img width="669" height="138" alt="image" src="https://github.com/user-attachments/assets/a0dc5880-12b7-45ba-a84d-be69664d1fc7" />
<img width="683" height="131" alt="image" src="https://github.com/user-attachments/assets/e8af187b-61a0-45fd-a5a0-60d1c25152d5" />
<img width="703" height="144" alt="image" src="https://github.com/user-attachments/assets/ff7c1ccc-6257-41ab-82d4-71a4df79551d" />
```

### แบบฝึกหัด 3: Schema Design และ Complex Queries
**คำสั่ง**: สร้างระบบฐานข้อมูลร้านค้าออนไลน์:

1. สร้าง Schemas:
   - `ecommerce` - สำหรับข้อมูลหลัก
   - `analytics` - สำหรับข้อมูลการวิเคราะห์
   - `audit` - สำหรับการตรวจสอบ

2. สร้างตารางใน ecommerce schema:
   - `categories` (category_id, name, description)
   - `products` (product_id, name, description, price, category_id, stock)
   - `customers` (customer_id, name, email, phone, address)
   - `orders` (order_id, customer_id, order_date, status, total)
   - `order_items` (order_item_id, order_id, product_id, quantity, price)

3. ใส่ข้อมูลตัวอย่างดังนี้
   ```
   
-- ใส่ข้อมูลใน categories
INSERT INTO ecommerce.categories (name, description) VALUES
    ('Electronics', 'Electronic devices and gadgets'),
    ('Clothing', 'Apparel and fashion items'),
    ('Books', 'Books and educational materials'),
    ('Home & Garden', 'Home improvement and garden supplies'),
    ('Sports', 'Sports equipment and accessories');

-- ใส่ข้อมูลใน products
INSERT INTO ecommerce.products (name, description, price, category_id, stock) VALUES
    ('iPhone 15', 'Latest Apple smartphone', 999.99, 1, 50),
    ('Samsung Galaxy S24', 'Android flagship phone', 899.99, 1, 45),
    ('MacBook Air', 'Apple laptop computer', 1299.99, 1, 30),
    ('Wireless Headphones', 'Bluetooth noise-canceling headphones', 199.99, 1, 100),
    ('Gaming Mouse', 'High-precision gaming mouse', 79.99, 1, 75),
    
    ('T-Shirt', 'Cotton casual t-shirt', 19.99, 2, 200),
    ('Jeans', 'Denim blue jeans', 59.99, 2, 150),
    ('Sneakers', 'Comfortable running sneakers', 129.99, 2, 80),
    ('Jacket', 'Winter waterproof jacket', 89.99, 2, 60),
    ('Hat', 'Baseball cap', 24.99, 2, 120),
    
    ('Programming Book', 'Learn Python programming', 39.99, 3, 40),
    ('Novel', 'Best-selling fiction novel', 14.99, 3, 90),
    ('Textbook', 'University mathematics textbook', 79.99, 3, 25),
    
    ('Garden Tools Set', 'Complete gardening tool kit', 49.99, 4, 35),
    ('Plant Pot', 'Ceramic decorative pot', 15.99, 4, 80),
    
    ('Tennis Racket', 'Professional tennis racket', 149.99, 5, 20),
    ('Football', 'Official size football', 29.99, 5, 55);

-- ใส่ข้อมูลใน customers
INSERT INTO ecommerce.customers (name, email, phone, address) VALUES
    ('John Smith', 'john.smith@email.com', '555-0101', '123 Main St, City A'),
    ('Sarah Johnson', 'sarah.j@email.com', '555-0102', '456 Oak Ave, City B'),
    ('Mike Brown', 'mike.brown@email.com', '555-0103', '789 Pine Rd, City C'),
    ('Emily Davis', 'emily.d@email.com', '555-0104', '321 Elm St, City A'),
    ('David Wilson', 'david.w@email.com', '555-0105', '654 Maple Dr, City B'),
    ('Lisa Anderson', 'lisa.a@email.com', '555-0106', '987 Cedar Ln, City C'),
    ('Tom Miller', 'tom.miller@email.com', '555-0107', '147 Birch St, City A'),
    ('Amy Taylor', 'amy.t@email.com', '555-0108', '258 Ash Ave, City B');

-- ใส่ข้อมูลใน orders
INSERT INTO ecommerce.orders (customer_id, order_date, status, total) VALUES
    (1, '2024-01-15 10:30:00', 'completed', 1199.98),
    (2, '2024-01-16 14:20:00', 'completed', 219.98),
    (3, '2024-01-17 09:15:00', 'completed', 159.97),
    (1, '2024-01-18 11:45:00', 'completed', 79.99),
    (4, '2024-01-19 16:30:00', 'completed', 89.98),
    (5, '2024-01-20 13:25:00', 'completed', 1329.98),
    (2, '2024-01-21 15:10:00', 'completed', 149.99),
    (6, '2024-01-22 12:40:00', 'completed', 294.97),
    (3, '2024-01-23 08:50:00', 'completed', 199.99),
    (7, '2024-01-24 17:20:00', 'completed', 169.98),
    (1, '2024-01-25 10:15:00', 'completed', 39.99),
    (8, '2024-01-26 14:35:00', 'completed', 599.97),
    (4, '2024-01-27 11:20:00', 'processing', 179.98),
    (5, '2024-01-28 09:45:00', 'shipped', 44.98),
    (6, '2024-01-29 16:55:00', 'completed', 129.99);

-- ใส่ข้อมูลใน order_items
INSERT INTO ecommerce.order_items (order_id, product_id, quantity, price) VALUES
    -- Order 1: John Smith
    (1, 1, 1, 999.99),  -- iPhone 15
    (1, 4, 1, 199.99),  -- Wireless Headphones
    
    -- Order 2: Sarah Johnson  
    (2, 4, 1, 199.99),  -- Wireless Headphones
    (2, 6, 1, 19.99),   -- T-Shirt
    
    -- Order 3: Mike Brown
    (3, 7, 1, 59.99),   -- Jeans
    (3, 5, 1, 79.99),   -- Gaming Mouse
    (3, 6, 1, 19.99),   -- T-Shirt
    
    -- Order 4: John Smith
    (4, 5, 1, 79.99),   -- Gaming Mouse
    
    -- Order 5: Emily Davis
    (5, 9, 1, 89.99),   -- Jacket
    
    -- Order 6: David Wilson
    (6, 3, 1, 1299.99), -- MacBook Air
    (6, 12, 2, 14.99),  -- Novel x2
    
    -- Order 7: Sarah Johnson
    (7, 16, 1, 149.99), -- Tennis Racket
    
    -- Order 8: Lisa Anderson
    (8, 8, 2, 129.99),  -- Sneakers x2
    (8, 10, 1, 24.99),  -- Hat
    (8, 11, 1, 39.99),  -- Programming Book
    
    -- Order 9: Mike Brown
    (9, 4, 1, 199.99),  -- Wireless Headphones
    
    -- Order 10: Tom Miller
    (10, 2, 1, 899.99), -- Samsung Galaxy S24
    (10, 6, 3, 19.99),  -- T-Shirt x3
    (10, 14, 1, 49.99), -- Garden Tools Set
    
    -- Order 11: John Smith
    (11, 11, 1, 39.99), -- Programming Book
    
    -- Order 12: Amy Taylor
    (12, 1, 1, 999.99), -- iPhone 15 (ลดราคาเหลือ 599.97)
    
    -- Order 13: Emily Davis (processing)
    (13, 17, 6, 29.99), -- Football x6
    
    -- Order 14: David Wilson (shipped)
    (14, 15, 2, 15.99), -- Plant Pot x2
    (14, 12, 1, 14.99), -- Novel
    
    -- Order 15: Lisa Anderson
    (15, 8, 1, 129.99); -- Sneakers
   ```
   สร้าง queries เพื่อหาคำตอบ:
   - หาสินค้าที่ขายดีที่สุด 5 อันดับ
   - หายอดขายรวมของแต่ละหมวดหมู่
   - หาลูกค้าที่ซื้อสินค้ามากที่สุด

```sql
-- พื้นที่สำหรับคำตอบ - เขียน SQL commands ทั้งหมด
docker exec -it multi-postgres psql -U postgres

CREATE SCHEMA ecommerce;
CREATE SCHEMA analytics;
CREATE SCHEMA audit;

CREATE TABLE ecommerce.categories (
  category_id   BIGSERIAL PRIMARY KEY,
  name          TEXT NOT NULL,
  description   TEXT
);

CREATE TABLE ecommerce.products (
  product_id    BIGSERIAL PRIMARY KEY,
  name          TEXT NOT NULL,
  description   TEXT,
  price         NUMERIC(10,2) NOT NULL,
  category_id   BIGINT REFERENCES ecommerce.categories(category_id),
  stock         INT NOT NULL
);

CREATE TABLE ecommerce.customers (
  customer_id   BIGSERIAL PRIMARY KEY,
  name          TEXT NOT NULL,
  email         TEXT NOT NULL,
  phone         TEXT,
  address       TEXT
);

CREATE TABLE ecommerce.orders (
  order_id      BIGSERIAL PRIMARY KEY,
  customer_id   BIGINT REFERENCES ecommerce.customers(customer_id),
  order_date    TIMESTAMP NOT NULL,
  status        TEXT NOT NULL,
  total         NUMERIC(12,2) NOT NULL
);

CREATE TABLE ecommerce.order_items (
  order_item_id BIGSERIAL PRIMARY KEY,
  order_id      BIGINT REFERENCES ecommerce.orders(order_id),
  product_id    BIGINT REFERENCES ecommerce.products(product_id),
  quantity      INT NOT NULL,
  price         NUMERIC(10,2) NOT NULL
);
```

**ผลการทำแบบฝึกหัด 3:**
```
ใส่ Screenshot ของ:
1. โครงสร้าง schemas และ tables (\dn+, \dt ecommerce.*)
<img width="785" height="416" alt="image" src="https://github.com/user-attachments/assets/2fd21cee-bd44-411f-a28f-c0e7f3599352" />
2. ข้อมูลตัวอย่างในตารางต่างๆ
<img width="579" height="515" alt="image" src="https://github.com/user-attachments/assets/4f264b53-7f2a-442d-b2de-4d691d958a1c" />
<img width="405" height="358" alt="image" src="https://github.com/user-attachments/assets/51c75cc8-615e-418f-81c2-b1057fdb1ebe" />
3. ผลการรัน queries ที่สร้าง
<img width="294" height="317" alt="image" src="https://github.com/user-attachments/assets/e6086aa5-6df8-4471-82ff-ecea4dd58a93" />
4. การวิเคราะห์ข้อมูลที่ได้
1. สินค้าขายดีที่สุด
Football และ T-Shirt ถูกซื้อมากที่สุด → แสดงให้เห็นว่าได้รับความนิยมของสินค้าราคาต่ำที่มีการซื้อซ้ำบ่อย
กลุ่มสินค้าประเภท Electronics เช่น iPhone 15 และ Wireless Headphones มียอดขายสูงถึงเเม้มีปริมาณจำนวนชิ้นไม่มาก เพราะราคาต่อชิ้นค่อนข้างสูง

2. ยอดขายตามหมวดหมู่
หมวด Electronics สร้างรายได้สูงสุด 75% ของรายได้ทั้งหมดแสดงว่าเป็นหมวดที่ต้องให้ความสำคัญ เช่น ทำโปรโมชั่นเฉพาะกลุ่ม หรือเพิ่ม stock
หมวด Home & Garden มีรายได้น้อยที่สุด ต้องทำการตลาดเพิ่มเติมหรือเพิ่มสินค้าใหม่ให้มีความน่าสนใจมากขึ้น

3. ลูกค้าที่ซื้อสินค้ามากที่สุด
David Wilson มีการใช้จ่ายสูงสุดสามารถให้สิทธิพิเศษเพื่อรักษาลูกค้ากลุ่มนี้
สามารถใช้ข้อมูลนี้ทำ Customer Segmentation ต่อได้
```


## การทดสอบความเข้าใจ

### Quiz 1: Conceptual Questions
ตอบคำถามต่อไปนี้:

1. อธิบายความแตกต่างระหว่าง Named Volume และ Bind Mount ในบริบทของ PostgreSQL
2. เหตุใด shared_buffers จึงควรตั้งเป็น 25% ของ RAM?
3. การใช้ Schema ช่วยในการจัดการฐานข้อมูลขนาดใหญ่อย่างไร?
4. อธิบายประโยชน์ของการใช้ Docker สำหรับ Database Development

**คำตอบ Quiz 1:**
```
เขียนคำตอบที่นี่
1.Named Volume: พื้นที่เก็บข้อมูลที่ Docker สร้างและจัดการให้ เหมาะสำหรับการใช้งานจริง
  Bind Mount: การเชื่อมโยงโฟลเดอร์ในเครื่องเรากับคอนเทนเนอร์โดยตรง เหมาะสำหรับการพัฒนาและแก้ไขไฟล์
2.ตั้ง shared_buffers ที่ 25% ของ RAM เพื่อให้ PostgreSQL มีพื้นที่เก็บข้อมูลที่ใช้บ่อยได้อย่างมีประสิทธิภาพ โดยไม่แย่งหน่วยความจำจากระบบปฏิบัติการ (OS) ซึ่งจะช่วยให้ระบบโดยรวมทำงานได้ดีที่สุด
3.การใช้ Schema ช่วยให้การจัดการฐานข้อมูลขนาดใหญ่เป็นระเบียบและง่ายขึ้น โดยทำหน้าที่คล้ายกับการสร้าง โฟลเดอร์ย่อย หรือ เนมสเปซ (namespace) ในฐานข้อมูลเดียว
4.Docker ช่วยให้นักพัฒนาสร้างและจัดการ สภาพแวดล้อมฐานข้อมูลที่สอดคล้องกัน ได้อย่างรวดเร็วและง่ายดาย ทำให้ทีมทำงานร่วมกันได้ดีขึ้นและลดปัญหาความไม่เข้ากันของเวอร์ชันซอฟต์แวร์.

```


## สรุปและการประเมินผล
## สิ่งที่ได้เรียนรู้: Docker และ PostgreSQL
## การใช้ Docker เพื่อจัดการฐานข้อมูล:
  ## Docker Volume: ใช้เพื่อเก็บข้อมูลอย่างถาวร ทำให้ข้อมูลในฐานข้อมูลไม่หายไปเมื่อ Container ถูกลบ
  -การจัดการทรัพยากร: สามารถจำกัด Memory และ CPU ของ Container ได้ ช่วยให้ควบคุมการใช้ทรัพยากรของเซิร์ฟเวอร์ได้อย่างมีประสิทธิภาพ
  -Container Lifecycle: เข้าใจขั้นตอนการสร้าง, รัน, หยุด, และลบ Container
  ## โครงสร้างสถาปัตยกรรมของ PostgreSQL:
  -Memory Management: การจัดการหน่วยความจำ เช่น Shared Buffers ที่ใช้เก็บข้อมูลที่เข้าถึงบ่อย และ Work Memory ที่ใช้ในการประมวลผลคำสั่ง
  -การจัดระเบียบ: ใช้ Schema เพื่อจัดกลุ่มตารางและวัตถุต่าง ๆ ทำให้ฐานข้อมูลเป็นระเบียบและจัดการได้ง่ายขึ้น
  -การจัดการผู้ใช้และสิทธิ์: User และ Role ช่วยในการกำหนดสิทธิ์การเข้าถึงข้อมูลที่แตกต่างกันไปตามหน้าที่
  ## ความปลอดภัยของฐานข้อมูล:
  -การจัดการสิทธิ์: การสร้าง Users/Roles ที่เหมาะสมและกำหนดสิทธิ์ในระดับต่าง ๆ เช่น การอนุญาตให้ผู้ใช้บางคนเข้าถึงได้เฉพาะบางตารางเท่านั้น
  ## Best Practices: ปฏิบัติตามหลักการความปลอดภัย เช่น การใช้รหัสผ่านที่รัดกุมและกำหนดสิทธิ์ขั้นต่ำที่จำเป็น (principle of least privilege) เพื่อลดความเสี่ยงจากการเข้าถึงข้อมูลโดยไม่ได้รับอนุญาต

### สิ่งที่ได้เรียนรู้

1. **Docker Fundamentals**:
   - การใช้ Docker Volume เพื่อความคงทนของข้อมูล
   - การจำกัดทรัพยากร (Memory, CPU)
   - การจัดการ Container lifecycle

2. **PostgreSQL Architecture**:
   - โครงสร้าง Database Cluster
   - Memory Management (Shared Buffers, Work Memory)
   - User และ Role Management
   - Schema และ Namespace Organization

4. **Security และ Privileges**:
   - การสร้างและจัดการ Users/Roles
   - การกำหนดสิทธิ์ในระดับต่างๆ
   - Best Practices สำหรับ Database Security


### Best Practices ที่ควรจำ

1. **Docker Management**:
   - ใช้ Named Volume สำหรับข้อมูลสำคัญ
   - จำกัดทรัพยากรเพื่อป้องกันระบบล่ม
   - ตั้งชื่อ Container และ Volume ให้มีความหมาย

2. **PostgreSQL Configuration**:
   - ปรับแต่ง Memory settings ตามขนาด RAM
   - ใช้ Schema เพื่อจัดระเบียบฐานข้อมูล
   - สร้าง User ตามหลักการ Least Privilege

3. **Security**:
   - ใช้รหัสผ่านที่แข็งแรง
   - จำกัดสิทธิ์ User ตามความจำเป็น
   - ติดตามการเข้าถึงผ่าน pg_stat_activity

4. **Monitoring**:
   - ตรวจสอบ Resource usage เป็นประจำ
   - ใช้ pg_stat_* views เพื่อติดตามประสิทธิภาพ
   - สำรองข้อมูลเป็นประจำ


---
**หมายเหตุสำคัญ**: 
- อ่านคำแนะนำทุกข้อก่อนเริ่มทำการทดลอง
- Screenshot ควรมีความชัดเจนและแสดงผลลัพธ์ที่เกี่ยวข้อง  
- หากพบปัญหาให้บันทึกและอธิบายวิธีแก้ไขในรายงาน
- การทดลองนี้เป็นพื้นฐานสำคัญสำหรับการเรียนรู้ในบทเรียนถัดไป

### ทรัพยากรเพิ่มเติม
- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/)
- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
