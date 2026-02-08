# 04. Android Ilova (Traffix Node Mobile): Texnik va Strukturaviy Taxlil

**Traffix Node Mobile** — bu loyihaning "chekka qurilmalari" (Edge devices) bilan ishlovchi qismi bo'lib, har bir o'rnatilgan smartfonni global tarmoqning faol tuguniga aylantiradi. Ilova oddiy foydalanuvchi qurilmasini professional darajadagi proksi serverga aylantirish uchun murakkab xizmatlardan foydalanadi.

## 🏗️ Arxitektura va Texnik Stack
Ilova zamonaviy Android standartlari asosida qurilgan:
*   **Language**: Kotlin (Google tomonidan tavsiya etilgan eng tezkor til).
*   **Architecture**: MVVM (Model-View-ViewModel) — UI va mantiqni to'liq ajratish uchun.
*   **Asosiy kutubxonalar**:
    *   `Retrofit 2`: Backend API bilan RESTful muloqot qilish uchun.
    *   `OkHttp3`: Tarmoq so'rovlarini optimallashtirish va xavfsizlik.
    *   `Coroutines & Flow`: Parallel jarayonlarni boshqarish (masalan, trafikni o'lchash jarayoni foydalanuvchi interfeysini qotirib qo'ymaydi).
    *   `Jetpack Compose`: Zamonaviy va dinamik foydalanuvchi interfeysi (UI).
    *   `Koin/Dagger Hilt`: Dependency Injection (qaramliklarni boshqarish) uchun.

## ⚙️ Ishlash Tartibi (Core Logic)
Ilova faollashtirilganda quyidagi jarayonlar zanjiri ishga tushadi:

### 1. Tugunni Ro'yxatdan O'tkazish (Node Registration)
Ilova serverga ulanishdan oldin qurilmaning xususiyatlarini (Brand, Model, OS version, IP address) yuboradi. Server qurilmani autentifikatsiya qiladi va unga maxsus `NodeID` beradi.

### 2. SOCKS5 TLS Tunneling
Ilovaning eng muhim qismi — bu proksi so'rovlarini qabul qilish. U o'zida kichik bir "Local Proxy Client"ni ishlatadi. Bu mijoz serverdan kelayotgan buyruqlarni real vaqtda bajara boshlaydi:
*   Mijozlar so'rovi shifrlangan TLS tunnel orqali mobil qurilmaga keladi.
*   Qurilma o'zining internet tarmog'i (Wi-Fi yoki 4G) orqali so'rovni manzilga yuboradi.
*   Qabul qilingan ma'lumotni yana shifrlangan holda orqaga qaytaradi.

### 3. Fonda Ishlovchi Xizmat (Foreground Service)
Android tizimi foydalanuvchi ilovadan chiqqanda uni o'chirib yubormasligi uchun `Foreground Service`dan foydalaniladi:
*   **Sticky Notification**: Foydalanuvchi doimiy bildirishnoma orqali ilova ishlayotganini va qancha pul topayotganini ko'rib turadi.
*   **Wakelock Manager**: Qurilmani uxlab qolishdan (deep sleep) saqlaydi, lekin energiya sarfini minimal darajada ushlab turadi.

## 📊 Monitoring va Hisob-kitob (Telemetry)
Ilova serverga har 30 soniyada "Heartbeat" (yurak urishi) signallarini yuboradi. Bu signallarda:
*   Qurilmaning onlayn holati.
*   Batareya quvvati (Battery level).
*   Internet tezligi (Latency/Ping).
*   Qurilmaning harorati (CPU Temperature) — qurilma qizib ketishining oldini olish uchun.

## 🛡️ Ruxsatnomalar va Xavfsizlik
Ilova foydalanuvchi shaxsiy ma'lumotlariga (galereya, kontaktlar) teginmaydi, faqat tarmoq bilan bog'liq ruxsatlarni talab qiladi:
*   `ACCESS_WIFI_STATE`: Tarmoq turini aniqlash.
*   `POST_NOTIFICATIONS`: Android 13+ da xabarnomalar ko'rsatish.
*   `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`: Batareya cheklovlarini chetlab o'tish (Doze mode bypass).
*   `FOREGROUND_SERVICE_SPECIAL_USE`: Android 14 qoidalariga ko'ra proksi servisini ishlatish uchun.

## 💰 Monetizatsiya va Balans
Foydalanuvchi barcha moliyaviy amallarni to'g'ridan-to'g'ri ilovada boshqaradi:
*   **Real-time Balance**: Har bir 100 KB trafik uchun balans yangilanadi.
*   **Withdrawal History**: Pulni kripto hamyoniga chiqarganligi haqidagi tarixni ko'rish.
*   **Referral System**: Boshqalarni taklif qilib qo'shimcha bonuslar olish.

## 🏁 Xulosa
Android ilova — bu shunchaki interfeys emas, balki murakkab tarmoq protokollari va tizim xizmatlari kombinatsiyasidir. U foydalanuvchi uchun pul ishlash vositasi, tizim uchun esa tarmoqning eng chekka nuqtalaridagi "ishchi kuchi" bo'lib xizmat qiladi.
