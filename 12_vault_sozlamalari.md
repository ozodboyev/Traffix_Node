# 12. HashiCorp Vault Sozlamalari (Secrets Management)

**Traffix Node** loyihasida barcha maxfiy kalitlar, parollar va tokenlar markazlashgan holda **HashiCorp Vault** tizimida saqlanadi. Bu xavfsizlikning eng yuqori darajasini ta'minlaydi.

## 🗝 Vault Server Konfiguratsiyasi (`vault/config.hcl`)

```hcl
storage "file" {
  path = "/opt/Traffix_Node/vault/data"
}

listener "tcp" {
  address     = "127.0.0.1:8200"
  tls_disable = true
}

api_addr = "http://127.0.0.1:8200"
ui = true
disable_mlock = true
```

## 🛠 Backend integratsiyasi (`backend/config.yaml`)

Backend ishga tushayotganda Vault bilan quyidagi parametrlar orqali bog'lanadi:

```yaml
secrets:
  source: "vault"
  vault_address: "http://127.0.0.1:8200"
  vault_token: "hvs.CAES..." # Root yoki AppRole Token
  vault_path: "traffix/backend" # Kalitlar saqlanadigan yo'l
```

## 📋 Vault ichida saqlanadigan kalitlar ro'yxati
Tizim quyidagi o'zgaruvchilarni avtomatik ravishda Vault'dan qidiradi:
1.  `DB_PASSWORD` - Ma'lumotlar bazasi paroli.
2.  `JWT_SECRET` - Foydalanuvchi sessiyalari uchun maxfiy kalit.
3.  `EMAIL_PASSWORD` - SMTP server paroli.
4.  `TELEGRAM_BOT_TOKEN` - Operator bot tokeni.
5.  `NOWPAYMENTS_API_KEY` - To'lov tizimi kaliti.
6.  `FIREBASE_CREDENTIALS` - Push-bildirishnomalar uchun JSON.

## 🚀 Vault bilan ishlash buyruqlari

### Vault'ni ochish (Unseal)
Vault serveri qayta ishga tushganda "muhrlangan" (sealed) holatda bo'ladi. Uni ochish uchun:
```bash
vault operator unseal <KEY_1>
vault operator unseal <KEY_2>
vault operator unseal <KEY_3>
```

### Kalitlarni o'qish/yozish
```bash
# Kalitlarni ko'rish
vault kv get traffix/backend

# Yangi kalit qo'shish
vault kv put traffix/backend DB_PASSWORD="new_password"
```

## 🛡 Xavfsizlik afzalliklari
*   **No Plaintext**: Kalitlar hech qachon kod ichida yoki oddiy fayllarda (`config.yaml`) ochiq holda saqlanmaydi.
*   **Auto-unlock**: Tizim `VAULT_TOKEN` orqali backend ishga tushganda avtomatik avtorizatsiyadan o'tadi.
*   **Isolation**: Har bir xizmat faqat o'ziga tegishli bo'lgan "path" (yo'l) dagi ma'lumotlarni o'qiy oladi.
