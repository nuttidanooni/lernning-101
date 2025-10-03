Tasky – แอปจัดการงาน (Go + Vue)
ภาพรวม

Tasky เป็นโปรเจกต์ฝึกหัด full‑stack สำหรับผู้เริ่มต้น ตั้งแต่ไม่เคยเขียนโปรแกรม คุณจะได้เรียนรู้การพัฒนาเว็บด้วย Go (Gin), PostgreSQL และ Vue 3 รวมถึงการจัดการคอนเทนเนอร์ด้วย Docker และ docker‑compose.

ฟีเจอร์ของแอป

สมัครสมาชิกและล็อกอินด้วย JWT

สร้าง/อ่าน/แก้ไข/ลบงาน (CRUD) พร้อมสถานะ (todo, doing, done) และกำหนดวันครบกำหนด

แสดงรายการงานในหน้าเว็บพร้อมแบบฟอร์มเพิ่ม/แก้ไข

เทคโนโลยีที่ใช้

Backend: Go 1.23 (Gin) + PostgreSQL

Frontend: Vue 3 + Vite + axios

สื่อสารผ่าน REST API

ใช้ Docker และ docker‑compose ติดตั้งง่าย

การเริ่มต้นใช้งาน
การรันแบบ production

ติดตั้ง Docker และ docker‑compose บนเครื่องของคุณ

โคลน repo นี้แล้วเข้าไปในโฟลเดอร์

เรียกคำสั่ง

docker compose up -d --build


docker‑compose จะสร้าง service ทั้งสาม: db, api และ web และรันขึ้นมาใน background

ตรวจสอบว่าบริการทำงานแล้ว:

API: http://localhost:8080/health

เว็บ: http://localhost:5173

เมื่อไม่ใช้งานสามารถหยุดได้ด้วย docker compose down

ค่าเริ่มต้นของฐานข้อมูลและ secret สามารถเปลี่ยนได้ใน docker-compose.yml:

POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB

JWT_SECRET สำหรับเซ็น JWT

พอร์ต 8080 และ 5173 สามารถปรับได้หากชนกับพอร์ตอื่น

การพัฒนา (hot reload)

หากต้องการแก้ไขโค้ดแล้วเห็นผลทันที ให้ใช้ service api-dev ที่ตั้งค่า Air (live reload) และ Makefile เพื่อช่วย

รันชุด dev ด้วย make:

make dev


คำสั่งนี้จะ build และรัน service db, api-dev และ web พร้อมแมปโวลุ่มโค้ด local สำหรับ hot reload

แก้ไขไฟล์ Go ใน backend/ แล้ว Air จะ rebuild อัตโนมัติและรีสตาร์ท API

แก้ไขไฟล์ Vue ใน frontend/ แล้ว Vite จะ reload หน้าเว็บโดยอัตโนมัติ

Makefile มี target ที่ช่วยงานเช่น:

make dev – ขึ้น service สำหรับ dev

make tidy – รัน go mod tidy ใน container

make restart – รีสตาร์ท service api-dev

make down – หยุดทุก service

make psql – เข้าสู่ psql shell ของฐานข้อมูล

การแก้ไขพอร์ต dev

ถ้ามีพอร์ต 8080 ถูกใช้อยู่แล้ว สามารถแก้ docker-compose.yml ส่วน api-dev ให้แมปพอร์ตใหม่ เช่น "8081:8080" และตั้ง VITE_API_BASE ใน service web ให้ชี้ไปพอร์ตนั้น

โครงสร้างโฟลเดอร์หลัก
tasky/
├─ docker-compose.yml    # กำหนด service และ environment
├─ Makefile              # คำสั่งลัดสำหรับพัฒนา
├─ backend/              # โค้ด Go API
│  ├─ Dockerfile         # สำหรับ build production
│  ├─ Dockerfile.dev     # สำหรับ dev ใช้ Air
│  ├─ .air.toml          # การตั้งค่า Air
│  ├─ go.mod / go.sum
│  ├─ main.go
│  ├─ internal/
│  │  ├─ db/             # ฟังก์ชันเชื่อมฐานข้อมูล
│  │  ├─ auth/           # JWT helper
│  │  ├─ models/         # struct สำหรับ Users/Tasks
│  │  └─ handlers/       # HTTP handler (auth, tasks)
│  └─ migrations/        # SQL สำหรับสร้างตาราง
└─ frontend/             # โค้ด Vue3 + Vite
   ├─ Dockerfile         # รัน dev server
   ├─ package.json
   ├─ vite.config.ts
   ├─ index.html
   └─ src/
      ├─ main.ts
      ├─ api.ts          # ตั้ง baseURL และ token
      ├─ router.ts       # กำหนด routes
      ├─ App.vue
      └─ pages/
         ├─ Login.vue
         └─ Tasks.vue

ฐานข้อมูล

ตารางสร้างผ่านไฟล์ migration SQL ใน backend/migrations/001_init.sql:

users – เก็บอีเมลและรหัสผ่าน (hash) ของผู้ใช้

tasks – เก็บงานของแต่ละ user พร้อมสถานะและ due date

ค่า PRIMARY KEY เป็น serial auto‑increment และใช้ user_id เป็น foreign key เชื่อมกับตาราง users

การ Deploy จริง

เพื่อ deploy แอปนี้:

สั่ง build image production

docker compose build api web


รันในเซิร์ฟเวอร์ด้วย docker compose up -d

ตั้ง reverse proxy (เช่น nginx) ให้ชี้มายังพอร์ตของ API และ frontend ตามต้องการ

เปลี่ยนค่า JWT_SECRET เป็นค่าสุ่มที่ปลอดภัย และใช้ฐานข้อมูลจริงแทน container local

การมีส่วนร่วม

โปรเจกต์นี้เหมาะสำหรับการศึกษาต่อยอด:

เพิ่มฟิลด์ในตาราง tasks เช่น priority หรือ tag

เพิ่ม search/filter ในหน้ารายการงาน

เขียน Unit Test ฝั่ง Go

ใช้ UI library สำหรับ frontend เพื่อประสบการณ์ผู้ใช้ที่ดีขึ้น