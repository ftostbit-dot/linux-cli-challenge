# ติดตั้ง WSL (Windows Subsystem for Linux) บน Legion 5

ใช้เวลาประมาณ 10-15 นาที (รวมเวลาดาวน์โหลด) — ทำครั้งเดียวใช้ได้ตลอด

## ขั้นตอน

1. **เปิด PowerShell แบบ Administrator**
   กด `Win` พิมพ์ "PowerShell" → คลิกขวา → "Run as administrator"

2. **รันคำสั่งติดตั้ง WSL**
   ```powershell
   wsl --install
   ```
   คำสั่งนี้จะติดตั้ง WSL2 พร้อม Ubuntu (Linux distro ที่ใช้กันมากที่สุด) ให้อัตโนมัติ

3. **รีสตาร์ทเครื่อง** เมื่อติดตั้งเสร็จ (จะมีข้อความแจ้งให้ restart)

4. **หลัง restart, Ubuntu จะเปิดขึ้นมาเองและถามให้ตั้ง**
   - Username (ตัวเล็กทั้งหมด ไม่ต้องมีช่องว่าง เช่น `front`)
   - Password (พิมพ์แล้วจะไม่เห็นตัวอักษรขึ้นบนจอ ปกติ ไม่ error)

5. **ทดสอบว่าใช้งานได้**
   ```bash
   ls
   pwd
   ```
   ถ้าเห็น prompt ประมาณ `front@LAPTOP:~$` แปลว่าสำเร็จแล้ว

## เปิดใช้งานครั้งต่อไป

พิมพ์ `wsl` ใน PowerShell หรือ Windows Terminal หรือค้นหา "Ubuntu" ใน Start Menu

## เข้าถึงไฟล์ Windows จาก WSL

ไฟล์ใน Windows (เช่น `C:\Users\front\Desktop`) จะอยู่ที่:
```bash
cd /mnt/c/Users/front/Desktop
```

## ถ้า `wsl --install` ไม่ทำงาน (Error / ไม่รู้จักคำสั่ง)

มักเกิดจาก Windows เวอร์ชันเก่าเกินไป หรือ virtualization ปิดอยู่ใน BIOS — ให้บอกผม
error message ที่ขึ้นมา จะช่วยแก้เป็นเคสไปครับ

## ทางเลือกอื่นถ้าไม่อยากลง WSL

- **Git Bash** (ติดมากับ Git for Windows) — ใช้คำสั่ง Linux พื้นฐานได้บางส่วน แต่ไม่ครบเท่า WSL
  (ไม่มี `ps`, `chmod` ทำงานไม่เหมือน Linux จริง) ไม่แนะนำสำหรับ challenge นี้
- **Online terminal เช่น replit.com หรือ katacoda** — ใช้ได้ทันทีไม่ต้องติดตั้ง แต่ไม่มี process
  จริงให้ฝึกจัดการ (ข้อ 3 ของ Definition of Done จะฝึกได้จำกัด)

**แนะนำ: ใช้ WSL** เพราะจะได้ Linux แท้ๆ ครบทุกคำสั่งที่ต้องใช้ตลอด 5 วัน
