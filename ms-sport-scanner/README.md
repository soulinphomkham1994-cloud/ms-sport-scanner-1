# 🏭 MS SPORT — Production Status Scanner

ລະບົບສະແກນ QR Code ເພື່ອອັບເດດສະຖານະການຜະລິດ ໃນ Google Sheets ແບບ Real-time  
สแกน QR Code แล้วอัปเดตสถานะลง Google Sheets ทันที — deploy บน Vercel + GitHub

---

## 📁 โครงสร้างโปรเจ็ก

```
ms-sport-scanner/
├── public/
│   └── index.html          ← หน้า UI หลัก (สแกน + เลือกสถานะ)
├── api/
│   └── update-status.js    ← Vercel Serverless Function (เขียนลง Google Sheets)
├── package.json
├── vercel.json
├── .gitignore
└── README.md
```

---

## 🔧 วิธีตั้งค่าทีละขั้น

### ขั้นที่ 1 — สร้าง Google Service Account

1. ไปที่ [Google Cloud Console](https://console.cloud.google.com/)
2. สร้าง Project ใหม่ (หรือใช้อันที่มีอยู่)
3. เปิด **APIs & Services → Library** → ค้นหา **Google Sheets API** → กด **Enable**
4. ไปที่ **APIs & Services → Credentials** → กด **Create Credentials → Service Account**
5. ตั้งชื่อ เช่น `ms-sport-scanner` → กด **Create and Continue** → **Done**
6. คลิก Service Account ที่สร้าง → ไปแท็บ **Keys** → **Add Key → Create new key → JSON**
7. ดาวน์โหลดไฟล์ `.json` — จะได้ข้อมูลสำคัญ 2 อย่าง:
   - `client_email` → ใช้เป็น `GOOGLE_CLIENT_EMAIL`
   - `private_key`  → ใช้เป็น `GOOGLE_PRIVATE_KEY`

### ขั้นที่ 2 — แชร์ Google Sheet ให้ Service Account

1. เปิด Google Sheet:  
   `https://docs.google.com/spreadsheets/d/1lPp5LuwT6lAt0wozRduI8ErarHk-5iwI5PWoZwmTZoY/`
2. กดปุ่ม **Share** (แชร์)
3. ใส่ email ของ Service Account (เช่น `ms-sport-scanner@your-project.iam.gserviceaccount.com`)
4. ให้สิทธิ์ **Editor** → กด **Send**

### ขั้นที่ 3 — Upload โปรเจ็กขึ้น GitHub

```bash
# เปิด Terminal แล้วรันคำสั่งนี้
git init
git add .
git commit -m "Initial MS SPORT Scanner"

# สร้าง repo ใหม่ใน GitHub แล้วรัน:
git remote add origin https://github.com/YOUR_USERNAME/ms-sport-scanner.git
git branch -M main
git push -u origin main
```

### ขั้นที่ 4 — Deploy บน Vercel

1. ไปที่ [vercel.com](https://vercel.com) → Login ด้วย GitHub
2. กด **Add New Project** → เลือก repo `ms-sport-scanner`
3. กด **Deploy** (ไม่ต้องเปลี่ยน settings ใดๆ)
4. หลัง deploy เสร็จ → ไปที่ **Settings → Environment Variables**
5. เพิ่ม Environment Variables ดังนี้:

| Variable Name        | Value                                      |
|----------------------|--------------------------------------------|
| `GOOGLE_CLIENT_EMAIL`| `your-service-account@project.iam.gserviceaccount.com` |
| `GOOGLE_PRIVATE_KEY` | `-----BEGIN RSA PRIVATE KEY-----\n...`    |
| `SPREADSHEET_ID`     | `1lPp5LuwT6lAt0wozRduI8ErarHk-5iwI5PWoZwmTZoY` |
| `SHEET_NAME`         | `2026`                                     |
| `INV_COLUMN`         | `5`  (คอลัมน์ E)                           |
| `STATUS_COLUMN`      | `6`  (คอลัมน์ F)                           |

> ⚠️ สำหรับ `GOOGLE_PRIVATE_KEY` ให้วาง private key ทั้งหมดจากไฟล์ JSON รวมถึง `-----BEGIN...-----`  
> Vercel จะจัดการ `\n` ให้อัตโนมัติ

6. หลังเพิ่ม env vars เสร็จ → กด **Redeploy** (ใน Deployments → ⋯ → Redeploy)

---

## 🗂️ โครงสร้าง Google Sheet ที่ต้องการ

Sheet ชื่อ `2026` ต้องมีข้อมูลแบบนี้:

| A | B | C | D | **E (Invoice No)** | **F (Status)** |
|---|---|---|---|-------------------|----------------|
|   |   |   |   | INV-001           | ກຳລັງດຳເນີນການ |
|   |   |   |   | INV-002           | ກຳລັງພິມແບບ P1 |
|   |   |   |   | INV-003           |                |

- **คอลัมน์ E** → หมายเลข Invoice (เช่น `INV-001`, `INV-2025-0042`)
- **คอลัมน์ F** → สถานะ (app จะเขียนค่านี้)

---

## 📱 วิธีใช้งาน

1. เปิด app บนมือถือผ่าน URL จาก Vercel (ต้องเป็น HTTPS)
2. **เลือกสถานะ** จาก dropdown ก่อน เช่น `ກຳລັງພິມແບບ P1`
3. กด **ເປີດກ້ອງຫຼັງ** → ส่อง QR Code บนใบกำกับการผลิต
4. App สแกนเจอ INV code → อัปเดต Google Sheet ทันที
5. ดูผลและประวัติได้ที่ด้านล่าง

### ทางเลือกถ้ากล้องไม่ทำงาน:
- กด **ອັບໂຫຼດຮູບ** → ถ่ายรูป QR แล้วเลือกไฟล์
- หรือพิมพ์ INV code เองในช่อง → กด **ບັນທຶກ**

---

## 🔄 การสร้าง QR Code

QR Code ที่ใช้สแกนควรมีข้อความเป็น Invoice No เท่านั้น เช่น:
```
INV-001
```
หรือมีข้อความอื่นก็ได้ — app จะดึงเฉพาะส่วน `INV-XXXXX` ออกมาอัตโนมัติ

สามารถสร้าง QR Code ได้จากเว็บ เช่น [qr-code-generator.com](https://www.qr-code-generator.com/)  
โดยใส่ค่าจาก column E ของ Google Sheet

---

## ❓ แก้ปัญหาที่พบบ่อย

| ปัญหา | วิธีแก้ |
|-------|---------|
| `Missing GOOGLE_CLIENT_EMAIL` | ตรวจสอบ Environment Variables ใน Vercel |
| `ບໍ່ພົບໃບກຳກັບ` | ตรวจสอบว่า INV code ตรงกับ column E ใน Sheet |
| กล้องไม่เปิด | ต้องเปิดผ่าน HTTPS เท่านั้น |
| `403 Forbidden` จาก Sheets | แชร์ Sheet ให้ Service Account email แล้วหรือยัง? |

---

## 📞 ติดต่อ

MS SPORT: **+8562057519669**
