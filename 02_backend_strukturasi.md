# 02. Backend Arxitekturasi: Chuqur Texnik Tahlil (Deep Dive)

Backend tizimi **Clean Architecture** va **DDD (Domain-Driven Design)** tamoyillari asosida qurilgan. Bu tizimni oson skalallash va yangilash imkonini beradi.

## 🔄 Trafikning O'tish Jarayoni (Request Life-cycle)
Mijoz so'rov yuborganda quyidagi zanjir ishga tushadi:
1.  **Ingress**: Nginx so'rovni qabul qiladi va TLS (SSL) ni yechadi.
2.  **API Gateway**: So'rov qaysi panelga (Admin/Operator/Client) tegishli ekanligini aniqlaydi va autentifikatsiya (JWT) dan o'tkazadi.
3.  **Proxy Router**: Mijoz so'ragan mamlakat yoki shahar bo'yicha eng mos keladigan **Active Node** (Android qurilma) ni Proxy Pool'dan topadi.
4.  **Tunnel Manager**: Backend va Android qurilma o'rtasida shifrlangan TLS tunnelni ochadi.
5.  **Traffic Radar**: Har bir byte'ni o'lchaydi va har bir session uchun alohida log yuritadi.
6.  **Billing Engine**: Har 5-10 soniyada radar ma'lumotlarini bazaga yozadi va foydalanuvchi balansini yangilaydi.

## 📁 Komponentlar va Ularning Mas'uliyati

### 🛡️ `internal/security` - Xavfsizlik Qalqoni
*   **Challenge-Response Auth**: Foydalanuvchi ulanayotgan qurilmaning haqiqiyligini tekshiradi (Fingerprint verification).
*   **OTP Engine**: Admin kirishi uchun dinamik parollarni generatsiya qiladi va Gmail orqali yuboradi.

### 📡 `internal/modules/checker` - Aqlli Analizator
Bu modul sun'iy intellekt elementlaridan foydalanib:
*   **VPN Detector**: Nod'lar (provayderlar) VPN orqali bog'lanishining oldini oladi (faqat haqiqiy uy interneti bo'lishi shart).
*   **GeoIP Lookup**: MaxMind bazasi yordamida har bir IP'ning koordinatalari, provayderi (ASN) va sifatini (Proxy Score) aniqlaydi.

### 💾 `internal/repository` - Ma'lumotlar Boshqaruvi
*   Barcha SQL so'rovlari `pgx` kutubxonasi yordamida optimallashtirilgan.
*   **Transaction Handling**: Balans bilan bog'liq ishlarda ma'lumotlar yo'qolmasligi uchun `ACID` tranzaksiyalaridan foydalaniladi.

### 🔋 `internal/services/pocket_service.go`
Mobil ilovaga xizmat ko'rsatuvchi maxsus servis. U qurilmaning quvvatini (battery level) va tarmoq holatini (Wi-Fi/4G) nazorat qiladi.

## ⚡ Unumdorlik Ko'rsatkichlari (Benchmarks)
*   **Concurrent Connections**: 10,000+ bir vaqtdagi sessiyalar (bitta o'rtacha serverda).
*   **Memory Usage**: Go'ning samaradorligi evaziga minimal (8-15 MB RAM bo'sh holatda).
*   **Response Time**: API javobi o'rtacha < 50ms.

## 🛠️ Concurrency Patternlari
Loyiha Go'ning `Channels` va `WaitGroups` texnologiyalaridan unumli foydalanadi. Masalan, trafikni o'lchash jarayoni asosiy oqimga xalaqit bermasligi uchun alohida `Goroutine`larda (fonda) bajariladi.
