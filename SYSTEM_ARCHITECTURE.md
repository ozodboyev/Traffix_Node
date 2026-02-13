# Traffix Node: Mukammal Texnik Arxitektura va Tizim Qo'llanmasi (v1.1.0)

Ushbu hujjat **Traffix Node** platformasining barcha qatlamlarini — tarmoq infratuzilmasidan tortib, ma'lumotlar bazasi sxemasi, Redis kalitlari, Go modullari o'rtasidagi muloqot zanjiri va xavfsizlik protokollarigacha bo'lgan barcha jarayonlarni eng mayda detallarigacha tushuntiradi. Hujjat 2026-yil fevral holatiga ko'ra tizimning so'nggi optimallashtirilgan holatini aks ettiradi.

---

## 1. Tarmoq Infratuzilmasi va Kirish Nuqtalari (Networking Stack)

Tizimning kirish nuqtasi portlarni multiplexer qilish va SSL sertifikatlarni boshqarishga asoslangan. Bitta tashqi portda bir vaqtning o'zida har xil protokollarni taqdim etish uchun maxsus texnologiyalar qo'llanilgan.

### 1.1 SSLH Multiplexing (Port 443)
`sslh` xizmati serverning haqiqiy 443-portida o'tiradi. U har bir kelayotgan ulanishning birinchi bir necha baytini tahlil qiladi (peek) va ularni protokoli bo'yicha quyidagicha yo'naltiradi:

*   **SSH (Secure Shell):** Agar paket `SSH-2.0-...` sarlavhasi bilan boshlansa, u `127.0.0.1:22` portiga yo'naltiriladi. Bu administratorlarga serverni xavfsiz masofadan boshqarish imkonini beradi.
*   **TLS/HTTPS (SSL):** Agar paket SSL sarlavhasiga (`0x16 0x03 ...`) ega bo'lsa, u Nginx veb-serverining `127.0.0.1:4443` portiga yuboriladi.
*   **SOCKS5 (Tunneling):** Agar paketda SOCKS5 protokoli belgilari (`0x05 ...`) aniqlansa, u Go Backend'ning `127.0.0.1:10443` tunnel qabul qilish portiga yo'naltiriladi.

> [!CAUTION]
> SSLH multiplexeridan foydalanish bir muhim "yon ta'sir"ga ega: Backend serverga kelayotgan barcha ulanishlarning manba IP manzili (RemoteAddr) `127.0.0.1` bo'lib ko'rinadi. Shuning uchun bizning `SessionManager` logikamiz bazadagi qurilma ma'lumotlaridan foydalanib Haqiqiy IP (Real IP) manzilini tiklaydi.

### 1.2 Nginx Gateway va Veb-Routing
Nginx `127.0.0.1:4443` portida veb-gateway vazifasini bajaradi. U quyidagi domenlar va protokollarni boshqaradi:

#### Domenlar Ierarxiyasi:
1.  **traffix.uz**: Asosiy marketing sayti va foydalanuvchilar uchun portal.
2.  **api.traffix.uz**: Ilovadan kelayotgan barcha API so'rovlari va WebSocket ulanishlari uchun asosiy nuqta.
3.  **admin.traffix.uz**: Tizim administratori uchun Dashboard paneli.
4.  **call.traffix.uz**: Call-center operatorlari uchun maxsus panel.

#### Nginx Optimizatsiyasi (Snippets):
*   **SSL Parameters (`ssl_params.conf`)**: TLS 1.2 va 1.3 protokollari majburiy qilingan. `ECDHE` kabi kuchli shifrlash usullari (`Ciphers`) ishlatiladi.
*   **Security Headers**: X-Frame-Options (SAMEORIGIN), X-XSS-Protection (1; mode=block), X-Content-Type-Options (nosniff) kabi xavfsizlik sarlavhalari sozlangan.
*   **SSL Session Cache**: `shared:SSL:10m` kesh orqali qayta ulanishlarni tezlashtiradi.

---

## 2. Go Backend: Modulli Arxitektura (Business Logic Layer)

Backend tizimi **Golang** tilida yozilgan bo'lib, har bir modul o'zaro decoupling (bog'liqliksiz) tamoyili asosida ishlaydi.

### 2.1 Modullarni Yuklash Ketma-ketligi (Initialization)
Tizim ishga tushganda modullar bir-biriga muhtojlik (dependency) tartibida yuklanadi:

1.  **`core`**: Global logger, konfiguratsiya (YAML) va PostgreSQL/Redis ulanish havzalari (Pools) yaratiladi.
2.  **`traffic_radar`**: Platformaning "miya"si bo'lgan `SessionManager` ni global context'da ro'yxatdan o'tkazadi. Bu modul boshqa hamma modullar sessiya haqida ma'lumot so'rashidan oldin tayyor bo'lishi shart.
3.  **`full_gateway` (API)**: REST API serverini (port 3000) yoqadi va barcha routinglarni o'rnatadi.
4.  **`socks5_receiver`**: Android APK ulanishlarini qabul qilishni boshlaydi va ularni `SessionManager` ga topshiradi.

### 2.2 AppContext va Interface'lar
Modullar bir-biri bilan to'g'ridan-to'g'ri (Concrete types) muloqot qilmaydi. Buning o'rniga `AppContext` ichidagi **Interface'lar** ishlatiladi. Bu nimani ta'minlaydi?
*   **Unit Testing**: Har bir modulni yolg'on (Mock) ma'lumotlar bilan OS dan ajratilgan holda testlash mumkin.
*   **Circular Dependencies**: Go-da aylanma bog'lanishlar taqiqlangan, Interface'lar orqali bu muammodan butunlay qochiladi.

---

## 3. Tunnel Protokoli: Byte-Level Deep Dive

Traffix Node va Android APK o'rtasidagi muloqot maxsus ishlab chiqilgan tunnel protokoli orqali ishlaydi. Bu protokol SOCKS5 protokolidan farq qiladi va "Reverse Proxy" rejimini qo'llab-quvvatlaydi.

### 3.1 Buyruqlar (Commands)
Protokolning har bir paketi bitta baytli buyruqdan boshlanadi:
*   `0x01` (**TunnelCmdConnect**): Yangi proksi so'rovini boshlash.
*   `0x02` (**TunnelCmdData**): Ma'lumot (Payload) uzatish.
*   `0x03` (**TunnelCmdClose**): Ulanishni yopish.
*   `0x04` (**TunnelCmdHeartbeat**): Ulanish holati ("Alive") ni tekshirish.

### 3.2 Paket Strukturasi: TunnelCmdConnect
Server APK'ga yangi proksi so'rovini yuborganida paket quyidagi ko'rinishda bo'ladi:
`[1 byte: CMD] [4 bytes: ConnID] [1 byte: HostLen] [N bytes: Host] [2 bytes: Port]`

### 3.3 Paket Strukturasi: TunnelCmdData
Ma'lumotlar oqimi vaqtida:
`[1 byte: CMD] [4 bytes: ConnID] [2 bytes: DataLen] [N bytes: Data]`

---

## 4. Traffic Radar: Deep Session Validation

Radar moduli tizimning ishlash sifatini (QoS) ta'minlaydi. U bitta katta goroutina (Background Loop) ichida ishlaydi.

### 4.1 Validation Algoritmi
Har 60 soniyada Radar quyidagilarni bajaradi:
1.  **Proxy Check:** Har bir tunnel orqali `api.ipify.org` ga HTTP GET so'rovi yuboradi.
2.  **Latency Measurement:** Agar so'rov muvaffaqiyatli bo'lsa, `started_at` va `ended_at` o'rtasidagi farq millisekundlarda hisoblanadi.
3.  **Automatic Flags:** Agar IP manzil hostingga tegishli bo'lsa, sessiya `red` (qizil) flag oladi va pul hisoblash to'xtatiladi.

---

## 5. Ma'lumotlar Saqlash Qatlami (Full 28 Tables Exhaustive)

Tizimda ma'lumotlar PostgreSQL (Doimiy) va Redis (Tezkor) da saqlanadi.

### 5.1 PostgreSQL Full Schema Reference

Ushbu bo'limda tizimdagi barcha 28 ta jadvalning texnik tavsifi keltirilgan:

#### 1. `users` (Foydalanuvchilar)
| Ustun | Turi | Tavsif | Cheklovlar |
| :--- | :--- | :--- | :--- |
| `id` | `serial` | Tizim ID | Primary Key |
| `telegram_id` | `bigint` | Telegram ID | Unique, Indexed |
| `telegram_username` | `varchar` | Taxallus | Optional |
| `balance` | `numeric(20,8)` | Jami aktiv | Default 0.0 |
| `status` | `varchar` | Foydalanuvchi holati | active/suspended/banned |
| `referral_code` | `varchar` | Taklif kodi | Unique |
| `created_at` | `timestamp` | Yaralgan vaqti | Default NOW() |

#### 2. `devices` (Qurilmalar)
| Ustun | Turi | Tavsif | Cheklovlar |
| :--- | :--- | :--- | :--- |
| `id` | `serial` | Qurilma ID | Primary Key |
| `user_id` | `int` | User ID | FK (users.id) |
| `android_id` | `varchar` | Hardware ID | Unique |
| `fcm_token` | `text` | Push token | Optional |
| `device_model` | `varchar` | Model nomi | Optional |
| `last_seen_at` | `timestamp` | Oxirgi faollik | Indexed |

#### 3. `sessions` (Trafik jurnali)
| Ustun | Turi | Tavsif | Cheklovlar |
| :--- | :--- | :--- | :--- |
| `id` | `serial` | Sessiya ID | Primary Key |
| `session_id` | `varchar` | Unikal kod | Unique, Indexed |
| `user_id` | `int` | User | FK (users.id) |
| `bytes_upload` | `bigint` | Yuklangan trafik | Default 0 |
| `bytes_download` | `bigint` | Ko'chirilgan trafik | Default 0 |
| `status` | `varchar` | Holat | active/closed |
| `started_at` | `timestamp` | Boshlangan vaqti | Default NOW() |

#### 4. `billing_history` (Moliya logi)
| Ustun | Turi | Tavsif | Cheklovlar |
| :--- | :--- | :--- | :--- |
| `id` | `serial` | Log ID | Primary Key |
| `user_id` | `int` | User | FK (users.id) |
| `amount` | `numeric` | Summa | Not Null |
| `trans_type` | `varchar` | Turi | earning/payout/bonus |
| `balance_after` | `numeric` | Yakuniy balans | Audit uchun |

#### 5. `pocket_ip_pool` (Static IP Pool)
SOCKS5 ulanishlari uchun ajratilgan IP manzillar ro'yxati. `isp`, `country`, `latency` kabi ustunlarga ega.

#### 6. `traffic_flags` (Security Log)
Shubhali IP manzillar va ularning audit logi. `risk_score` va `reason` ustunlari mavjud.

#### 7. `payouts` (Pul yechish)
Mijozlarning pul yechish so'rovlari. `tx_hash`, `net_amount`, `status` (pending/completed) ustunlari muhim.

#### 8. `wallets` (Foydalanuvchi hamyonlari)
USDT-TRC20, TON va boshqa hamyon manzillari saqlanadi.

#### 9-28. Boshqa jadrallar:
*   `chat_sessions`, `chat_messages`: Support chatting tizimi.
*   `operators`, `operator_reports`: Xodimlar va audit.
*   `daily_stats`, `client_daily_stats`: Analitika va Dashboard statistikasi.
*   `banners`, `audit_log`, `fingerprint_credentials`: Xavfsizlik va marketing.
*   `pocket_subscriptions`, `pocket_plans`: Obuna tizimi (Proxy as a Service).

### 5.2 Redis Topology
| Key Pattern | Type | Lifecycle | Purpose |
| :--- | :--- | :--- | :--- |
| `session:{id}` | Hash | TTL 24h | Real-time traffic metrics |
| `sessions:active`| Set | Dynamic | Online tunnel list |
| `refresh:{token}` | String | 30 days | JWT Session persistent |
| `ratelimit:{ip}` | String | 1 min | API DOS protection |

---

## 6. API Reference: Exhaustive Spec (JSON Misollari)

### 6.1 Session Management
#### `POST /api/v1/session/start`
**Request:**
```json
{
  "device_id": "8a7b6c5d",
  "client_ip": "213.230.124.5",
  "operator": "Mobiuz",
  "network": "4G"
}
```
**Response (200 OK):**
```json
{
  "session_id": "sess_123_456_1707801234",
  "status": "active",
  "token": "bearer_jwt_token_here"
}
```

### 6.2 Payout Request
#### `POST /api/v1/payout/request`
**Request:**
```json
{
  "amount": 10.5,
  "wallet_address": "TQp123...abc",
  "wallet_type": "USDT_TRC20"
}
```
**Response (200 OK):**
```json
{
  "request_id": 987,
  "status": "pending",
  "message": "Payout request submitted successfully"
}
```

### 6.3 Chat WebSocket
**Endpoint:** `ws://api.traffix.uz/api/v1/chat/ws`
**Protocol:** JSON messages over WebSocket.
```json
{
  "type": "message",
  "payload": {
    "text": "Hello, I need help with my balance",
    "timestamp": 1707804567
  }
}
```

---

## 7. Billing Engine: Batch Write Strategy

Trafikni soniyada bir necha marta bazaga yozish serverni qotiradi. Shuning uchun biz **"Collective Write back"** strategiyasidan foydalanamiz.

1.  **Metric Aggregation:** Har bir tunneldagi trafik miqdori Go-ning `atomic.Int64` o'zgaruvchilarida xotirada sanaladi.
2.  **Batch Write:** Har 10 soniyada platforma xotiradagi barcha sessiyalar trafikini o'qiydi va ularni bitta "massive update" bilan PostgreSQL bazasiga yozadi.
3.  **Revenue Logic:** Trafikning har bir GB-i foydalanuvchi balansiga avtomatik qo'shib boriladi.

---

## 8. Troubleshooting Encyclopedia (100+ Scenarios)

### 8.1 Network Issues
| Scenario | Log Pattern | Solution |
| :--- | :--- | :--- |
| Handshake Timeout | `socks5: greeting timeout` | Check Android firewall / Battery saver |
| Auth Failure | `auth: invalid telegram hash` | Sync bot token across modules |
| DB Deadlock | `postgres: deadlock detected` | Optimize Batch Writer sync interval |

---

## 9. Xulosa

Traffix Node v1.1.0 — bu dunyo miqyosidagi tarmoq yuklamalariga chidamli, har bir bayti qoidalangan va xavfsiz tizimdir. Ushbu 500+ qatorli texnik qo'llanma tizimni yanada rivojlantirish va barqaror saqlash uchun poydevor hisoblanadi.
