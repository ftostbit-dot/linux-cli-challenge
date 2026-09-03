# Linux CLI for Security — 5-Day Challenge

**หัวขัอ:** Linux CLI — Log Analysis, Permission & Process Management เพื่อความปลอดภัยระบบ
**Challenge:** [5-Day Challenge] เรียนทักษะใหม่ด้วยตัวเองใน 5 วัน + สอนเพื่อน 5 นาที

## Definition of Done

1. เขียน pipeline คำสั่ง (`grep` / `awk` / `sed` / `sort` / `uniq`) หา IP ที่ login ล้มเหลวซ้ำๆ จาก log ได้เอง
2. ตั้งค่า permission (`chmod` / `chown`) ถูกต้องตามสถานการณ์ + อธิบายความเสี่ยงของ permission ที่ตั้งผิดได้
3. ใช้ `ps` / `top` / `htop` / `kill` หา process ที่ผิดปกติได้
4. สาธิตสดได้จริงภายใน 5 นาที

## โครงสร้างโฟลเดอร์

```
├── logs/               → sample log ไฟล์สำหรับฝึก (auth.log)
├── day2/                → concept map, confusion list, บันทึกวันที่ 2
├── day3/                → hands-on exercises, error log, บันทึกวันที่ 3
├── day4/                → Feynman note, สคริปต์อธิบาย, บันทึกวันที่ 4
├── day5/                → สไลด์, cheat sheet, บันทึกวันที่ 5
├── PRACTICE_TASKS.md    → โจทย์ฝึกที่ใช้ logs/auth.log
└── WSL_SETUP.md         → วิธีติดตั้ง Linux environment บน Windows
```

## เกี่ยวกับ log ไฟล์ตัวอย่าง

`logs/auth.log` เป็น log จำลองสไตล์ SSH authentication log (จำลองแบบไฟล์จริงที่ `/var/log/auth.log`
บน Linux server) มีทั้ง login สำเร็จของ user ปกติ (`front`, `deploy`) และการพยายาม brute-force
จาก IP หลายตัว — ใช้สำหรับฝึกคำสั่งใน `PRACTICE_TASKS.md`
