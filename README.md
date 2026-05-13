# Terminal Maintenance Issue Tracker (TMIT)

ระบบติดตามการแก้ไขประเด็นจาก Safety Tour — รองรับหลาย Terminal บน GitHub Pages + Supabase

---

## 📁 โครงสร้างไฟล์

```
repository/
├── index.html          ← Single-file web app ทั้งหมด
├── README.md           ← ไฟล์นี้
└── supabase_setup.sql  ← SQL สำหรับสร้าง tables + RLS + seed data
```

---

## 🚀 ขั้นตอนการ Setup ทั้งหมด

### STEP 1 — สร้าง Supabase Project

1. ไปที่ [https://supabase.com](https://supabase.com) → Sign In → **New Project**
2. ตั้งชื่อ Project เช่น `tmit-or`
3. เลือก Region: **Southeast Asia (Singapore)**
4. ตั้ง Database Password (เก็บไว้ให้ดี)
5. รอ ~2 นาที ให้ Project พร้อม

---

### STEP 2 — รัน SQL Setup

1. ใน Supabase Dashboard → **SQL Editor** (ไอคอน `</>` ด้านซ้าย)
2. คลิก **New Query**
3. เปิดไฟล์ `supabase_setup.sql` แล้ว **Copy ทั้งหมด** วางในช่อง Query
4. คลิก **Run** (▶) รอจนขึ้น `Success`

> ⚠️ ถ้า Error "already exists" — ปกติ ข้ามได้เลย (แค่ตาราง/index มีอยู่แล้ว)

---

### STEP 3 — สร้าง Super Admin User

1. Supabase Dashboard → **Authentication** → **Users** → **Add User** → **Create New User**
2. กรอก:
   - Email: `admin@or-terminal.local`
   - Password: (ตั้งรหัสที่แข็งแกร่ง)
3. คลิก **Create User**
4. คัดลอก UUID ของ User ที่สร้าง (คอลัมน์ ID)
5. กลับไป **SQL Editor** รัน SQL นี้ (แทน `<UUID>` ด้วย UUID จริง):

```sql
INSERT INTO public.profiles (id, display_name, role, must_change_password)
VALUES ('<UUID>', 'Super Admin', 'super_admin', false);
```

---

### STEP 4 — ดึง API Keys

1. Supabase Dashboard → **Settings** (⚙) → **API**
2. คัดลอก 2 ค่า:
   - **Project URL** → เช่น `https://abcxyz123.supabase.co`
   - **anon public key** → สตริงยาว

---

### STEP 5 — แก้ไข index.html

เปิดไฟล์ `index.html` แล้วแก้ใน `<script>` ส่วนบน:

```javascript
// 🔧 แก้ค่าเหล่านี้ก่อน deploy
const SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co';     // ← แทนด้วย URL จริง
const SUPABASE_ANON_KEY = 'YOUR_ANON_KEY';                   // ← แทนด้วย key จริง
```

ตัวอย่าง:
```javascript
const SUPABASE_URL = 'https://abcxyz123.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

### STEP 6 — ตั้งค่า Allowed Origins ใน Supabase

1. Supabase Dashboard → **Authentication** → **URL Configuration**
2. **Site URL**: ใส่ URL GitHub Pages เช่น `https://leomcpemail-source.github.io`
3. **Redirect URLs**: ใส่ `https://leomcpemail-source.github.io/tmit/**`

---

### STEP 7 — Push ขึ้น GitHub Pages

```bash
# สมมติ repository ชื่อ tmit
git init
git add .
git commit -m "Initial: Terminal Maintenance Issue Tracker"
git remote add origin https://github.com/leomcpemail-source/tmit.git
git push -u origin main
```

จากนั้น:
1. GitHub → Repository → **Settings** → **Pages**
2. Source: **Deploy from branch** → Branch: `main` → Folder: `/` (root)
3. คลิก **Save**
4. รอ ~1 นาที → URL จะเป็น `https://leomcpemail-source.github.io/tmit/`

---

### STEP 8 — เข้าใช้งานครั้งแรก

1. เปิด URL GitHub Pages
2. Login ด้วย `admin@or-terminal.local` + รหัสที่ตั้ง
3. เลือก Terminal → เข้าสู่ Dashboard

---

## 👤 การเพิ่ม User ใหม่

### วิธีที่ 1 — ผ่าน Admin Panel (แนะนำ)
1. Login ด้วย Super Admin หรือ Terminal Admin
2. ไป **Admin Panel** → Tab **User Management**
3. คลิก **+ เพิ่มผู้ใช้**
4. กรอก ชื่อ, Email, รหัสชั่วคราว, Role, Terminal
5. บันทึก → แจ้ง User ว่า email + temp password คืออะไร
6. User login ครั้งแรก → ระบบบังคับเปลี่ยนรหัสผ่านทันที

> ⚠️ ฟีเจอร์นี้ใช้ `supabase.auth.admin.createUser()` ซึ่งต้องใช้ **Service Role Key** ไม่ใช่ Anon Key  
> สำหรับ production ควรสร้าง Edge Function แทน หรือใช้ Supabase Dashboard สร้าง User แล้วใส่ profile ด้วย SQL

### วิธีที่ 2 — ผ่าน Supabase Dashboard + SQL
```sql
-- 1. สร้าง user ใน Auth (Dashboard → Authentication → Add User)
-- 2. คัดลอก UUID แล้วรัน:

INSERT INTO public.profiles (id, display_name, role, must_change_password)
VALUES ('<UUID>', 'ชื่อพนักงาน', 'engineer', true);

-- 3. กำหนด Terminal ให้ user (ถ้าไม่ใช่ super_admin):
INSERT INTO public.user_terminal_access (user_id, terminal_id, role)
VALUES (
  '<USER_UUID>',
  (SELECT id FROM terminals WHERE code = 'BKK'),
  'engineer'
);
```

---

## 🏭 การเพิ่ม Terminal ใหม่

### ผ่าน Supabase SQL:
```sql
INSERT INTO public.terminals (code, name, region)
VALUES ('CNX', 'คลังปิโตรเลียมเชียงใหม่', 'เหนือ');
```

### เพิ่ม Locations ให้ Terminal ใหม่:
```sql
WITH t AS (SELECT id FROM terminals WHERE code = 'CNX')
INSERT INTO public.locations (terminal_id, area, sub_area)
SELECT t.id, a.area, a.sub_area FROM t, (VALUES
  ('บริเวณถังเก็บ', 'Tank Farm A'),
  ('บริเวณจ่ายน้ำมัน', 'Loading Rack 1'),
  ('อาคาร/สาธารณูปโภค', 'Control Room')
) AS a(area, sub_area);
```

> Super Admin ล็อกอินแล้วจะเห็น Terminal ใหม่ใน Terminal Selector ทันที — ไม่ต้อง deploy code ใหม่

---

## 🔐 Role & Permission สรุป

| Role | สิทธิ์ |
|------|--------|
| **super_admin** | เข้าถึงทุก Terminal, จัดการทุกอย่าง |
| **terminal_admin** | จัดการ User + Master Data ใน Terminal ตัวเอง |
| **engineer** | สร้าง/แก้ไข/ลบ Issue ใน Terminal ตัวเอง |
| **viewer** | ดูข้อมูลอย่างเดียว |

---

## 🛠 Troubleshooting

| ปัญหา | วิธีแก้ |
|-------|---------|
| Login แล้ว redirect loop | ตรวจ `profiles` table ว่ามี row ของ user หรือยัง |
| ไม่เห็น Issue ของ Terminal | ตรวจ `user_terminal_access` ว่า assign user → terminal ถูกต้อง |
| RLS Error "row-level security policy" | ตรวจ RLS policy ว่า user มีสิทธิ์ใน terminal นั้น |
| CSS ไม่ work บน GitHub Pages | ไฟล์เดียว (single-file) ไม่มีปัญหา path แล้ว |
| CORS Error | ตรวจ Supabase → Authentication → URL Configuration |
| Export Excel/PDF ขึ้น Error | ตรวจว่า CDN โหลด jspdf และ xlsx ได้ (ต้องมี internet) |

---

## 📊 Database Tables สรุป

```
terminals          — Master list ของ Terminal ทั้งหมด
profiles           — ข้อมูล User + Role (เชื่อมกับ auth.users)
user_terminal_access — mapping user ↔ terminal (many-to-many)
issues             — รายการ Issue ทั้งหมด (มี terminal_id)
locations          — พื้นที่/พื้นที่ย่อย แยกตาม terminal
budget_types       — ประเภทงบ (CAPEX, OPEX-CM, OPEX-PM, Other)
issue_edit_log     — Log ทุกการแก้ไข Issue (สำหรับ Admin Report)
audit_log          — Log ระดับ System (INSERT/UPDATE/DELETE)
```

---

## 📝 Notes

- **Single-file architecture**: CSS + JS รวมใน `index.html` ไฟล์เดียว แก้ปัญหา path บน GitHub Pages
- **Session**: เก็บใน `sessionStorage` — หมดอายุเมื่อปิด Browser (ปลอดภัยกว่า localStorage)
- **Issue ID**: Format `{TERMINAL_CODE}-{NNN}` เช่น BKK-001, RYG-023
- **RLS**: บังคับทุก table — ไม่มีทางเข้าถึงข้อมูล terminal อื่นได้จาก frontend
- **Font**: Sarabun (Google Fonts) — รองรับภาษาไทยครบถ้วน

---

*Last updated: May 2026 | Stack: GitHub Pages + Supabase + Vanilla JS*
