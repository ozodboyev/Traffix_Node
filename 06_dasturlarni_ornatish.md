# 06. Server Dasturiy Ta'minoti va Versiyalar (Full Stack Setup)

Traffix Node loyihasi yuqori texnologik va ko'p komponentli tizim bo'lgani uchun, server muhiti (OS) juda diqqat bilan sozlanishi shart. Quyida barcha kerakli komponentlar va ularning optimal versiyalari keltirilgan.

## 🖥️ Operatsion Tizim (OS)
*   **Tavsiya etilgan**: Ubuntu 22.04 LTS (Jammy Jellyfish).
*   **CPU**: Kamida 2 ta yadro (4 va undan ko'p tavsiya etiladi).
*   **RAM**: Kamida 4 GB (Trafik foizi oshgan sari RAM talabi ko'payadi).
*   **Disk**: Kamida 20 GB SSD (Loglar va baza uchun).

---

## 🛠️ Dasturlar va Ularning Vazifalari

### 1. Go (Golang) — Versiya: 1.24+
Backend qismini kompilyatsiya qilish uchun asosiy til.
*   **Nega 1.24?**: Yangi versiyalarda Memory management (xotira boshqaruvi) yaxshilangan va TLS ulanishlari xavfsizroq.
*   **O'rnatish**: 
    ```bash
    wget https://go.dev/dl/go1.24.13.linux-amd64.tar.gz
    sudo tar -C /usr/local -xzf go1.24.13.linux-amd64.tar.gz
    export PATH=$PATH:/usr/local/go/bin
    ```

### 2. PostgreSQL — Versiya: 14.x yoki 15.x
Asosiy ma'lumotlar ombori.
*   **Vazifasi**: Foydalanuvchi ma'lumotlari, tranzaksiyalar va chat tarixini saqlaydi.
*   **Muhim sozlama**: `max_connections` parametrini kamida `200` qilib sozlash kerak, chunki bir vaqtning o'zida ko'plab mijozlar so'rov yuborishi mumkin.

### 3. Redis Server — Versiya: 6.0 yoki 7.x
Tezkor kesh (In-memory database).
*   **Vazifasi**: Faol proksi sessiyalarini va oxirgi 5 daqiqadagi jonli statistikani saqlash. Bu PostgreSQL'ga tushadigan og'irlikni 80% ga kamaytiradi.

### 4. Nginx — Versiya: 1.18.0 (Ubuntu Default)
Veb-server va Reverse Proxy.
*   **Vazifasi**: 80 va 443 portlarni boshqaradi. Admin va Operator panellarini dunyoga ochib beradi.
*   **Eng muhim vazifasi**: SSL (HTTPS) sertifikatlarini boshqarish.

### 5. HashiCorp Vault — Versiya: 1.12+ (LTS)
Maxfiy kalitlar menejeri.
*   **Vazifasi**: Tizimning "seyfi". Barcha parollar va API kalitlar shu yerda shifrlangan holda turadi.

### 6. Systemd (Native Linux Service)
Tizimni fonda (Background) boshqarish:
*   `traffix-node.service` — Barcha jarayonlarni kontrol qiladi.
*   Server restart bo'lganda dastur avtomatik ishga tushishini ta'minlaydi.

---

## 📡 Tarmoq va Firewall Sozlamalari (Security Group)
Cloud provayderda (masalan, AWS, Google Cloud, DigitalOcean) quyidagi portlarni ochiq qoldirish shart:

| Port | Protokol | Vazifasi | Izoh |
| :--- | :--- | :--- | :--- |
| **80** | TCP/HTTP | Public Website | Nginx boshqaradi |
| **443** | TCP/HTTPS | Admin & API | Nginx (SSL) boshqaradi |
| **1080** | TCP/SOCKS5 | Proxy Network | **Eng muhim proksi porti** |
| **8888** | TCP/HTTP | HTTP Proxy | Alternativ proksi porti |
| **22** | TCP/SSH | Management | Faqat sizning IP uchun! |
| **5432** | TCP/DB | PostgreSQL | Tashqi dunyo uchun yopiq bo'lishi shart |

## 📦 Qo'shimcha Loyiha Kutubxonalari (Go Packages)
*   `github.com/jackc/pgx/v4`: Bazaga ulanish uchun eng tezkor drayver.
*   `github.com/gorilla/websocket`: Chat va Real-time stats uchun.
*   `github.com/robfig/cron/v3`: Har kunlik hisobotlar va avtomatik to'lovlar uchun "taymer".

## ✅ Yakuniy Tekshiruv Buyrug'i:
Dasturlarni o'rnatib bo'lgach, ularning holatini ushbu buyruq bilan tekshiring:
`systemctl status nginx postgresql redis-server vault`
Barchasi "active (running)" yozuvini ko'rsatishi shart.
