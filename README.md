# MOBY-ROBOT

ชุดโค้ดและเอกสารประกอบสำหรับโปรเจกต์หุ่นยนต์ควบคุมด้วยจอย พร้อมระบบกล้องและหน้าเว็บสำหรับดูภาพสด ซูม และบันทึกภาพ โดยแบ่งการทำงานหลักระหว่าง ESP32-S3 กับ Raspberry Pi 5

## ภาพรวมการทำงาน

- **ESP32-S3** รับข้อมูลจากจอย Bluetooth ผ่าน Bluepad32 แล้วควบคุมมอเตอร์ซ้าย/ขวาและเซอร์โว pan ของกล้อง
- **Raspberry Pi 5** เชื่อมต่อกล้อง ประมวลผลภาพ และเปิดเว็บแอปสำหรับดูภาพสด ซูม และถ่ายภาพ ระบบสามารถต่อยอดให้บันทึกภาพไปยัง Google Drive และข้อมูลไปยัง Google Sheets ได้
- **การเชื่อมต่อระหว่าง ESP32 กับ Pi** มีโค้ดรองรับการสื่อสารผ่าน USB Serial หรือ TCP/Wi-Fi สำหรับคำสั่งควบคุมที่เกี่ยวข้อง ทั้งนี้การควบคุมมอเตอร์ทำงานบน ESP32

## หน้าที่ที่ได้รับมอบหมาย

ในงานนี้ **หน้าที่ที่ได้รับมอบหมายคือเขียนโค้ดฝั่ง Raspberry Pi และดูแลระบบกล้อง** ได้แก่ การเชื่อมต่อ/อ่านภาพจากกล้อง การแสดงภาพสดผ่านหน้าเว็บ การซูมและบันทึกภาพ รวมถึงส่วนเชื่อมต่อบริการจัดเก็บข้อมูลที่เกี่ยวข้อง ส่วนโค้ด ESP32 ทำหน้าที่รับจอยและควบคุมมอเตอร์กับเซอร์โว

## โครงสร้างไฟล์

```text
.
├── README.md
└── appendix_code/
    ├── README.txt
    ├── docs/
    │   ├── START_HERE_TH.md
    │   ├── CONTROLLER_SETUP.md
    │   └── README_CONTROLLER.md
    ├── esp32/
    │   ├── esp32_s3_servo_test.ino
    │   ├── esp32_gamepad_test.ino
    │   └── test.ino
    └── raspberry_pi/
        ├── app_realtime.py
        ├── meter_service.py
        ├── controller_server.py
        ├── serial_controller.py
        ├── authorize_google_drive.py
        ├── requirements.txt
        ├── meter-controller.service.example
        └── templates/
            └── index.html
```

## อธิบายแต่ละส่วน

### `appendix_code/esp32/` — ชุดควบคุมหุ่นยนต์

- **`esp32_s3_servo_test.ino`** — โค้ด ESP32-S3 หลักในชุดไฟล์นี้ ใช้ Bluepad32 รับจอย Bluetooth, ผสมค่าจอยเพื่อควบคุมมอเตอร์สองข้างผ่าน TB6612FNG และควบคุมมุมเซอร์โว pan
- **`esp32_gamepad_test.ino`** — โค้ดทดสอบการจับคู่จอยและอ่านค่าปุ่ม/แกนสติ๊ก แสดงสถานะผ่าน Serial Monitor โดยไม่สั่งมอเตอร์
- **`test.ino`** — สเก็ตช์ทดสอบขนาดเล็ก ใช้ตรวจสอบบอร์ดหรือการอัปโหลดเบื้องต้น

### `appendix_code/raspberry_pi/` — เว็บแอป กล้อง และบริการบน Pi

- **`app_realtime.py`** — เว็บแอป Flask สำหรับภาพสดจากกล้อง มีหน้าเว็บควบคุมซูมและถ่ายภาพ บันทึกภาพลงเครื่อง และทำงานส่งภาพไป Google Drive/เพิ่มข้อมูลลง Google Sheets ในเบื้องหลัง นอกจากนี้ยังเริ่มบริการรับคำสั่งจาก ESP32 ผ่าน TCP หรือ Serial ตามค่าตั้ง
- **`meter_service.py`** — ส่วนบริการกล้องและจัดการข้อมูลภาพ รองรับกล้อง USB ผ่าน OpenCV และกล้อง CSI ผ่าน Picamera2 มีฟังก์ชันซูม บันทึกภาพ, OCR อ่านตัวเลขมิเตอร์ด้วย Tesseract และเชื่อมต่อ Google Drive/Sheets
- **`controller_server.py`** — TCP server สำหรับรับคำสั่งจาก ESP32 แล้วส่งให้ตัวจัดการคำสั่งประมวลผล
- **`serial_controller.py`** — อ่านคำสั่งจาก ESP32 ผ่านพอร์ต USB Serial และส่งผลตอบกลับ
- **`authorize_google_drive.py`** — เครื่องมือสร้าง Google Drive OAuth token บนเครื่องที่เปิดเบราว์เซอร์ได้
- **`templates/index.html`** — หน้าเว็บตัวอย่างสำหรับดูภาพสด สั่งถ่ายภาพ และซูม
- **`requirements.txt`** — รายการแพ็กเกจ Python ที่ใช้ เช่น Flask, OpenCV, pytesseract, Google API client, dotenv และ pyserial
- **`meter-controller.service.example`** — ตัวอย่างไฟล์ systemd สำหรับตั้งให้บริการเริ่มทำงานอัตโนมัติบน Raspberry Pi

### `appendix_code/docs/` — คู่มือ

- **`START_HERE_TH.md`** — คู่มือติดตั้งและเริ่มใช้งานตั้งแต่เตรียมอุปกรณ์ ตั้งค่า Pi/กล้อง ไปจนถึงทดสอบระบบ
- **`CONTROLLER_SETUP.md`** — สรุปการตั้งค่าการเชื่อมต่อ ESP32-S3 กับ Raspberry Pi ทั้ง Wi-Fi และ USB Serial
- **`README_CONTROLLER.md`** — อธิบายหน้าที่ของ ESP32, Pi และรูปแบบการควบคุม รวมถึงแนวทางการต่ออุปกรณ์

## เทคโนโลยีหลัก

- **ไมโครคอนโทรลเลอร์:** ESP32-S3, Arduino, Bluepad32, ESP32Servo
- **คอมพิวเตอร์บอร์ดเดี่ยว:** Raspberry Pi 5, Python, Flask
- **กล้องและภาพ:** OpenCV; Picamera2 สำหรับกล้อง CSI (ตามการติดตั้ง); Tesseract OCR สำหรับอ่านตัวเลขเมื่อเปิดใช้งาน
- **การจัดเก็บข้อมูลเสริม:** Google Drive API และ Google Sheets API

## การเริ่มต้น

ดูคู่มือภาษาไทยที่ [`appendix_code/docs/START_HERE_TH.md`](appendix_code/docs/START_HERE_TH.md) ก่อนเริ่มติดตั้ง เพราะต้องเตรียมฮาร์ดแวร์ ไลบรารี และค่าตั้งของกล้อง/การเชื่อมต่อให้ตรงกับอุปกรณ์จริง

รายการ Python packages อยู่ที่ [`appendix_code/raspberry_pi/requirements.txt`](appendix_code/raspberry_pi/requirements.txt) ส่วนโค้ด Arduino ให้เปิดไฟล์ `.ino` ที่ต้องการด้วย Arduino IDE และเลือกบอร์ด ESP32 ให้ตรงรุ่น

> **ข้อควรทราบ:** โฟลเดอร์นี้เป็นชุดไฟล์ประกอบ/ภาคผนวก และมีโค้ดหลายส่วนสำหรับการทดสอบหรือการทำงานคนละหน้าที่ ควรตรวจคู่มือและตรวจชื่อไฟล์เริ่มต้นให้ตรงกับชุดที่นำไปติดตั้งจริงก่อนรัน อย่านำไฟล์ credentials, OAuth token, `.env` หรือรหัสผ่านขึ้น repository

## ความปลอดภัย

ก่อนทดสอบมอเตอร์และเซอร์โว ให้ยึดหุ่นยนต์ให้อยู่ในตำแหน่งปลอดภัยและใช้แหล่งจ่ายไฟที่เหมาะสม แยกไฟเลี้ยงมอเตอร์/เซอร์โวตามข้อกำหนดของอุปกรณ์ และตรวจสอบการต่อกราวด์ร่วมตามคู่มือ
