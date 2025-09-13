# Lab 01: PostgreSQL Docker Setup and Basic Operations

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

ใส่ Screenshot ของผลการรัน docker --version และ docker run hello-world ที่นี่
<img width="547" height="75" alt="image" src="https://github.com/user-attachments/assets/24a7abe4-7018-42da-b128-296b2a057e3e" />
<img width="1199" height="638" alt="image" src="https://github.com/user-attachments/assets/95d4fb16-1275-4c10-b751-198f503063b6" />


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

ใส่ Screenshot ของผลการรัน docker images ที่นี่
<img width="961" height="138" alt="image" src="https://github.com/user-attachments/assets/ff1da321-ff38-4254-8157-e32abe52cff7" />


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

ใส่ Screenshot ของผลการรัน docker volume ls และ docker volume inspect postgres-data ที่นี่
<img width="1176" height="386" alt="image" src="https://github.com/user-attachments/assets/6e432ba1-cffc-4980-abdf-e00abf842ed6" />



### Step 3: Create PostgreSQL Container with Volume


# สร้างและรัน PostgreSQL Container พร้อม Volume
```bash
  docker run --name postgres-lab -e POSTGRES_PASSWORD=admin123 -e POSTGRES_DB=testdb -e POSTGRES_USER=postgres -v postgres-data:/var/lib/postgresql/data -v postgres-config:/etc/postgresql -p 5432:5432 --memory="1g" --cpus="1.0" -d postgres -c shared_buffers=256MB -c work_mem=16MB -c maintenance_work_mem=128MB
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

ใส่ Screenshot ของผลการรัน docker run ที่นี่
<img width="1486" height="136" alt="image" src="https://github.com/user-attachments/assets/7738fbeb-ecaa-4682-a73a-1cb58536e0b7" />


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

ใส่ Screenshot ของ:
1. ผลการรัน docker ps
<img width="1460" height="127" alt="image" src="https://github.com/user-attachments/assets/bbb091e2-fc75-4796-9d6d-91960e5fc224" />

2. ส่วนหนึ่งของ docker logs postgres-lab
<img width="1483" height="374" alt="image" src="https://github.com/user-attachments/assets/7bf0008e-50bf-4c48-a7e5-b9b666fe627c" />

3. ผลการรัน docker stats
<img width="1481" height="509" alt="image" src="https://github.com/user-attachments/assets/65fc3d95-a87a-4290-b9af-1f8c3d02477b" />


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

ใส่ Screenshot ของ:
1. ผลการรัน SELECT version();
<img width="1478" height="180" alt="image" src="https://github.com/user-attachments/assets/e1591fd0-2418-46ae-9882-db383ee310c1" />

2. ผลการรัน SHOW shared_buffers; SHOW work_mem; SHOW maintenance_work_mem;SHOW effective_cache_size;
<img width="475" height="582" alt="image" src="https://github.com/user-attachments/assets/96618c77-8ec0-4b2b-b221-0b1c8304b229" />

3. ผลการรัน \l และ \du
<img width="1468" height="689" alt="image" src="https://github.com/user-attachments/assets/ffdc7127-347f-4ca0-be7f-6d9f4bac38a6" />
<img width="1480" height="179" alt="image" src="https://github.com/user-attachments/assets/a95d3212-4ce6-4c72-a629-dad88f19f422" />


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

ใส่ Screenshot ของ:
1. ผลการสร้าง lab_db
   <img width="1035" height="476" alt="image" src="https://github.com/user-attachments/assets/fed74449-5783-4e97-bac1-f818f5989118" />

2. ผลการรัน \l+ แสดงฐานข้อมูลทั้งหมด
  <img width="1474" height="558" alt="image" src="https://github.com/user-attachments/assets/bfb824d3-5c81-40b8-ab93-a593ca46dd23" />

3. ผลการ query ข้อมูลฐานข้อมูล
<img width="1156" height="480" alt="image" src="https://github.com/user-attachments/assets/f3bc726c-2157-47c6-b244-2223eb7b344d" />


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

ใส่ Screenshot ของ:
1. ผลการสร้าง users ทั้งหมด
  <img width="594" height="542" alt="image" src="https://github.com/user-attachments/assets/2ef281c1-1675-4bc1-9968-c8710e92dbda" />

2. ผลการรัน \du+
   <img width="1142" height="336" alt="image" src="https://github.com/user-attachments/assets/21383bf7-f65b-4b20-bd7b-dabca7095a66" />

3. ผลการ query pg_roles
<img width="1355" height="468" alt="image" src="https://github.com/user-attachments/assets/1f571951-44c1-4363-9073-7623f2c02aa2" />


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

ใส่ Screenshot ของ:
1. ผลการ ALTER USER commands
  <img width="898" height="135" alt="image" src="https://github.com/user-attachments/assets/2f1e4fe2-7feb-43ab-8a54-c35ae7bb119d" />

2. ผลการรัน \dp test_permissions
  <img width="1170" height="244" alt="image" src="https://github.com/user-attachments/assets/d8909064-58ec-4238-9456-d5601e64fc2d" />

3. ผลการ GRANT commands
<img width="737" height="447" alt="image" src="https://github.com/user-attachments/assets/e800398d-c7b6-4755-b69e-de7c4b882be1" />
<img width="758" height="98" alt="image" src="https://github.com/user-attachments/assets/377e0edb-ea80-4cfe-8325-3f2cbcc0c59b" />
<img width="766" height="78" alt="image" src="https://github.com/user-attachments/assets/aaff82b3-d9cc-464d-b5f0-73cf3d844348" />

**คำถาม
 
Access Privileges   postgres=arwdDxtm/postgres มีความหมายอย่างไร
postgres=arwdDxtm/postgres มีหมายความว่า role postgres เป็นเจ้าของตารางนี้ และมีสิทธิ์ครบถ้วน (INSERT, SELECT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER, MERGE) กับตารางนี้

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

ใส่ Screenshot ของ:
1. ผลการสร้าง schemas (\dn+)
   <img width="1232" height="337" alt="image" src="https://github.com/user-attachments/assets/11afed83-4bb7-41d7-9907-badb1189326f" />

2. ผลการสร้างตารางในแต่ละ schema
<img width="697" height="486" alt="image" src="https://github.com/user-attachments/assets/ab13384c-a227-486e-8849-d7cb9bf3827f" />

3. ผลการใส่ข้อมูลและ query ข้อมูล
   <img width="1184" height="643" alt="image" src="https://github.com/user-attachments/assets/45806cba-4441-48a5-943f-e14d0444264b" />

4. ข้อมูลในตาราง employee_orders ที่จะใช้สำหรับ JOIN ข้าม schema
<img width="1000" height="217" alt="image" src="https://github.com/user-attachments/assets/673af721-4d29-4a69-8a74-c27913e816b7" />


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

ใส่ Screenshot ของ:
1. ผลการแสดง search_path
<img width="573" height="145" alt="image" src="https://github.com/user-attachments/assets/a8468859-859a-43d6-91f7-d29704ceafd2" />
<img width="930" height="169" alt="image" src="https://github.com/user-attachments/assets/0c45b7da-c105-4330-8562-bcb36d62f982" />

2. ผลการ query ภายใน schema เดียวกัน (sales.customers + sales.orders)
  <img width="885" height="365" alt="image" src="https://github.com/user-attachments/assets/9c8cdf75-6461-4814-8730-0f59568fa3da" />

3. ผลการ JOIN ข้าม schemas (sales + hr + inventory)
   <img width="1056" height="409" alt="image" src="https://github.com/user-attachments/assets/b5bb2be0-4750-4198-9857-2f9669393641" />

4. ข้อมูลที่แสดงจาก complex join ข้าม 3 schemas
<img width="1116" height="549" alt="image" src="https://github.com/user-attachments/assets/bec1759a-4565-45a7-b976-6ba8accb7cec" />


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

ใส่ Screenshot ของ:
1. ผลการเชื่อมต่อด้วย lab_user
   <img width="1255" height="126" alt="image" src="https://github.com/user-attachments/assets/9490d92b-8011-4ce3-85f4-d51220e31622" />

2. ผลการทดสอบสิทธิ์ต่างๆ
   <img width="1239" height="278" alt="image" src="https://github.com/user-attachments/assets/16d513ea-73c2-4f77-9296-822885b99998" />

3. ข้อความ error (ถ้ามี) เมื่อไม่มีสิทธิ์
<img width="512" height="53" alt="image" src="https://github.com/user-attachments/assets/28c16014-8321-433c-857a-9e390f9669ef" />


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

ใส่ Screenshot ของ:
1. ผลการหยุดและเริ่ม Container
<img width="575" height="77" alt="image" src="https://github.com/user-attachments/assets/8c588c7e-1484-4823-9f96-5a50f91b1bc4" />
<img width="1480" height="298" alt="image" src="https://github.com/user-attachments/assets/fb5ae9ac-6658-4a70-ad1a-8e28a258889d" />

2. ยืนยันว่าข้อมูลยังอยู่หลังจาก restart
   <img width="1382" height="169" alt="image" src="https://github.com/user-attachments/assets/6d44688f-30d0-4610-aa25-120b4c57d322" />

4. ผลการสร้าง container พร้อม bind mount
<img width="1477" height="88" alt="image" src="https://github.com/user-attachments/assets/d9282288-2783-49b2-8698-f6854f9c3be9" />
<img width="1469" height="192" alt="image" src="https://github.com/user-attachments/assets/a2dcd93e-72cb-4e35-966c-51e322e892c2" />


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

ใส่ Screenshot ของ resource usage และ volume information ที่นี่
<img width="1462" height="116" alt="image" src="https://github.com/user-attachments/assets/df84b842-0078-4942-8559-082083cfcc10" />
<img width="1476" height="532" alt="image" src="https://github.com/user-attachments/assets/e2b975f9-2fb3-4497-95e1-e60bc7cb7280" />


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

ใส่ Screenshot ของ:
1. Database statistics
   <img width="1307" height="405" alt="image" src="https://github.com/user-attachments/assets/f4bbc203-3dc2-47ce-a956-3c5093774d39" />

2. Memory configuration
   <img width="801" height="435" alt="image" src="https://github.com/user-attachments/assets/aa0f6c71-43cb-4c9f-a514-9a48dcf54b6e" />

3. Active connections
<img width="958" height="268" alt="image" src="https://github.com/user-attachments/assets/20cbc735-e4a6-4983-a8a1-a113e72683ae" />


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
docker run --name multi-postgres ^
 -e POSTGRES_PASSWORD=multipass123 ^
 -v multi-postgres-data:/var/lib/postgresql/data ^
 -p 5434:5432 ^
 --memory="1.5g" ^
 --cpus="1.5" ^
 -d postgres

```

**ผลการทำแบบฝึกหัด 1:**

ใส่ Screenshot ของ:
1. คำสั่งที่ใช้สร้าง container
<img width="1523" height="104" alt="image" src="https://github.com/user-attachments/assets/c6b5899c-dfcf-42a4-8f73-8139219cfe32" />

2. docker ps แสดง container ใหม่
<img width="1465" height="232" alt="image" src="https://github.com/user-attachments/assets/cc5519a6-8956-4a0e-b4f3-bd839e4e7253" />

3. docker stats แสดงการใช้ resources
<img width="1471" height="270" alt="image" src="https://github.com/user-attachments/assets/455211af-0ab4-4d11-8b32-050310650462" />



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
-- สร้าง Role Groups
CREATE ROLE app_developers NOLOGIN;
CREATE ROLE data_analysts NOLOGIN;
CREATE ROLE db_admins NOLOGIN;

-- สร้าง Users พร้อมรหัสผ่าน และกำหนดให้เป็นสมาชิก Role Groups
CREATE USER dev_user WITH PASSWORD 'dev123' LOGIN;
GRANT app_developers TO dev_user;

CREATE USER analyst_user WITH PASSWORD 'analyst123' LOGIN;
GRANT data_analysts TO analyst_user;

CREATE USER admin_user WITH PASSWORD 'admin123' LOGIN;
GRANT db_admins TO admin_user;

-- ตรวจสอบรายชื่อผู้ใช้และกลุ่ม
\du

```

**ผลการทำแบบฝึกหัด 2:**

ใส่ Screenshot ของ:
1. การสร้าง roles และ users
<img width="581" height="168" alt="image" src="https://github.com/user-attachments/assets/aa19b639-64f3-430d-b46c-130049e6530d" />
<img width="831" height="372" alt="image" src="https://github.com/user-attachments/assets/be854018-9069-439b-993e-fee4100bafa3" />

2. ผลการรัน \du แสดงผู้ใช้ทั้งหมด
<img width="1144" height="321" alt="image" src="https://github.com/user-attachments/assets/8e59378e-ae67-455f-afba-2e0eed2bd709" />

3. ผลการทดสอบเชื่อมต่อด้วย user ต่างๆ
<img width="942" height="119" alt="image" src="https://github.com/user-attachments/assets/f10d98d0-787c-49ab-a57c-20b8e5e65b5f" />



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

```sql
   
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
```
   สร้าง queries เพื่อหาคำตอบ:
   - หาสินค้าที่ขายดีที่สุด 5 อันดับ
   - หายอดขายรวมของแต่ละหมวดหมู่
   - หาลูกค้าที่ซื้อสินค้ามากที่สุด
```
```sql
  -- พื้นที่สำหรับคำตอบ - เขียน SQL commands ทั้งหมด

```

**ผลการทำแบบฝึกหัด 3:**

ใส่ Screenshot ของ:
1. โครงสร้าง schemas และ tables (\dn+, \dt ecommerce.*)
<img width="1220" height="243" alt="image" src="https://github.com/user-attachments/assets/05b8007d-1b22-4b44-a35d-64a257e196a4" />
<img width="765" height="272" alt="image" src="https://github.com/user-attachments/assets/9d09b4d6-2226-4cb4-b983-5ca90bcf5523" />

2. ข้อมูลตัวอย่างในตารางต่างๆ
<img width="1374" height="465" alt="image" src="https://github.com/user-attachments/assets/e6a96873-7c1a-4a31-ae67-5c8b598a27f1" />
<img width="1300" height="442" alt="image" src="https://github.com/user-attachments/assets/a488ce4e-8b63-4fbf-a194-14376d344a17" />
<img width="957" height="274" alt="image" src="https://github.com/user-attachments/assets/4ab3be1f-e4e2-4227-9b72-ea0aba008228" />

3. ผลการรัน queries ที่สร้าง
<img width="841" height="429" alt="image" src="https://github.com/user-attachments/assets/de20ff1a-c25c-4adf-a661-ac4c419c53d5" />
<img width="1141" height="433" alt="image" src="https://github.com/user-attachments/assets/a48277ce-b4a9-496b-bd0f-9e53c8e09f2f" />
<img width="1127" height="365" alt="image" src="https://github.com/user-attachments/assets/24549c05-594c-46e5-bba0-544ec9c9f999" />

4. การวิเคราะห์ข้อมูลที่ได้
สินค้าขายดี 5 อันดับแสดงว่ามีความต้องการสูง และควรจัดสต็อกเพิ่ม
ลูกค้าที่ซื้อสินค้ามากที่สุด อาจเป็นลูกค้าที่ร้านควรทำโปรโมชั่นหรือดูแลเป็นพิเศษ



## การทดสอบความเข้าใจ

### Quiz 1: Conceptual Questions
ตอบคำถามต่อไปนี้:

1. อธิบายความแตกต่างระหว่าง Named Volume และ Bind Mount ในบริบทของ PostgreSQL
2. เหตุใด shared_buffers จึงควรตั้งเป็น 25% ของ RAM?
3. การใช้ Schema ช่วยในการจัดการฐานข้อมูลขนาดใหญ่อย่างไร?
4. อธิบายประโยชน์ของการใช้ Docker สำหรับ Database Development

**คำตอบ Quiz 1:**

1.Named Volume คือพื้นที่เก็บข้อมูลที่ Docker จัดการให้เอง อยู่ในที่ที่ Docker กำหนดและแยกออกจากไฟล์ระบบของเรา ช่วยให้การเก็บข้อมูลปลอดภัยและง่ายต่อการย้ายหรือแบ็คอัพ
Bind Mount คือการเอาโฟลเดอร์หรือไฟล์จริงๆ บนเครื่องเรา (host) มาเชื่อมกับ Container ทำให้เราสามารถแก้ไขไฟล์ในเครื่องเราและเห็นผลใน Container ได้ทันที เหมาะกับการพัฒนาและดีบัก แต่ถ้าใช้ผิดวิธีก็อาจเสี่ยงเรื่องสิทธิ์การเข้าถึงไฟล์ได้
2.shared_buffers คือพื้นที่หน่วยความจำที่ PostgreSQL ใช้เก็บข้อมูลที่อ่านจากดิสก์เอาไว้ชั่วคราว เพื่อให้การอ่านข้อมูลเร็วขึ้นถ้าตั้งไว้เยอะเกินก็อาจทำให้ระบบปฏิบัติการเหลือ RAM น้อยเกินไป จนทำงานช้าลง ถ้าน้อยเกินไปก็เสียประสิทธิภาพของฐานข้อมูล
3.Schema เปรียบเหมือนโฟลเดอร์แยกกลุ่มข้อมูลในฐานข้อมูลเดียวกันช่วยจัดระเบียบข้อมูลให้ง่ายต่อการจัดการ เช่น แยกข้อมูลของแต่ละโมดูลหรือทีมออกจากกัน แล้วยังช่วยป้องกันชื่อซ้ำ และกำหนดสิทธิ์เข้าถึงข้อมูลได้ละเอียดมากขึ้น ทำให้ฐานข้อมูลใหญ่ๆ ดูแลได้ง่ายขึ้น
4.Docker ทำให้เราสามารถสร้างฐานข้อมูลขึ้นมาใช้งานได้เร็วมาก โดยไม่ต้องติดตั้งซับซ้อน มีความยืดหยุ่น สามารถสร้างหลายๆ รุ่นหรือหลายๆ ระบบพร้อมกันโดยไม่ชนกัน ช่วยให้ทีมพัฒนาทำงานเหมือนกัน เพราะทุกคนใช้ Container ตัวเดียวกัน
นอกจากนี้ยังง่ายต่อการทดลอง และย้อนกลับไปใช้สถานะเก่าๆ ได้สบายๆ



## สรุปและการประเมินผล

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
