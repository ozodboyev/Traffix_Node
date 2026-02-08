# 07. Tizimni Ishga Tushirishdan Oldin Parametrlarni Tekshirish

Server o'rnatilib, kod build qilingandan so'ng, tizimni jonli (live) holatga o'tkazishdan oldin oxirgi va o'ta muhim tekshiruvlar bosqichi keladi. Ushbu qadamda bitta kichik xato butun tizim bloklanishiga olib kelishi mumkin.

## 🔍 1. `config.yaml` Faylining Chuqur Tekshiruvi

### A. Database Bo'limi
*   `host`, `port`, `user`, `password` — bazaga ulanish ma'lumotlari haqiqatan ham ishlaydimi?
*   **Test buyrug'i**: `psql -h localhost -U traffix -d traffix_node` (Parolni so'rasa, demak hammasi joyida).

### B. Security Bo'limi
*   `jwt_secret` o'rnatilganmi? (Agar bo'sh bo'lsa, foydalanuvchilar login qila olmaydi).
*   `admin_email`: Bu yerga aynan sizning, kirish huquqiga ega bo'lgan pochtangiz yozilganmi?
*   **Rate Limit**: `rate_limit_requests` juda kichik emasmi? (Masalan, 5 ta qilib qo'yilgan bo'lsa, mijozlar bir zumda bloklanib qoladi. Optimal qiymat: 100+).

### C. Email Sozlamalari
*   `email.enabled` holati `true`mi?
*   **Challenge test**: Admin panelga kirib "Login" tugmasini bosing. 5 soniya ichida pochtangizga xat kelmasa, SMTP sozlamalarida yoki "App Password"da xato bor.

---

## 🔒 2. Vault va Maxfiy Ma'lumotlar Holati
*   Vault "Unseal" (ochilgan) holatdami?
*   `VAULT_TOKEN` o'zgaruvchisi tizimda bormi?
*   **Tekshiruv**: `vault kv get secret/traffix` buyrug'i natija beryaptimi?

---

## 📡 3. Tarmoq va Proksi Testi
*   **Port 1080**: Bu proksining asosiy eshigi. `netstat -tunlp | grep 1080` orqali u ishlayotganini ko'ring.
*   **SSL Sertifikati**: Nginx orqali bog'langan domen (masalan, `api.traffix.uz`) brauzerda "Xavfsiz" (Secure/Lock icon) ko'rinyaptimi?

---

## 📂 4. Fayl Tizimi va Ruxsatlar (Permissions)
Serverda loyiha fayllari uchun ruxsatlar noto'g'ri bo'lsa, dastur log yozolmaydi yoki fayl yuklolmaydi:
*   `uploads/` papkasi o'qish va yozish uchun ruxsatga egami (`755` yoki `777`)?
*   `main` (server) fayli ijro etuvchi (Executable) holatdami?
*   **Buyruq**: `chmod +x server`

---

## 🧪 5. Oxirgi "Stress Test" (Pre-flight check)
Tizimni qo'lda bir martaga ishga tushirib loglarni ko'ring:
```bash
./server
```
Logda quyidagi yozuvlar chiqishi shart:
1. `[INFO] Config loaded successfully`
2. `[INFO] Database connected`
3. `[INFO] Redis connected`
4. `[INFO] SOCKS5 Proxy started on port 1080`
5. `[INFO] HTTP Server started on port 3000`

**Agar yuqoridagilarning barchasi bajarilgan bo'lsa, tizim `systemd` orqali doimiy ishlashga tayyor!**
