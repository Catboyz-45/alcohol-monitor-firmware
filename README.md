# Alcohol Monitor Firmware

Firmware สำหรับระบบตรวจวัดแอลกอฮอล์ด้วย ESP32 และเซนเซอร์ MQ-3 พัฒนาด้วย Arduino Framework และ PlatformIO เชื่อมต่อ NETPIE ผ่าน MQTT เพื่อส่งค่าการวัดและรับ Threshold จากแอป พร้อมแสดงผลบนจอ OLED และบันทึกข้อมูลไปยัง Firebase Realtime Database

โปรเจกต์นี้เป็นส่วนฝั่งอุปกรณ์ IoT ของระบบ Alcohol Monitor และพัฒนาขึ้นเพื่อการเรียนรู้ การทดลอง และนำเสนอผลงาน

## เกี่ยวกับโปรเจกต์

Alcohol Monitor Firmware อ่านค่าสัญญาณแอนะล็อกจากเซนเซอร์ MQ-3 ด้วย ESP32 จากนั้นเฉลี่ยตัวอย่างเพื่อลดสัญญาณรบกวน แปลงค่าเป็นสเกล 0–100 และเปรียบเทียบกับ Threshold สองระดับเพื่อควบคุมไฟแจ้งเตือน

อุปกรณ์ส่งค่า BAC จำลอง ค่าดิบ ระดับแจ้งเตือน และ Threshold ไปยัง NETPIE ผ่าน MQTT รวมถึงบันทึกค่าที่เกินเกณฑ์ขั้นต่ำไปยัง Firebase Realtime Database แอป Flutter ที่ใช้ดูข้อมูลและตั้งค่าอุปกรณ์แยกอยู่ใน [alcohol-monitor-app](https://github.com/Catboyz-45/alcohol-monitor-app)

## ความสามารถหลัก

- อ่านค่าแอนะล็อกจากเซนเซอร์ MQ-3 ผ่านขา ADC ของ ESP32
- เฉลี่ยค่าจาก 10 ตัวอย่างเพื่อลดสัญญาณรบกวน
- แปลงค่าดิบ ADC เป็นสเกลการแสดงผล 0–100
- แบ่งการแจ้งเตือนเป็น 3 ระดับตาม Threshold สองค่า
- ควบคุม LED สองดวงตามระดับแจ้งเตือน
- แสดงค่าล่าสุดและ Threshold บนจอ OLED SSD1306
- เชื่อมต่อ Wi-Fi และ NETPIE MQTT
- ส่งข้อมูลไปยัง NETPIE Message และ Device Shadow
- รับ Threshold ใหม่ในรูปแบบ JSON ผ่าน MQTT
- ตรวจสอบและเรียงค่าให้ `threshold1 <= threshold2`
- ส่งรายการวัดที่เกินค่าขั้นต่ำไปยัง Firebase Realtime Database
- เชื่อมต่อ MQTT ใหม่อัตโนมัติเมื่อการเชื่อมต่อหลุด
- แยก Wi-Fi, NETPIE credentials และ Firebase URL ออกจาก source code

## เทคโนโลยีและอุปกรณ์

| หมวด | เทคโนโลยี/อุปกรณ์ |
| --- | --- |
| Microcontroller | ESP32 DOIT DevKit V1 |
| Framework | Arduino Framework |
| Build system | PlatformIO |
| Sensor | MQ-3 Alcohol Sensor |
| Display | SSD1306 OLED 128×64, I2C |
| IoT platform | NETPIE |
| Messaging | MQTT, PubSubClient |
| Cloud storage | Firebase Realtime Database |
| Data format | JSON, ArduinoJson |

## การเชื่อมต่ออุปกรณ์

| อุปกรณ์ | ขา ESP32 |
| --- | --- |
| MQ-3 Analog Output | GPIO 34 |
| LED ระดับเตือน 1 | GPIO 4 |
| LED ระดับเตือน 2 | GPIO 19 |
| OLED SDA | GPIO 21 |
| OLED SCL | GPIO 22 |
| OLED I2C Address | `0x3C` |

ควรใช้ตัวต้านทานจำกัดกระแสกับ LED และตรวจสอบแรงดันขา Analog Output ของโมดูล MQ-3 ไม่ให้เกินช่วงอินพุตที่ ESP32 รองรับ

## MQTT Topics

| ทิศทาง | Topic | หน้าที่ |
| --- | --- | --- |
| Subscribe | `@msg/home/device_control` | รับค่า Threshold จากแอป |
| Publish | `@msg/home/device_status` | ส่งสถานะอุปกรณ์และผลวัด |
| Publish | `@msg/home/bac` | ส่งผลวัดสำหรับหน้าติดตาม |
| Publish | `@shadow/data/update` | อัปเดต NETPIE Device Shadow |

ตัวอย่างคำสั่งปรับ Threshold:

```json
{
  "threshold1": 20,
  "threshold2": 50
}
```

ตัวอย่างข้อมูลที่อุปกรณ์ส่ง:

```json
{
  "bac": 32.5,
  "raw": 1331,
  "threshold1": 20,
  "threshold2": 50,
  "alertLevel": 1,
  "millis": 120000
}
```

## โครงสร้างโปรเจกต์

```text
.
├── include/
│   └── secrets.example.h  # ตัวอย่างค่าการเชื่อมต่อ
├── src/
│   └── main.cpp           # การอ่านเซนเซอร์ แสดงผล และเชื่อมต่อ Cloud
├── .gitignore
├── platformio.ini         # Board, framework และ dependencies
└── README.md
```

## การติดตั้งและรัน

### สิ่งที่ต้องมี

- Visual Studio Code และ PlatformIO IDE หรือ PlatformIO Core
- บอร์ด ESP32 DOIT DevKit V1
- เซนเซอร์ MQ-3
- จอ OLED SSD1306 แบบ I2C
- NETPIE device credentials
- Firebase Realtime Database หากต้องการเก็บข้อมูลบน Cloud

### ขั้นตอน

```bash
git clone https://github.com/Catboyz-45/alcohol-monitor-firmware.git
cd alcohol-monitor-firmware
cp include/secrets.example.h include/secrets.h
```

แก้ไข `include/secrets.h` ด้วยค่าของ Wi-Fi, NETPIE และ Firebase ของตนเอง ไฟล์นี้ถูกระบุใน `.gitignore` และห้าม commit ขึ้น repository

Build firmware:

```bash
pio run
```

อัปโหลดลงบอร์ด:

```bash
pio run --target upload
```

เปิด Serial Monitor ที่ 115200 baud:

```bash
pio device monitor --baud 115200
```

## การตรวจสอบคุณภาพโค้ด

```bash
pio run
```

สถานะการตรวจล่าสุด:

- `pio run` — ผ่าน
- RAM usage — 14.5% (47,648 จาก 327,680 bytes)
- Flash usage — 73.6% (964,757 จาก 1,310,720 bytes)

## สิ่งที่ได้เรียนรู้

- อ่านค่า ADC และลดสัญญาณรบกวนด้วยการเฉลี่ยตัวอย่าง
- ออกแบบ Threshold หลายระดับและควบคุมเอาต์พุต
- แสดงผลบน OLED ผ่าน I2C
- ส่งและรับข้อมูล JSON ผ่าน MQTT
- ใช้ NETPIE Device Shadow สำหรับเก็บสถานะล่าสุด
- เชื่อมต่อ Firebase Realtime Database จาก ESP32
- จัดการการ reconnect ของ Wi-Fi และ MQTT
- แยก credentials ออกจาก source code

## สถานะและข้อจำกัด

โปรเจกต์นี้เป็นต้นแบบเพื่อการศึกษา ไม่ใช่อุปกรณ์ตรวจวัดแอลกอฮอล์ที่ผ่านการรับรองทางการแพทย์หรือกฎหมาย ค่า `bac` ในโค้ดเป็นการแปลงค่าดิบ ADC ไปยังสเกล 0–100 แบบเชิงเส้น ไม่ใช่ค่า BAC ทางกฎหมาย และต้องสอบเทียบกับอุปกรณ์มาตรฐานก่อนใช้อ้างอิง

การเชื่อมต่อ MQTT ปัจจุบันใช้พอร์ต 1883 และการเชื่อมต่อ Firebase ใช้ `setInsecure()` จึงยังไม่มีการตรวจสอบ TLS certificate อย่างสมบูรณ์ ก่อนใช้จริงควรเปลี่ยนเป็นการเชื่อมต่อ TLS ที่ตรวจสอบ certificate, กำหนด Firebase Security Rules, เพิ่ม timeout สำหรับการเชื่อมต่อ และย้ายงานสำคัญไปยัง trusted backend

## แนวทางพัฒนาต่อ

- เพิ่มสูตรสอบเทียบ MQ-3 จากข้อมูลการทดลองจริง
- ใช้ MQTT ผ่าน TLS พร้อมตรวจสอบ CA certificate
- ตรวจสอบ Firebase TLS certificate แทน `setInsecure()`
- เพิ่ม timeout และ backoff สำหรับ Wi-Fi/MQTT reconnect
- เพิ่ม acknowledgment หลังอัปเดต Threshold
- เพิ่ม watchdog และสถานะข้อผิดพลาดบน OLED
- เพิ่ม unit tests สำหรับการแปลงค่าและการจัดลำดับ Threshold
- เพิ่มการจัดเก็บค่าชั่วคราวเมื่ออินเทอร์เน็ตขาด
- เพิ่ม OTA firmware update ที่มีการตรวจสอบลายเซ็น

## ผู้พัฒนา

พัฒนาโดย [Catboyz-45](https://github.com/Catboyz-45) เพื่อใช้เป็นผลงานระหว่างเรียนและประกอบการสัมภาษณ์
