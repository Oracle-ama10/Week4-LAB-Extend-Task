# 🚀 Project Summary: Team Chat Microservice
**รายวิชา:** ENGSE207 Software Architecture | **Assignment:** Lab 4 Add-on (Week 4)

---

## 🎯 ภาพรวมของโปรเจกต์ (Executive Summary)
โปรเจกต์นี้เป็นการต่อยอดระบบ Task Board (Layered Architecture) จาก Lab 4 โดยทำการเพิ่มฟีเจอร์ **"Team Chat"** เข้าไปในระบบ เราตัดสินใจออกแบบสถาปัตยกรรมใหม่ให้อยู่ในรูปแบบ **Microservices** โดยแยกส่วนแชทออกมาเป็น `Chat Service` อย่างเด็ดขาด เพื่อให้รองรับการเชื่อมต่อแบบ Real-time จำนวนมหาศาลได้โดยไม่กระทบกับประสิทธิภาพของ Task Board เดิม

---

## 🛠️ Stack เทคโนโลยีหลัก (Technology & Architecture)

เราเลือกใช้สถาปัตยกรรมแบบ **Polyglot Persistence** (เลือกใช้ Database ให้เหมาะกับงาน) และเน้น Event-Driven Architecture ดังนี้:

* 🟢 **Core Runtime:** `Node.js` ร่วมกับ `Express.js` (สถาปัตยกรรม Non-blocking I/O ทนทานต่อ Concurrent Connections สูง)
* ⚡ **Real-time Engine:** `Socket.io` (จัดการ WebSocket, มีระบบ Room จัดการกลุ่มบอร์ดแชทได้ทันที)
* 🗄️ **Legacy Database:** `SQLite` (ฐานข้อมูลเดิมจาก Lab 4 สำหรับเก็บข้อมูล Task และ User เชิงสัมพันธ์)
* 🍃 **Chat Database:** `MongoDB` (NoSQL สำหรับเก็บประวัติข้อความ เน้น Write-throughput สูงและยืดหยุ่น)
* 🔄 **Message Broker / Cache:** `Redis Pub/Sub` (หัวใจสำคัญในการทำ Horizontal Scaling เพื่อกระจายข้อความข้าม Chat Service Instances)
* ☁️ **Object Storage:** `AWS S3` (แบ่งเบาภาระ Database ด้วยการแยกเก็บไฟล์รูปภาพ/เอกสารแนบ)

---

## 📦 โครงสร้างการส่งงาน (Repository Structure)

```
week4_homework_Group[ใส่เลขกลุ่มของคุณ]/
 ┣ 📜 README.md                 📍 สรุปภาพรวมของโปรเจกต์ (คุณอยู่ที่นี่)
 ┣ 🖼️ architecture_diagram.png  📍 แผนภาพสถาปัตยกรรมระบบโดยรวม (C4/Container Level)
 ┣ 🖼️ event_flow_diagram.png    📍 แผนภาพ Sequence แสดงการทำงานของ Events
 ┣ 📄 service_design.pdf        📍 เอกสารตอบคำถามข้อ 1-3 และแนวทางแก้ปัญหา (Challenges) ข้อ 5
 ┗ 📄 api_design.yaml           📍 เอกสาร API Specification (OpenAPI 3.0) ครอบคลุม REST และ WS
```


## 🗂️ รูปแบบการจัดเก็บข้อมูล (Data Models Synergy)
จุดเด่นของดีไซน์กลุ่มเราคือการทำงานร่วมกันระหว่าง Database 2 ชนิด:

1. ฐานข้อมูลเดิม: SQLite (Task Service)
ทำหน้าที่เก็บข้อมูลเชิงสัมพันธ์ที่มีโครงสร้างตายตัว (Relational)

```
-- อ้างอิงจาก tasks.db ของ Lab 4
CREATE TABLE boards ( id INTEGER PRIMARY KEY, name TEXT );
CREATE TABLE users ( id INTEGER PRIMARY KEY, username TEXT );
```

2. ฐานข้อมูลใหม่: MongoDB (Chat Service)
ทำหน้าที่เก็บข้อความที่เน้นความเร็วและปริมาณมหาศาล โดยเก็บ boardId และ senderId เป็น Foreign Key โยงกลับไปที่ SQLite
```
// Collection: "messages"
{
  "_id": "60d5ec49f1b2c8b1f8e4e1a1",
  "boardId": 101,          // 🔗 Link to SQLite Board ID
  "senderId": 42,          // 🔗 Link to SQLite User ID
  "type": "text",          // หรือ "file"
  "content": "สวัสดีทีม!",    // หรือ "[https://s3.aws.com/](https://s3.aws.com/)..."
  "createdAt": "2026-03-23T10:00:00Z"
}
```

🛡️ กลไกการแก้ปัญหาที่สำคัญ (Key Solutions)
Message Order: ใช้ Timestamp และ Sequence Number จากฝั่ง Server ป้องกันปัญหาเวลาเครื่อง Client ไม่ตรงกัน

Delivery Guarantee: ใช้ระบบ Acknowledge (ACK) ร่วมกับ Idempotency Key ป้องกันข้อความเบิ้ลเวลาเน็ตหลุด

Chat History Tiering: เก็บข้อมูลใหม่ใน MongoDB และย้ายข้อมูลเก่า (6 เดือน+) ไปอัดไฟล์ Zip ลง AWS S3 เพื่อประหยัดงบและรักษาความเร็ว Database


จัดทำโดย: กลุ่ม [5] | รายวิชา ENGSE207 | มหาวิทยาลัยเทคโนโลยีราชมงคลล้านนา | พ.ศ. 2568
