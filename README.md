# 📍 راه‌اندازی ماژول GPS با آردوینو

## 🔍 معرفی
ماژول **GPS (Global Positioning System)** برای دریافت موقعیت جغرافیایی (طول، عرض، ارتفاع)، سرعت و زمان از ماهواره‌ها استفاده می‌شود. ماژول‌های رایج: **NEO-6M**, **NEO-8M**, **BN-880**.

## 📦 پیش‌نیازها
### سخت‌افزار:
- آردوینو (Uno/Nano/Mega)
- ماژول GPS (مثال: NEO-6M)
- آنتن GPS (اغلب داخلی)
- سیم‌های جامپر

### نرم‌افزار:
- کتابخانه **TinyGPS++** (توصیه می‌شود)
- یا کتابخانه **SoftwareSerial**

## 🔌 اتصالات ساده
```
ماژول NEO-6M → آردوینو Uno
VCC  → 3.3V یا 5V (بسته به ماژول)
GND  → GND
TX   → پین 4 (RX آردوینو)
RX   → پین 3 (TX آردوینو)
```

## 📡 نصب کتابخانه
در آردوینو IDE:
1. **Sketch → Include Library → Manage Libraries**
2. جستجوی **"TinyGPSPlus"**
3. نصب کتابخانه توسط **Mikal Hart**

## 🚀 کد پایه GPS
```arduino
#include <TinyGPS++.h>
#include <SoftwareSerial.h>

// تعریف پین‌های ارتباط نرم‌افزاری
SoftwareSerial gpsSerial(4, 3); // RX, TX (پین 4 به TX ماژول)
TinyGPSPlus gps;

void setup() {
  Serial.begin(9600);      // ارتباط با کامپیوتر
  gpsSerial.begin(9600);   // ارتباط با ماژول GPS
  
  Serial.println("GPS Receiver Ready");
  Serial.println("Waiting for satellite signal...");
}

void loop() {
  // دریافت و پردازش داده‌های GPS
  while (gpsSerial.available() > 0) {
    if (gps.encode(gpsSerial.read())) {
      displayGPSInfo();
    }
  }
  
  // اگر برای مدت طولانی سیگنالی نبود
  if (millis() > 5000 && gps.charsProcessed() < 10) {
    Serial.println("No GPS data received!");
    delay(2000);
  }
}

void displayGPSInfo() {
  Serial.println("\n--- GPS Data ---");
  
  // موقعیت جغرافیایی
  if (gps.location.isValid()) {
    Serial.print("Latitude: ");
    Serial.println(gps.location.lat(), 6);
    Serial.print("Longitude: ");
    Serial.println(gps.location.lng(), 6);
  } else {
    Serial.println("Location: Not available");
  }
  
  // تاریخ و زمان
  if (gps.date.isValid()) {
    Serial.print("Date: ");
    Serial.print(gps.date.year());
    Serial.print("/");
    Serial.print(gps.date.month());
    Serial.print("/");
    Serial.println(gps.date.day());
  }
  
  if (gps.time.isValid()) {
    Serial.print("Time: ");
    Serial.print(gps.time.hour() + 3); // تبدیل به ساعت ایران
    Serial.print(":");
    Serial.print(gps.time.minute());
    Serial.print(":");
    Serial.println(gps.time.second());
  }
  
  // تعداد ماهواره‌ها
  Serial.print("Satellites: ");
  Serial.println(gps.satellites.value());
  
  // ارتفاع
  if (gps.altitude.isValid()) {
    Serial.print("Altitude: ");
    Serial.print(gps.altitude.meters());
    Serial.println(" meters");
  }
  
  // سرعت
  if (gps.speed.isValid()) {
    Serial.print("Speed: ");
    Serial.print(gps.speed.kmph());
    Serial.println(" km/h");
  }
  
  Serial.println("----------------\n");
  delay(2000); // نمایش هر ۲ ثانیه
}
```

## 🎯 تست سریع
1. **اتصالات** را چک کنید
2. **کد** را آپلود کنید
3. **Serial Monitor** را باز کنید (9600 baud)
4. **ماژول** را نزدیک پنجره یا فضای باز قرار دهید
5. **منتظر دریافت ماهواره** باشید (ممکن است ۱-۵ دقیقه طول بکشد)

## 🛠️ عیب‌یابی
| مشکل | راه‌حل |
|------|--------|
| **هیچ داده‌ای دریافت نمی‌شود** | اتصال TX/RX را معکوس کنید |
| **موقعیت نامعتبر است** | ماژول را به فضای باز منتقل کنید |
| **"No GPS data"** | سرعت Serial را به 9600 تنظیم کنید |
| **فقط زمان دریافت می‌شود** | منتظر بمانید، موقعیت نیاز به ماهواره‌های بیشتر دارد |

## 💡 نکات کلیدی
- **اولین راه‌اندازی** ممکن است تا **۵ دقیقه** طول بکشد (جمع‌آوری Almanac)
- **فضای باز** = عملکرد بهتر
- **نزدیک پنجره** = حداقل نیاز
- **داخل ساختمان** = احتمالاً کار نمی‌کند
- **باتری پشتیبان** (در NEO-6M) برای راه‌اندازی سریع‌تر مفید است

## 📊 خروجی نمونه
```
GPS Receiver Ready
Waiting for satellite signal...

--- GPS Data ---
Latitude: 35.689198
Longitude: 51.388973
Date: 2024/2/22
Time: 14:30:22
Satellites: 8
Altitude: 1200.5 meters
Speed: 0.0 km/h
```

## 🔄 حالت‌های مختلف
```arduino
// فقط نمایش موقعیت
if (gps.location.isUpdated()) {
  Serial.print("LOC: ");
  Serial.print(gps.location.lat(), 6);
  Serial.print(",");
  Serial.println(gps.location.lng(), 6);
}

// نمایش مسیر حرکت
void trackMovement() {
  static double lastLat = 0, lastLng = 0;
  
  if (gps.location.isValid()) {
    float distance = gps.distanceBetween(
      lastLat, lastLng,
      gps.location.lat(), gps.location.lng()
    );
    
    if (distance > 10.0) { // اگر بیش از ۱۰ متر جابه‌جا شد
      Serial.print("Moved: ");
      Serial.print(distance);
      Serial.println(" meters");
      lastLat = gps.location.lat();
      lastLng = gps.location.lng();
    }
  }
}
```

## 🎮 پروژه‌های ایده‌آل
1. **ردیاب خودرو**
2. **ثبت‌کننده مسیر (Logger)**
3. **ساعت GPS**
4. **سرعت‌سنج**
5. **یافتن جهت (با دو GPS)**

## 📍 نکته مهم
دقت موقعیت‌یابی معمولاً **۲-۵ متر** است. برای دقت بیشتر از ماژول‌های **GPS-RTK** استفاده کنید.

**پیش به سوی ساخت پروژه‌های مبتنی بر موقعیت‌یابی!** 🚀
