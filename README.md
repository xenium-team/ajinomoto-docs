# ajinomoto-docs

# ขั้นตอนเตรียมข้อมูล & Deploy 
## 🔹 เตรียมข้อมูล

### 1.เพิ่ม Subdomain

จำนวน: 3 Domain

```
ตัวอย่าง:
[URL_API] https://kooranconnect-api.xenium-dev.site
[URL_LIFF] https://kooranconnect-liff.xenium-dev.site
[URL_ADMIN] https://kooranconnect-admin.xenium-dev.site

```

### 2. postgres 17.12 ขึ้นไป

หรือใช้เซิร์ฟเวอร์เดียวกับ taily ได้
สร้าง database name : ajinomoto



### Server Requirements

สำหรับการติดตั้งระบบ AJINOMOTO บน Production Server ต้องมีทรัพยากรขั้นต่ำดังนี้

| รายการ | Minimum Requirements | Recommended |
| --- | --- | --- |
| CPU | 2 vCPU | 4 vCPU |
| RAM | 4 GB | 8 GB |
| Storage | 40 GB SSD | 100 GB SSD |
| Operating System | Ubuntu 22.04 LTS | Ubuntu 24.04 LTS |
| Docker | Docker Engine + Compose V2 | Latest supported version |
| Database | PostgreSQL 17.12+ | PostgreSQL 17.12+ |

### หมายเหตุ

- พื้นที่ Storage ต้องเผื่อสำหรับ Database, Docker Images, Application Logs และ Database Backup
- หากใช้งาน PostgreSQL Server ร่วมกับระบบอื่น ควรจัดสรร CPU, RAM และ Storage เพิ่มเติมตามปริมาณการใช้งาน
- ข้อกำหนดนี้เป็นค่าเริ่มต้นสำหรับระบบขนาดเล็กถึงปานกลาง ควรประเมินทรัพยากรอีกครั้งตามจำนวนผู้ใช้งานและปริมาณข้อมูลจริง
- สเปกดังกล่าวเป็นข้อเสนอเบื้องต้น ยังไม่ได้ผ่านการทดสอบ Load Test ของระบบ AJINOMOTO

### 3. ตั้งค่าไฟล์

🔹เพิ่ม Domain

ไฟล์ apps\admin\vite.config.ts

ไฟล์ apps\liff\vite.config.ts

```
ตัวอย่าง:
allowedHosts: [".xenium-dev.site"]
```

🔹 k8s\docker\web.Dockerfile

ARG VITE_API_URL=[URL_API]
ARG VITE_LIFF_ID= กำหนดตามเอกสาร SECRET.md

🔹docker-compose.prod.yml
```
environment api และ liff กำหนดตามเอกสาร SECRET.md

  JWT_SECRET: 
  TOKEN_ENCRYPTION_KEY: 
  METRICS_TOKEN: 
  LINE_CHANNEL_ID: 
  LINE_CHANNEL_SECRET: 
  LINE_MESSAGING_TOKEN: 
  LINE_RICHMENU_REGISTERED_ID:
  LIFF_ID:
```      
🔹เปลียน DATABASE_URL ของ api และ worker

```
ตัวอย่าง:
DATABASE_URL=postgresql://postgres:password@db.example.com:5432/ajinomoto
```
🔹เปลียน CORS_ORIGINS api 
[URL_ADMIN],[URL_LIFF]

```
ตัวอย่าง:
CORS_ORIGINS: https://kooranconnect-admin.xenium-dev.site,https://kooranconnect-liff.xenium-dev.site
```

🔹เปลียน PUBLIC_BASE_URL  api 
[URL_API]

```
ตัวอย่าง:
PUBLIC_BASE_URL: https://kooranconnect-api.xenium-dev.site
```

🔹เปลียน VITE_API_URL  admin 
[URL_API]
```
 VITE_API_URL: https://kooranconnect-api.xenium-dev.site
```
🔹เปลียน VITE_API_URL  liff 
[URL_API]
```
 VITE_API_URL: https://kooranconnect-api.xenium-dev.site
```

### 4. Build
```
docker compose -f docker-compose.prod.yml build 

docker compose -f docker-compose.prod.yml up -d

docker compose -f docker-compose.prod.yml exec api node dist/database/migrate.js up
docker compose -f docker-compose.prod.yml exec api node dist/scripts/stores-import.js --dev-sample
```

USERNAME และ ADMIN_PASSWORD  กำหนดตามเอกสาร SECRET.md
```
docker compose -f docker-compose.prod.yml exec api \
  node dist/scripts/create-admin.js \
  --username [USERNAME] \
  --name "Super Admin" \
  --super

docker compose -f docker-compose.prod.yml exec \
  -e ADMIN_PASSWORD=[ADMIN_PASSWORD] \
  api node dist/scripts/create-admin.js \
  --username superadmin --reset
```
### 4. Test 
ส่ง URL_LIFF และ URL_ADMIN กลับมาให้ ทางทีมทดสอบ 
เมื่อผ่านแล้วจะ แจ้งให้รัน อีกครั้ง
🔹docker-compose.prod.yml

เปลียน

ไฟล์ k8s\docker\web.Dockerfile

admin VITE_DEV: "false"

ไฟล์ k8s\docker\web.Dockerfile

VITE_DEV="false"

แล้วรัน 

docker compose -f docker-compose.prod.yml up -d --build --force-recreate
