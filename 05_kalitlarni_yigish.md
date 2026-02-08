# 05. Kalitlarni Yig'ish va Xavfsizlikni Sozlash (Configuration Secrets)

Loyihani ishga tushirishdan oldin xavfsizlik arxitekturasini tushunish va barcha kalitlarni strategik nuqtalarga joylashtirish muhimdir. Tizim maxfiy ma'lumotlarni o'g'irlanishdan himoya qilish uchun bir necha qatlamli xavfsizlikka ega.

## 🗝️ Asosiy Kalitlar va Ularning Vazifalari

### 1. Sistema Kalitlari
*   **JWT_SECRET**: Foydalanuvchi va admin sessiyalarini imzolash (Sign) uchun ishlatiladi. Bu kalit murakkab bo'lishi shart (kamida 32-64 ta tasodifiy harf va sonlar). Agar bu kalit ochilsa, begona shaxslar sessiyalarni soxtalashtirishi mumkin.
*   **ADMIN_OTP_SECRET**: Admin paneli uchun bir martalik kodlarni generatsiya qilish algoritmi uchun ishlatiladi.

### 2. Tashqi API Kalitlari
*   **TELEGRAM_BOT_TOKEN**: `@BotFather` orqali olingan token. Bu orqali tizim adminlarga shubhali harakatlar haqida xabar yuboradi (masalan: "Serverga hujum bo'ldi" yoki "Yangi katta to'lov so'raldi").
*   **NOWPAYMENTS_API_KEY**: Foydalanuvchilarga pullarini USDT (TRC20) kabi kriptovalyutalarda to'lab berish uchun kripto-gateway kaliti. Uni `nowpayments.io` panelidan olasiz.
*   **NOWPAYMENTS_IPN_SECRET**: To'lovlar holati o'zgarganini (Approved/Success) uchinchi tomon tasdiqlashi uchun webhook kaliti.

### 3. Email (SMTP) Sozlamalari
Tizim Gmail, Zoho yoki boshqa korporativ pochta orqali ishlaydi:
*   **SMTP_USER**: Admin pochta manzili.
*   **SMTP_PASSWORD**: **Eng muhim nuqta!** Bu oddiy pochta paroli emas, balki "App Password" (Ilova paroli) bo'lishi shart.
*   **SMTP_HOST & PORT**: Gmail uchun (`smtp.gmail.com`, `587`), Zoho uchun (`smtp.zoho.com`, `587`).

## 🏢 Kalitlarni Saqlash Joylari (Hierarchy)

Tizim kalitlarni quyidagi ketma-ketlikda qidiradi (Priority Order):

### 🔼 1. Vault (Eng yuqori ustuvorlik)
HashiCorp Vault korporativ darajadagi xavfsizlikni ta'minlaydi:
*   Kalitlar server diskida shifrlangan holatda turadi.
*   Tizim ishga tushganda Vault "yoqilmaguncha" (Unseal qilinmaguncha) xizmatlar kalitlarni o'qiy olmaydi.
*   **Buyruq**: `vault kv put secret/traffix jwt_secret="Sizning_Kodingiz" ...`

### 🔼 2. Environment Variables
Operatsion tizim darajasidagi o'zgaruvchilar:
*   Odatda `/etc/systemd/system/traffix-node.service` fayali ichida yoki Linux po'stlog'ida (shell) saqlanadi.
*   Format: `TRAFFIX_DATABASE_PASSWORD`, `TRAFFIX_JWT_SECRET`.

### 🔼 3. `config.yaml` (Faqat sinovlar uchun)
Eng oson, lekin eng xavfli yo'l. Kalitlar to'g'ridan-to'g'ri config fayli ichida ochiq yoziladi.
*   **Sotuvdan oldin ushbu fayldagi barcha sirlar o'chirib chiqilishi yoki `VAULT_SECRET:` prefiksi bilan yozilishi shart.**

## 🧼 Xavfsizlik Gigenasi (Best Practices)
*   **Rotation**: Har 6 oyda barcha kalitlarni yangilab turish tavsiya etiladi.
*   **Isolation**: Dev (test) va Production (ishchi) serverlar uchun alohida kalitlar ishlating.
*   **Firebase Credentials**: `/opt/Traffix_Node/backend/firebase-key.json` fayli faqat serverda bo'lishi kerak. Bu fayl Android foydalanuvchilariga PUSH xabarnomalar yuborish huquqini beradi.

## ✅ Yakuniy qadam
Barcha kalitlarni yig'ib bo'lgach, ularni serverdagi `.env` yoki Vault tizimiga kiritish va `traffix-node` xizmatini restart qilish kerak. Shundan so'ng loglarda `[INFO] Secrets loaded successfully` yozuvi chiqishi lozim.
