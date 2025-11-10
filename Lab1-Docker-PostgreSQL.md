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
```
ใส่ Screenshot ของผลการรัน docker --version และ docker run hello-world ที่นี่
```
<img width="1114" height="483" alt="image" src="https://github.com/user-attachments/assets/1aed6c89-d1e7-4377-b0ae-2bb8c7cf5aa3" />


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
ใส่ Screenshot ของผลการรัน docker images ที่นี่
```
<img width="1302" height="467" alt="image" src="https://github.com/user-attachments/assets/5e542976-5de6-4524-9daf-c4dfe5fa183f" />

```
[
    {
        "Id": "sha256:7be29db83c6f7084804058fe18952f19424fdab3131caaaf2e90dd21076e0341",
        "RepoTags": [
            "postgres:latest"
        ],
        "RepoDigests": [
            "postgres@sha256:7be29db83c6f7084804058fe18952f19424fdab3131caaaf2e90dd21076e0341"
        ],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2025-09-08T20:04:25Z",
        "DockerVersion": "",
        "Author": "",
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 161233640,
        "GraphDriver": {
            "Data": null,
            "Name": "overlayfs"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:daf557c4f08e0cb77b2af2a08d8bca4f333cf408d88191911715e6f393eca215",
                "sha256:dc138e461a05a4e4fa3cfd5a2d71ede572302939fed3b655dcbb25bf1fcf60f7",
                "sha256:7c4190151c1b5635a67495d2e32da60261d0725200ff00967fc8028d6cbc575a",
                "sha256:c88342a46ab489f98c59be325bc55b8e85633ef209e35efa6d545fdf86646a61",
                "sha256:7a21e896d4ffbc0e2a51ba5bb1675c35ef8558cdf0965a6c8c2a144a51fc4e5e",
                "sha256:fc11892dea433f101badeb9c56e27ade72886f6d563e12e5aa87a9f6a92f667e",
                "sha256:c464b36f8368c20e816aac91e3ac6d9ffffe64db27a6e0e44591e50a9f338b4d",
                "sha256:9a9c5a2faf5583db9b2011059308c3380406327b7c61fce0ab39b17ebb08213c",
                "sha256:4e385a15ff02d7a235266904b333bcd8b9861788770446eaf79d2f45595b8547",
                "sha256:54066a1a5524dbe203f86ab9f84dc805e37048039bcc08bbe20329520c9d098e",
                "sha256:318f76d63bfb29412fa017c61aea4f5398ae250e4907d893bbccb74177d798b5",
                "sha256:b86740af0101b83ff7234d9b63efe9286d70c30baba74ab3f36352b5cc32ab0a",
                "sha256:d1b7fe873873803d7870cfea1e0a6897c8cfad11b1de4cbd25fdf0d11b5fc481",
                "sha256:3f010b4c9898cc755a97a042765530497eb72ec002d51c1af9fbb65005ad3151"
            ]
        },
        "Metadata": {
            "LastTagTime": "2025-09-09T03:07:39.715405115Z"
        },
        "Descriptor": {
            "mediaType": "application/vnd.oci.image.index.v1+json",
            "digest": "sha256:7be29db83c6f7084804058fe18952f19424fdab3131caaaf2e90dd21076e0341",
            "size": 10229
        },
        "Config": {
            "Cmd": [
                "postgres"
            ],
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/postgresql/17/bin",
                "GOSU_VERSION=1.18",
                "LANG=en_US.utf8",
                "PG_MAJOR=17",
                "PG_VERSION=17.6-1.pgdg13+1",
                "PGDATA=/var/lib/postgresql/data"
            ],
            "ExposedPorts": {
                "5432/tcp": {}
            },
            "Labels": null,
            "OnBuild": null,
            "StopSignal": "SIGINT",
            "User": "",
            "Volumes": {
                "/var/lib/postgresql/data": {}
            },
            "WorkingDir": ""
        }
    }
]
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
ใส่ Screenshot ของผลการรัน docker volume ls และ docker volume inspect postgres-data ที่นี่
```
<img width="753" height="440" alt="image" src="https://github.com/user-attachments/assets/f041cbb0-9cfe-463c-a3bf-50e12608ad3e" />


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
```
ใส่ Screenshot ของผลการรัน docker run ที่นี่
```
<img width="891" height="192" alt="image" src="https://github.com/user-attachments/assets/32b08608-26b9-4749-9a18-e862a4fc7970" />


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
1. ผลการรัน docker ps
2. ส่วนหนึ่งของ docker logs postgres-lab
3. ผลการรัน docker stats
```
<img width="1311" height="121" alt="image" src="https://github.com/user-attachments/assets/5ada448d-24a4-4fa5-92df-991394976c20" />

<img width="1303" height="383" alt="image" src="https://github.com/user-attachments/assets/c6546ffd-1e41-4b34-bdba-bdd3c6ae0544" />

<img width="1039" height="75" alt="image" src="https://github.com/user-attachments/assets/cb4e1c81-2090-42cd-9a99-7227b64026d0" />



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
1. ผลการรัน SELECT version();
2. ผลการรัน SHOW shared_buffers; SHOW work_mem; SHOW maintenance_work_mem;SHOW effective_cache_size;
3. ผลการรัน \l และ \du
```
<img width="1165" height="496" alt="image" src="https://github.com/user-attachments/assets/6d0b5b2d-8cd6-4394-8346-80ebf5880603" />

<img width="1262" height="143" alt="image" src="https://github.com/user-attachments/assets/cdddc3c9-2171-43b8-ac70-5722c7bcc044" />

<img width="1354" height="136" alt="image" src="https://github.com/user-attachments/assets/43c3bda2-1f2c-4c49-9c25-d2f9440a2e53" />

<img width="1314" height="140" alt="image" src="https://github.com/user-attachments/assets/576df873-80db-41f8-a8ab-bb798ac001c0" />

<img width="1292" height="126" alt="image" src="https://github.com/user-attachments/assets/8cca42e7-0e53-4cdd-a806-538b6a188caa" />

<img width="1146" height="493" alt="image" src="https://github.com/user-attachments/assets/85bde639-6839-4848-8723-8f97d363373e" />

<img width="1333" height="135" alt="image" src="https://github.com/user-attachments/assets/c0d56fc8-6405-42e0-b5dd-4387b10d8208" />



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
1. ผลการสร้าง lab_db
2. ผลการรัน \l+ แสดงฐานข้อมูลทั้งหมด
3. ผลการ query ข้อมูลฐานข้อมูล
```

<img width="823" height="202" alt="image" src="https://github.com/user-attachments/assets/3c36e8d8-9523-43cd-85a1-021f0a8a9525" />

<img width="1355" height="345" alt="image" src="https://github.com/user-attachments/assets/7697936d-46f4-4be7-ab42-a835aab369ca" />

<img width="1302" height="489" alt="image" src="https://github.com/user-attachments/assets/754beeec-8f11-4f7f-a25b-0bb46b21738a" />


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
1. ผลการสร้าง users ทั้งหมด
2. ผลการรัน \du+
3. ผลการ query pg_roles
```
```
 ผลการสร้าง users ทั้งหมด
postgres=# CREATE USER lab_user WITH 
    PASSWORD 'user123'
    LOGIN
    NOSUPERUSER
    NOCREATEDB
    NOCREATEROLE
    NOINHERIT
    NOREPLICATION
    CONNECTION LIMIT 10;
CREATE ROLE
postgres=# CREATE USER admin_user WITH 
    PASSWORD 'admin123' 
    SUPERUSER 
    CREATEDB 
    CREATEROLE
    LOGIN;
CREATE ROLE
postgres=# CREATE USER db_admin WITH
    PASSWORD 'dbadmin123'
    CREATEDB
    CREATEROLE
    LOGIN
    NOSUPERUSER;
CREATE ROLE
```

<img width="1291" height="268" alt="image" src="https://github.com/user-attachments/assets/c9280997-d3ef-4741-b4f6-102d6431338f" />

<img width="1232" height="473" alt="image" src="https://github.com/user-attachments/assets/784653b4-5b0b-4d72-8e6b-5978898463ed" />


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
1. ผลการ ALTER USER commands
2. ผลการรัน \dp test_permissions
3. ผลการ GRANT commands
```

<img width="1324" height="99" alt="image" src="https://github.com/user-attachments/assets/e93036bf-335f-4a66-9b58-bf2f3f0b1b39" />

<img width="1283" height="206" alt="image" src="https://github.com/user-attachments/assets/d8e48e96-765b-42f0-a5e9-5149a35d5419" />


<img width="1312" height="449" alt="image" src="https://github.com/user-attachments/assets/cd07255d-06fc-4127-b432-928e8abd29c5" />




**คำถาม
 ```
Access Privileges   postgres=arwdDxtm/postgres มีความหมายอย่างไร
 ```
 ```
postgres=arwdDxtm/postgres
user postgres มีสิทธิ์เต็มทุกอย่างบน object นี้ และสิทธิ์มอบโดย postgres
postgres = คนที่ได้สิทธิ์
arwdDxtm = ชุดสิทธิ์ที่ได้รับ (insert , select , update , truncate ,references , trigger ,update column-level)
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
1. ผลการสร้าง schemas (\dn+)
2. ผลการสร้างตารางในแต่ละ schema
3. ผลการใส่ข้อมูลและ query ข้อมูล
4. ข้อมูลในตาราง employee_orders ที่จะใช้สำหรับ JOIN ข้าม schema
```
<img width="1170" height="391" alt="image" src="https://github.com/user-attachments/assets/54119a86-9994-460c-a3f7-086b100e737e" />

<img width="1220" height="358" alt="image" src="https://github.com/user-attachments/assets/760079d6-0e3e-4d7c-b4b3-b933abecc351" />

<img width="1188" height="164" alt="image" src="https://github.com/user-attachments/assets/571232e3-f469-4565-962e-8b05767e5d7c" />

<img width="1313" height="394" alt="image" src="https://github.com/user-attachments/assets/486fa92d-a7d2-40df-9fb6-ac95ce4bf22d" />

<img width="1315" height="246" alt="image" src="https://github.com/user-attachments/assets/4be87520-529c-4d2a-97f1-e05df47c3dc6" />

<img width="1314" height="338" alt="image" src="https://github.com/user-attachments/assets/0b4c5027-a7b1-4c59-912d-c32f0aa1d9a8" />


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
1. ผลการแสดง search_path
2. ผลการ query ภายใน schema เดียวกัน (sales.customers + sales.orders)
3. ผลการ JOIN ข้าม schemas (sales + hr + inventory)
4. ข้อมูลที่แสดงจาก complex join ข้าม 3 schemas
```

<img width="1334" height="132" alt="image" src="https://github.com/user-attachments/assets/e1efccee-4aaf-4517-b5fd-cf6042bd75e3" />

<img width="1278" height="396" alt="image" src="https://github.com/user-attachments/assets/26f7c704-e941-4e45-8d07-c4196f91e754" />

<img width="1307" height="435" alt="image" src="https://github.com/user-attachments/assets/bfd587b9-cd51-47d1-bde5-b39c174fb62b" />

<img width="1278" height="341" alt="image" src="https://github.com/user-attachments/assets/67c70ac8-1652-4782-a8c0-d0e1b5b66d98" />



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
1. ผลการเชื่อมต่อด้วย lab_user
2. ผลการทดสอบสิทธิ์ต่างๆ
3. ข้อความ error (ถ้ามี) เมื่อไม่มีสิทธิ์
```
<img width="1168" height="91" alt="image" src="https://github.com/user-attachments/assets/dc9cc2e5-839c-4bcb-94f3-96f1fbdc7b61" />

<img width="1191" height="344" alt="image" src="https://github.com/user-attachments/assets/fbbaf159-d59a-428a-b63e-ec795a59de10" />

<img width="1205" height="179" alt="image" src="https://github.com/user-attachments/assets/9c4bcbee-d01a-4848-98d1-3a55625f4e5f" />


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
2. ยืนยันว่าข้อมูลยังอยู่หลังจาก restart
3. ผลการสร้าง container พร้อม bind mount
```

<img width="1185" height="370" alt="image" src="https://github.com/user-attachments/assets/c170d758-9e36-4a41-9e14-6f9d690deb26" />

<img width="1233" height="128" alt="image" src="https://github.com/user-attachments/assets/7859f4ed-5bf1-4ac7-821a-e6a97defec13" />

<img width="1226" height="105" alt="image" src="https://github.com/user-attachments/assets/b8ca2ffc-ab46-4f99-b961-a033226c4ed7" />



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
```

<img width="1187" height="425" alt="image" src="https://github.com/user-attachments/assets/0d423a40-31ac-47f2-abfa-22a5821e4cbe" />


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
2. Memory configuration
3. Active connections
```
<img width="1186" height="412" alt="image" src="https://github.com/user-attachments/assets/db39ff9a-33ec-40a5-8d1f-aa51beeb92cc" />

<img width="1181" height="381" alt="image" src="https://github.com/user-attachments/assets/73914c09-110c-4141-826c-d39a0c2c7ef7" />

<img width="1187" height="321" alt="image" src="https://github.com/user-attachments/assets/13423efd-6c0e-415e-918c-fd3625af6ead" />


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
docker run -d \
  --name multi-postgres \
  -e POSTGRES_PASSWORD=multipass123 \
  -p 5434:5432 \
  --memory="1536m" \
  --cpus="1.5" \
  -v multi-postgres-data:/var/lib/postgresql/data \
  postgres

oneliner:
docker run -d --name multi-postgres -e POSTGRES_PASSWORD=multipass123 -p 5434:5432 --memory="1536m" --cpus="1.5" -v multi-postgres-data:/var/lib/postgresql/data postgres


```

**ผลการทำแบบฝึกหัด 1:**
```
ใส่ Screenshot ของ:
1. คำสั่งที่ใช้สร้าง container
2. docker ps แสดง container ใหม่
3. docker stats แสดงการใช้ resources
```

<img width="1312" height="163" alt="image" src="https://github.com/user-attachments/assets/19c5be62-f4e8-4767-8d82-44d7e800be9f" />

<img width="1305" height="163" alt="image" src="https://github.com/user-attachments/assets/18525851-f851-4a41-abaf-653f86ce4786" />

<img width="1315" height="104" alt="image" src="https://github.com/user-attachments/assets/08863d3d-ba5e-4c75-a094-e0df5a56e598" />


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
-- สร้าง Role Groups
CREATE ROLE app_developers NOLOGIN;
CREATE ROLE data_analysts NOLOGIN;
CREATE ROLE db_admins NOLOGIN;

-- สร้าง Users และกำหนดรหัสผ่าน
CREATE ROLE dev_user LOGIN PASSWORD 'dev123';
CREATE ROLE analyst_user LOGIN PASSWORD 'analyst123';
CREATE ROLE admin_user LOGIN PASSWORD 'admin123';

-- จัด Users เข้ากลุ่ม
GRANT app_developers TO dev_user;
GRANT data_analysts TO analyst_user;
GRANT db_admins TO admin_user;

\du

```

**ผลการทำแบบฝึกหัด 2:**
```
ใส่ Screenshot ของ:
1. การสร้าง roles และ users
2. ผลการรัน \du แสดงผู้ใช้ทั้งหมด
3. ผลการทดสอบเชื่อมต่อด้วย user ต่างๆ
```

<img width="928" height="431" alt="image" src="https://github.com/user-attachments/assets/02936054-0af2-4741-9274-f9b68ccf5a35" />


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

<img width="1311" height="243" alt="image" src="https://github.com/user-attachments/assets/9fe5f877-4d54-4548-9435-0b5ac7ff8649" />

<img width="1313" height="240" alt="image" src="https://github.com/user-attachments/assets/c82179c7-95cc-4446-b9d1-84ba56945ac0" />

<img width="1309" height="288" alt="image" src="https://github.com/user-attachments/assets/efcb53be-1f02-446f-8b0b-ad2a048e9a25" />

<img width="1307" height="357" alt="image" src="https://github.com/user-attachments/assets/d3bf7f5a-981d-4ae2-8df4-db85fc5724ba" />

<img width="1312" height="364" alt="image" src="https://github.com/user-attachments/assets/e03c5fe8-c12b-4f87-8db2-ebe005a81943" />

<img width="1306" height="386" alt="image" src="https://github.com/user-attachments/assets/4f4acc14-399d-4103-905c-c4566db1bf37" />

<img width="1313" height="432" alt="image" src="https://github.com/user-attachments/assets/fe50a451-a420-4926-b98f-23d2dd4eb93b" />

2. ข้อมูลตัวอย่างในตารางต่างๆ

<img width="1305" height="234" alt="image" src="https://github.com/user-attachments/assets/3dea4e54-33aa-4d61-ad6f-bed50def678c" />

<img width="1314" height="353" alt="image" src="https://github.com/user-attachments/assets/c43a66d2-8ff6-4394-b84b-6f0ce374ed8b" />

<img width="1306" height="295" alt="image" src="https://github.com/user-attachments/assets/655cda25-4d3b-4a57-8ab3-30c52daa1824" />


3. ผลการรัน queries ที่สร้าง

<img width="1306" height="376" alt="image" src="https://github.com/user-attachments/assets/adbb4530-a6e5-48a0-befe-0efc98b844a8" />


4. การวิเคราะห์ข้อมูลที่ได้

<img width="1300" height="230" alt="image" src="https://github.com/user-attachments/assets/af9fa1b8-a5bd-46b9-aa30-177a2768d067" />


## การทดสอบความเข้าใจ

### Quiz 1: Conceptual Questions
ตอบคำถามต่อไปนี้:

1. อธิบายความแตกต่างระหว่าง Named Volume และ Bind Mount ในบริบทของ PostgreSQL
2. เหตุใด shared_buffers จึงควรตั้งเป็น 25% ของ RAM?
3. การใช้ Schema ช่วยในการจัดการฐานข้อมูลขนาดใหญ่อย่างไร?
4. อธิบายประโยชน์ของการใช้ Docker สำหรับ Database Development

**คำตอบ Quiz 1:**
```
1.Named Volume Docker จะจัดการได้โดยตรง แต่ Bind Mount ผูก (mount) โฟลเดอร์ในเครื่องเข้าใน container โดยตรง
2. เพราะเป็นค่าที่พอดี ถ้าตั้งน้อยไปก็ต้องอ่านจาก disk บ่อยจนช้า แต่ถ้าตั้งเยอะไปก็แย่ง RAM จากระบบปฏิบัติการจนไม่เหลือให้ OS cache ข้อมูล
3.แยกข้อมูลตามงานได้ ช่วยป้องกันการชนกันของชื่อตาราง รวมถึงจัดการสิทธิ์การเข้าถึงได้ง่ายขึน
4. ใช้งานง่าย สามารถแยกโปรเจกต์ออกจากกันไม่ให้ข้อมูลปนกันได้และทดสอบง่าย
```


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
