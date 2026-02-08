# 11. Nginx Konfiguratsiyasi (Mukammal Sozlamalar)

Ushbu hujjat **Traffix Node** loyihasining global tarmoq infratuzilmasi va Nginx serveri qanday sozlanganini ko'rsatadi. Tizim 4 ta asosiy server blokiga bo'lingan (Marketing, API, Admin, Operator).

## 🌍 Infratuzilma Xaritasi
*   **Marketing Site**: `traffix.uz` (Statik + API Proxy)
*   **API Backend**: `api.traffix.uz` (Asosiy xizmatlar + WebSocket)
*   **Admin Panel**: `admin.traffix.uz` (Boshqaruv markazi)
*   **Operator Portal**: `call.traffix.uz` (Mijozlar bilan aloqa)

## 🛠 Nginx Konfiguratsiyasi (`/etc/nginx/sites-enabled/traffix`)

```nginx
# 1. Marketing Site (traffix.uz)
server {
    listen 443 ssl http2;
    server_name traffix.uz www.traffix.uz;

    root /opt/Traffix_Node/frontend/marketing/providers;
    index index.html;

    # API Documentation (Deployed at traffix.uz/api/docs)
    location ^~ /api/docs {
        alias /opt/Traffix_Node/frontend/api-docs;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # API Proxy pass to Go Backend
    location /api/ {
        proxy_pass http://127.0.0.1:3000;
        include snippets/proxy_params.conf;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}

# 2. Main API Gateway (api.traffix.uz)
server {
    listen 443 ssl http2;
    server_name api.traffix.uz;

    # APK Downloads (Auto-latest version)
    location /api/v1/download/latest {
        root /opt/Traffix_Node/downloads;
        rewrite ^/api/v1/download/latest/?$ /latest.apk break;
    }

    # WebSocket Support for Chat
    location /api/v1/chat/ws {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    location / {
        proxy_pass http://127.0.0.1:3000;
        include snippets/proxy_params.conf;
    }
}
```

## 🔐 Xavfsizlik va SSL
*   Barcha domenlar **Let's Encrypt SSL (HSTS)** bilan himoyalangan.
*   HTTP so'rovlari avtomatik ravishda HTTPS ga yo'naltiriladi (Redirect 301).
*   WebSocket ulanishlari uchun maxsus `proxy_read_timeout 3600s` o'rnatilgan.

## 🚀 Optimallashtirish
*   `http2` yoqilgan (tezkor yuklanish uchun).
*   Statik fayllar (APK, Rasmlar, JS/CSS) uchun kesh boshqaruvi o'rnatilgan.
*   `^~` prefiksi orqali API Docs kabi muhim statik manzillar ustuvorligi (priority) ta'minlangan.
