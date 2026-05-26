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
| PUT | /api/todos/:id | แก้ไขชื่อ Todo |
| DELETE | /api/todos/:id | ลบ Todo |

## การเปลี่ยนแปลง Frontend (UI Redesign)

### ออกแบบใหม่ทั้งหมด
- เปลี่ยนธีมเป็น **Dark Theme** สไตล์ GitHub เพื่อให้เหมาะกับ Developer
- ใช้ฟอนต์ **IBM Plex Sans Thai** (body) + **JetBrains Mono** (heading/stats)
- ออกแบบโดยไม่พึ่ง CSS Framework ใดๆ ทั้งสิ้น

### ฟีเจอร์ใหม่
- **แก้ไขข้อความ (Inline Edit)** — กดปุ่ม ✏️ เพื่อแก้ไข Todo ได้โดยตรง
  - กด `Enter` หรือปุ่ม ✓ เพื่อบันทึก
  - กด `Escape` หรือปุ่ม ✕ เพื่อยกเลิก
  - เรียก `PUT /api/todos/:id` ไปยัง Backend
- **Stats Cards** — แสดงจำนวนงานทั้งหมด / เสร็จแล้ว / รอดำเนินการ แบบ real-time
- **Progress Bar** — แสดงเปอร์เซ็นต์งานที่เสร็จแล้วด้วย animation
- **Slide-in Animation** — Todo ใหม่จะมี animation เลื่อนเข้ามาเมื่อเพิ่ม
- **Empty State** — แสดงข้อความเมื่อยังไม่มีงานแทนการแสดงหน้าว่าง

### ไฟล์ที่แก้ไข
- `frontend/index.html` — ปรับ UI ใหม่ทั้งหมด เพิ่ม inline edit และ stats

### Backend ที่ต้องเพิ่ม (PUT endpoint)
เพื่อรองรับการแก้ไขข้อความ ต้องเพิ่ม endpoint นี้ใน `backend/index.js`:

```js
app.put('/api/todos/:id', async (req, res) => {
  const { id } = req.params;
  const { title } = req.body;
  if (!title || !title.trim()) return res.status(400).json({ error: 'title required' });
  const result = await pool.query(
    'UPDATE todos SET title=$1 WHERE id=$2 RETURNING *',
    [title.trim(), id]
  );
  if (result.rows.length === 0) return res.status(404).json({ error: 'not found' });
  res.json(result.rows[0]);
});
```