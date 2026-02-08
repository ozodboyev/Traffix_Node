# 08. Build Qilish va Ishga Tushirish (Mukammal Qo'llanma)

Bu bo'lim loyihani "qutidan" (out of the box) chiqarib, to'liq ishchi holatga keltirish bo'yicha eng batafsil instruksiyadir.

## 🏗️ 1-Bosqich: Infratuzilmani Tayyorlash

### A. Ma'lumotlar Bazasini Sozlash (Hard Mode)
PostgreSQL ichida yangi ekotizim yarating:
```bash
sudo -u postgres psql
```
Bazadagi buyruqlar:
```sql
CREATE DATABASE traffix_node;
CREATE USER traffix WITH ENCRYPTED PASSWORD 'top_secret_password';
GRANT ALL PRIVILEGES ON DATABASE traffix_node TO traffix;
ALTER DATABASE traffix_node OWNER TO traffix;
\q
```

### B. Migratsiyalarni Ishlatish
Loyiha jadvallarini noldan qurish uchun master-sxemani import qiling:
```bash
sudo -u postgres psql -d traffix_node < /opt/Traffix_Node/backend/migrations/000_complete_schema.sql
```
*Eslatma: Keyinchalik qo'shilgan ustunlar bo'lsa, barcha `.sql` fayllarni tartib bilan (001, 002...) yurgizib chiqing.*

---

## 🏗️ 2-Bosqich: Backendni Build Qilish

Go loyihasini kompilyatsiya qilish:
1.  **Dependencies**: `go mod download`
2.  **Vendoring** (ixtiyoriy): `go mod vendor`
3.  **Compilation**:
    ```bash
    go build -ldflags="-s -w" -o server ./cmd/server/main.go
    ```
    *-ldflags="-s -w" kodi binar fayl hajmini kichraytiradi va ortiqcha debug ma'lumotlarni o'chirib chiqadi (prod uchun).*

---

## 🏗️ 3-Bosqich: Tizim Xizmatini (Systemd) Sozlash
Faylni yarating: `sudo nano /etc/systemd/system/traffix-node.service`
```ini
[Unit]
Description=Traffix Node Backend Service
After=network.target postgresql.service redis-server.service

[Service]
Type=simple
User=root
WorkingDirectory=/opt/Traffix_Node/backend
ExecStart=/opt/Traffix_Node/backend/server
Restart=always
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```
*LimitNOFILE=65535 — Juda muhim! Bu serverga bir vaqtning o’zida ko'p ulanishlarni (SOCKS5 so'rovlari) qabul qilishga imkon beradi.*

---

## 🏗️ 4-Bosqich: Nginx va SSL (The Interface)

### Nginx Config namunasi:
```nginx
server {
    listen 80;
    server_name api.traffix.uz;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name api.traffix.uz;

    ssl_certificate /etc/letsencrypt/live/api.traffix.uz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.traffix.uz/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

### SSL Sertifikat olish:
```bash
sudo certbot --nginx -d api.traffix.uz
```

---

## 🏗️ 5-Bosqich: Domenni Tekshirish va ishga tushirish
1.  **DNS**: Domen (`api.traffix.uz`) serveringizning IP manzili bilan bog'langanligini tekshiring (`ping api.traffix.uz`).
2.  **Reload**: `sudo systemctl daemon-reload`
3.  **Start**: `sudo systemctl restart traffix-node`

## 🚀 Muvaffaqiyatli ishga tushganini qanday bilish mumkin?
Brauzerda `https://api.traffix.uz/api/v1/health` manzilini oching. Agar javobda `{"status":"ok"}` yozuvini va server vaqtini ko'rsangiz — **Tizim foydalanishga to'liq tayyor!**
