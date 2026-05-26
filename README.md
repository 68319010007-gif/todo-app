# DevOps Todo App — สัปดาห์ที่ 2

## Tech Stack
- **Backend**: Node.js + Express.js (port 3001)
- **Database**: PostgreSQL 16 (Docker)
- **Frontend**: Vanilla HTML/CSS/JS

## วิธีรันโปรเจกต์

### 1. Prerequisites
- Node.js v20+
- Docker Desktop (ต้องรันอยู่)

### 2. Clone โปรเจกต์
```bash
git clone git@github.com:YOUR-USERNAME/todo-app.git
cd todo-app
```

### 3. ตั้งค่า Database (รันครั้งเดียว)
```bash
docker run -d \
  --name todo-postgres \
  -e POSTGRES_DB=tododb \
  -e POSTGRES_USER=todouser \
  -e POSTGRES_PASSWORD=todopass123 \
  -p 5432:5432 \
  postgres:16-alpine
```

### 4. สร้างตาราง (รันครั้งเดียว)
```bash
docker exec -it todo-postgres psql -U todouser -d tododb \
  -c "CREATE TABLE todos (id SERIAL PRIMARY KEY, title VARCHAR(255) NOT NULL, completed BOOLEAN DEFAULT FALSE, created_at TIMESTAMPTZ DEFAULT NOW());"
```

### 5. ติดตั้ง Dependencies
```bash
cd backend && npm install
```

### 6. สร้างไฟล์ backend/.env
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=tododb
DB_USER=todouser
DB_PASSWORD=todopass123
PORT=3001
```

### 7. รัน Backend
```bash
npm run dev
```

### 8. เปิด Frontend
เปิดไฟล์ `frontend/index.html` ด้วย Browser

## API Endpoints
| Method | Path | คำอธิบาย |
|--------|------|-----------|
| GET | /health | ตรวจสอบสถานะ server |
| GET | /api/todos | ดึง Todo ทั้งหมด |
| POST | /api/todos | เพิ่ม Todo ใหม่ |
| PATCH | /api/todos/:id | toggle completed |
| DELETE | /api/todos/:id | ลบ Todo |