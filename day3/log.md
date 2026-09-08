# Day 3 Log — Hands-on & Break Things

## โจทย์ที่ทำเอง (ปิด tutorial)

### ข้อ 1: หา IP ที่พยายาม login ล้มเหลวมากที่สุด

```bash
grep "Failed" logs/auth.log | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c
```

**ผลลัพธ์:**
```
     10 185.220.101.5
      8 45.155.204.19
      8 91.240.118.77
```

**สรุป:** IP `185.220.101.5` พยายาม login ล้มเหลวบ่อยที่สุด (10 ครั้ง) — เข้าข่าย brute-force attack

**หมายเหตุ:** ตอนแรกลองใช้ `awk '{print $11}'` ดึง IP ตามตำแหน่งคอลัมน์ แต่เจอปัญหา (ดู Error 2)
จึงเปลี่ยนมาใช้ `grep -oE` ดึงตาม pattern ตัวเลขแทน แม่นยำกว่า

---

### ข้อ 2: หา username ที่ถูกลอง login ผิดมากที่สุด

```bash
grep "Failed" logs/auth.log | grep -v "invalid" | awk '{print $9}' | sort | uniq -c
```

**ผลลัพธ์:**
```
     10 admin
      2 oracle
      2 postgres
      7 root
      2 test
```

**สรุป:** username `admin` ถูกลองมากที่สุด (10 ครั้ง) ตามด้วย `root` (7 ครั้ง) — เป็น pattern ทั่วไป
ของการโจมตี เพราะเป็นชื่อ user ที่แฮกเกอร์ลองก่อนเสมอ

**หมายเหตุ:** ต้องใช้ `grep -v "invalid"` กรองบรรทัดที่มีคำว่า "invalid user" ออกก่อน ไม่งั้น
`awk '{print $9}'` จะได้คำว่า "invalid" ปนมาแทน username จริง (ดู Error 2 ประกอบ)

---

### ข้อ 3: แยกบรรทัด login สำเร็จออกเป็นไฟล์ใหม่

```bash
grep "Accepted" logs/auth.log > successful_logins.txt
```

**ผลลัพธ์:** สร้างไฟล์ `successful_logins.txt` สำเร็จ มี 5 บรรทัด — user `front` login สำเร็จ 4 ครั้ง
(password) และ user `deploy` login สำเร็จ 1 ครั้ง (publickey)

---

## Permission Practice

```bash
touch secret_config.txt
ls -l secret_config.txt          # -rwxrwxrwx (บน /mnt/c/)
chmod 600 secret_config.txt      # ไม่มีผล เพราะยังอยู่บน /mnt/c/ (ดู Error 3)

cd ~
touch secret_config.txt
ls -l secret_config.txt          # -rw-r--r-- (ค่า default ของ Linux แท้)
chmod 600 secret_config.txt
ls -l secret_config.txt          # -rw------- (สำเร็จ)
```

**สรุป:** เปลี่ยน permission จาก `rw-r--r--` (คนอื่นอ่านได้) เป็น `rw-------` (เจ้าของคนเดียวเข้าถึงได้)
เหมาะกับไฟล์ config ที่มีข้อมูลลับ เพราะป้องกันไม่ให้ user อื่นในเครื่องเดียวกันอ่าน/แก้ไขได้

---

## Process Practice

```bash
sleep 300 &              # รัน process เบื้องหลัง จำลอง process แปลกปลอม → PID 977
ps aux | grep sleep      # หา process → เจอ PID 977
kill 977                 # สั่งปิด process
ps aux | grep sleep      # เช็คซ้ำ → process หายไปแล้ว (Terminated)
```

**สรุป:** ใช้ `ps aux` ค้นหา process ที่รันอยู่ และ `kill <PID>` เพื่อหยุด process ที่ต้องสงสัยได้สำเร็จ

---

## Error ที่เจอ (3 แบบ)

### Error 1: พิมพ์ `wc -1` แทน `wc -l`
- **คำสั่งที่พิมพ์:** `grep "Failed" logs/auth.log | wc -1`
- **Error ที่ได้:** `error: unexpected argument '-1' found`
- **สาเหตุ:** พิมพ์เลข 1 แทนตัวอักษร L พิมพ์เล็ก (หน้าตาคล้ายกันมากในบางฟอนต์)
- **วิธีแก้:** พิมพ์ใหม่เป็น `wc -l` → ได้ผลลัพธ์ถูกต้อง (26 บรรทัด)

### Error 2: `awk` นับคอลัมน์ผิดเพราะ log บางบรรทัดมีจำนวนคำไม่เท่ากัน
- **คำสั่งที่พิมพ์:** `grep "Failed" logs/auth.log | awk '{print $11}'`
- **ผลลัพธ์ที่ผิดพลาด:** บางบรรทัดได้ค่าเป็น `git`, `ubuntu` (username) แทนที่จะเป็น IP
- **สาเหตุ:** บรรทัดที่มีคำว่า "invalid user" แทรกเข้ามา (เช่น `Failed password for invalid user git from ...`)
  ทำให้จำนวนคำในบรรทัดไม่เท่ากับบรรทัดปกติ คอลัมน์ที่ 11 เลยไม่ตรงกับ IP เสมอไป
- **วิธีแก้:** เปลี่ยนไปใช้ `grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'` ดึงตาม pattern ของ IP โดยตรง
  แทนการนับตำแหน่งคอลัมน์ ซึ่งแม่นยำกว่าเมื่อรูปแบบบรรทัดไม่สม่ำเสมอ
- **บทเรียน:** awk นับคอลัมน์แบบตายตัว ไม่ยืดหยุ่นกับ log ที่มีรูปแบบไม่สม่ำเสมอ ควรใช้ regex
  ดึงตาม pattern แทนเมื่อข้อมูลต้นทางไม่คงที่

### Error 3: `chmod` ไม่มีผลกับไฟล์ใน `/mnt/c/`
- **คำสั่งที่พิมพ์:** `chmod 600 secret_config.txt` (ตอนไฟล์อยู่ใน `/mnt/c/Users/.../linux-cli-challenge`)
- **ผลลัพธ์ที่คาด:** permission เปลี่ยนเป็น `-rw-------`
- **ผลลัพธ์จริง:** permission ยังเป็น `-rwxrwxrwx` เหมือนเดิม ไม่เปลี่ยนแปลงเลย
- **สาเหตุ:** ไฟล์อยู่บน `/mnt/c/` ซึ่งเป็น Windows NTFS filesystem ที่ WSL แค่เชื่อมเข้ามาให้ใช้งาน
  (DrvFs) — NTFS ไม่มีระบบ permission แบบ Linux จริง WSL เลยแสดง permission เป็น `rwxrwxrwx`
  เสมอ และคำสั่ง `chmod` จึงไม่มีผลใดๆ
- **วิธีแก้:** ย้ายไปทดสอบที่ home directory ของ Linux แท้ (`cd ~`) แทน ซึ่ง `chmod` ทำงานได้ปกติ
- **บทเรียน:** ต้องรู้ว่าไฟล์กำลังทำงานอยู่บน filesystem แบบไหน เพราะ permission ของ Linux
  ใช้ได้จริงเฉพาะกับ Linux filesystem เท่านั้น ไม่ใช่ไฟล์ที่ยืมมาจาก Windows

---

## Concept Map v2 — อัปเดตความเข้าใจ

เทียบกับ Concept Map วันที่ 2 วันนี้เข้าใจเพิ่มขึ้นในแต่ละหมวด:

```
Linux CLI
├── Text Processing
│   ├── grep "คำ" ไฟล์         → กรองบรรทัดที่มีคำนั้น (case-sensitive!)
│   ├── grep -v "คำ"           → กรองบรรทัดที่ "ไม่มี" คำนั้น (ใหม่วันนี้)
│   ├── grep -oE 'regex'       → ดึงเฉพาะส่วนที่ตรง pattern เช่น IP (ใหม่วันนี้)
│   ├── awk '{print $N}'       → ดึงคอลัมน์ที่ N (ต้องระวังถ้าจำนวนคำในบรรทัดไม่เท่ากัน!)
│   ├── sort | uniq -c         → นับจำนวนที่ซ้ำกัน (ต้อง sort ก่อนเสมอ)
│   └── > ไฟล์                 → เขียนผลลัพธ์ลงไฟล์แทนแสดงบนจอ (ใหม่วันนี้)
├── Permission
│   ├── ls -l                  → ดู permission string เช่น rw-r--r--
│   ├── chmod 600               → เจ้าของอ่าน/เขียนได้คนเดียว (r=4,w=2,x=1)
│   └── ⚠️ ใช้ได้เฉพาะ Linux filesystem จริง ไม่ใช่ /mnt/c/ (บทเรียนสำคัญวันนี้)
└── Process
    ├── คำสั่ง & (ต่อท้าย)      → รันเบื้องหลัง (background)
    ├── ps aux | grep คำ        → หา process ที่กำลังรัน + PID
    └── kill <PID>              → สั่งปิด process
```

**สิ่งที่เข้าใจผิดเมื่อวาน (Day 2) แล้ววันนี้แก้ไขได้:**
- คิดว่า `awk '{print $N}'` จะดึงคอลัมน์ถูกเสมอ → จริงๆ ต้องระวังบรรทัดที่มีจำนวนคำไม่เท่ากัน
- คิดว่า permission ของไฟล์ใน WSL เหมือนกันหมดไม่ว่าไฟล์อยู่ที่ไหน → จริงๆ ขึ้นกับว่าไฟล์อยู่บน
  Windows filesystem (`/mnt/c/`) หรือ Linux filesystem (`~`)




