# โจทย์ฝึก — ใช้ logs/auth.log

ทำตามลำดับ ห้ามเปิดเฉลยจนกว่าจะลองเองก่อนอย่างน้อย 5 นาทีต่อข้อ

## Day 2 — ทำตาม tutorial ลอกตาม 1 รอบ

ลองพิมพ์คำสั่งเหล่านี้ทีละคำสั่ง สังเกตว่า output เปลี่ยนไปอย่างไรเมื่อเพิ่ม flag/pipe:

```bash
cat logs/auth.log                        # ดูทั้งไฟล์
wc -l logs/auth.log                      # นับจำนวนบรรทัด
grep "Failed" logs/auth.log              # กรองเฉพาะบรรทัดที่มีคำว่า Failed
grep "Failed" logs/auth.log | wc -l      # นับว่ามีกี่บรรทัด
```

## Day 3 — ทำเองโดยปิด tutorial (อย่างน้อย 3 ข้อ)

**ข้อ 1: หา IP ที่พยายาม login ล้มเหลวมากที่สุด**
ใบ้: ต้อง extract คอลัมน์ IP ออกมาจากแต่ละบรรทัดด้วย `awk`, แล้ว `sort` + `uniq -c` เพื่อนับจำนวนครั้ง
เป้าหมาย: ได้ผลลัพธ์เป็นรายการ IP เรียงจากมากไปน้อยว่า IP ไหน brute-force หนักสุด

**ข้อ 2: หา username ที่ถูกลองมากที่สุดในการโจมตี**
ใบ้: บรรทัด `Failed password for <user> from <ip>` — ต้องดึงคำที่อยู่หลัง "for" ออกมา
ลองใช้ `awk '{print $9}'` แล้วดูว่า column ไหนตรงกับ username จริง (นับ column ให้ดี บาง
บรรทัดมีคำว่า "invalid user" แทรกเข้ามา ทำให้ column เลื่อน — นี่คือ error ที่ควรเจอและบันทึกไว้)

**ข้อ 3: แยกบรรทัด login สำเร็จ (Accepted) ออกมาเป็นไฟล์ใหม่**
ใบ้: `grep "Accepted" logs/auth.log > successful_logins.txt`
ลองเปิดไฟล์ที่ได้ดูว่า user ไหน login สำเร็จจาก IP ไหนบ้าง

**ข้อ 4 (โบนัส): ใช้ sed แทนที่ IP ทั้งหมดด้วยคำว่า `[REDACTED]`**
ใบ้: `sed 's/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/[REDACTED]/g'`
ลองสังเกตว่า regex pattern นี้ทำงานอย่างไร

### จงใจทำ error อย่างน้อย 3 แบบ (ตามที่โจทย์กำหนด)
ตัวอย่าง error ที่ควรเจอเองระหว่างทำโจทย์ข้างบน (บันทึกว่า error บอกอะไร + แก้อย่างไร):
- ลืมใส่ path ให้ถูก (`grep "Failed" auth.log` ตอนไม่ได้อยู่ในโฟลเดอร์ logs/)
- นับ column ของ awk ผิด (ได้ output เป็นคำว่า "from" แทนที่จะเป็น username จริง)
- ลืม escape จุด (`.`) ใน regex ตอนใช้ grep/sed หา IP ทำให้ match ผิดรูปแบบ

## Permission practice (สำหรับ Definition of Done ข้อ 2)

```bash
touch secret_config.txt
chmod 777 secret_config.txt      # ตั้งแบบเปิดกว้างเกินไป (ไม่ปลอดภัย)
ls -l secret_config.txt          # สังเกต permission string ที่เปลี่ยนไป
chmod 600 secret_config.txt      # แก้ให้เจ้าของอ่าน/เขียนได้คนเดียว (ปลอดภัยกว่า)
ls -l secret_config.txt
```
ลองอธิบายให้ตัวเองฟังว่าทำไม `777` ถึงอันตรายกว่า `600` สำหรับไฟล์ config ที่มีรหัสผ่านอยู่ข้างใน

## Process practice (สำหรับ Definition of Done ข้อ 3)

```bash
sleep 300 &        # รัน process ปลอมค้างไว้เบื้องหลัง (จำลอง process ผิดปกติ)
ps aux | grep sleep # หา process id (PID) ของมัน
kill <PID>          # สั่งปิด process ด้วย PID ที่เจอ
ps aux | grep sleep # เช็คว่าหายไปแล้วจริง
```

---

**เมื่อทำครบแล้ว** ให้บันทึกคำสั่งที่ใช้จริง + ผลลัพธ์ + error ที่เจอ ลงใน `day3/log.md`
เพื่อใช้เป็นหลักฐานการลงมือทำ (สกรีนช็อตด้วยก็ได้ตามที่โจทย์ขอ)
