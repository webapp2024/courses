# LIFF — คู่มือ Deploy (GitHub Pages)

หน้า LIFF สำหรับ **ครู** และ **ผู้ปกครอง** เสิร์ฟบน GitHub Pages  
เพราะ GAS HtmlService ไม่รองรับ `liff.init()`

---

## โครงสร้างไฟล์

```
liff/
├── liff.html       ← หน้าเดียวสำหรับทั้งครูและผู้ปกครอง
├── config.js       ← ⚠️ ต้องแก้ค่าก่อน deploy
├── style.css       ← สไตล์
└── README-deploy.md
```

### liff.html ทำอะไรบ้าง

**สำหรับครู** (login ด้วย username/password)
- หน้าแรก — โปรไฟล์ + โควต้าวันลา
- เช็คชื่อ — แตะการ์ดนักเรียนเพื่อสลับสถานะ (homeroom เท่านั้น)
- รออนุมัติ — อนุมัติ/ปฏิเสธใบลาพร้อม comment
- ประวัติ — รายการที่เคยอนุมัติ
- ใบลาของฉัน — ยื่นและติดตามใบลาตัวเอง

**สำหรับผู้ปกครอง** (auth ด้วย LINE User ID)
- ยื่นใบลาแทนบุตรหลาน
- ดูสถานะใบลา
- รับแจ้งเตือนเมื่อขาด/สาย และเมื่ออนุมัติ/ปฏิเสธ

---

## วิธี Deploy

### ขั้นที่ 1 — อัปโหลดขึ้น GitHub Pages

1. สร้าง Repository ใหม่ใน GitHub (เช่น `school-liff`)
2. อัปโหลดไฟล์ทุกไฟล์ในโฟลเดอร์ `liff/` ขึ้นไป  
   ให้ `liff.html` อยู่ที่ root ของ repo
3. ไปที่ **Settings > Pages**
4. Source: `Deploy from a branch` → branch `main` → folder `/ (root)`
5. กด **Save** → รอ 1–2 นาที
6. ได้ URL เช่น `https://USERNAME.github.io/school-liff/`

### ขั้นที่ 2 — สร้าง LIFF App

1. เข้า [LINE Developers Console](https://developers.line.biz)
2. เลือก Provider → เลือก Channel (Messaging API)
3. แท็บ **LIFF** → **Add**
4. กรอก:
   - LIFF app name: `ระบบเช็คชื่อ`
   - Size: `Full`
   - Endpoint URL: `https://USERNAME.github.io/school-liff/liff.html`
   - Scope: ติ๊ก `profile`
   - Bot link feature: `On (Aggressive)` แนะนำ
5. กด **Add** → คัดลอก **LIFF ID** ที่ได้

### ขั้นที่ 3 — แก้ config.js

เปิดไฟล์ `liff/config.js` แก้ 2 ค่า:

```js
const GAS_API_URL = "https://script.google.com/macros/s/XXXX/exec";
//                   ↑ Web App URL จาก Google Apps Script (ขั้นที่ 6 ใน gas/README.md)

const LIFF_ID = "1234567890-abcdEFGH";
//               ↑ LIFF ID จากขั้นที่ 2
```

แล้ว push ขึ้น GitHub อีกครั้ง

### ขั้นที่ 4 — ใส่ URL ในระบบ

1. เข้า Admin Web → เมนู **ตั้งค่า**
2. ใส่ LIFF URL: `https://liff.line.me/LIFF_ID`
3. กด **บันทึก**

---

## การเชื่อมต่อ API

ทุก request ส่งแบบ POST ไป GAS:

```js
fetch(GAS_API_URL, {
  method: 'POST',
  body: JSON.stringify({ action: 'actionName', ...params })
})
```

> ไม่ตั้ง `Content-Type` header → เป็น `text/plain` → ไม่เกิด CORS preflight

---

## การ Update LIFF

เมื่อแก้ไข `liff.html` หรือ `config.js`:

```bash
git add .
git commit -m "update"
git push
```

GitHub Pages อัปเดตอัตโนมัติภายใน 1–2 นาที  
ไม่ต้องแก้ LIFF ID หรือ URL ใหม่

---

## ปัญหาที่พบบ่อย

| ปัญหา | วิธีแก้ |
|---|---|
| `liff.init() failed` | ตรวจสอบ LIFF ID ใน config.js และ Endpoint URL ใน LINE Developers |
| ผู้ปกครองเข้าแล้วไม่เห็นข้อมูล | ต้องผูก LINE ก่อน — ติดต่อครูเพื่อส่งลิงก์ผูกบัญชี |
| ครูเช็คชื่อไม่ได้ (ไม่เห็นแท็บ) | ตรวจสอบว่า role เป็น `homeroom` และถูกกำหนดเป็นครูประจำชั้นในระบบ |
| แจ้งเตือนไม่ส่ง | ตรวจสอบ LINE Channel Access Token ในเมนูตั้งค่า |
| CORS error | ตรวจสอบว่า GAS_API_URL ถูกต้องและ Web App deploy แล้ว |
