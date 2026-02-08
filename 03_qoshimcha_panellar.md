# 03. Panellar va Interfeyslar: Biznes Boshqaruvi Markazi

Barcha interfeyslar **Single Page Application (SPA)** tamoyili asosida yaratilgan bo'lib, ular foydalanuvchiga muammosiz va tezkor ish faoliyatini taqdim etadi.

## 1. 🛡️ Admin Boshqaruv Markazi (Control Center)
Ushbu panel loyihaning "miyasini" kuzatish uchun xizmat qiladi.
*   **Real-Time Dashboard**: Jonli grafikalar orqali tizimning umumiy salomatligi (Active proxies, Online users, Live bandwidth) ko'rsatiladi.
*   **Security Bridge**: Vault tizimini masofadan qulflash yoki ochish imkoniyati.
*   **System Logs**: Serverda sodir bo'layotgan har bir muhim hodisani (Admin login, Code generation, Error stack) online kuzatish.
*   **Financial Reports**: Kunlik, haftalik va oylik daromadlar hisoboti.

## 2. 🎧 Call Center va Support Portali (Operator Panel)
Mijozlar bilan aloqa sifatini oshirish uchun mo'ljallangan.
*   **Intelligent Routing**: Kelgan xabarni birinchi bo'lib javob bera oladigan operatorga yo'naltirish.
*   **Session Context**: Operator mijoz bilan gaplashayotganda uning qaysi portdan foydalanayotgani va qurilmasi ma'lumotlarini ko'rib turadi (Support osonlashishi uchun).
*   **Customer Rating**: Operatorning ishiga go'yo "Uber"dagidek baho beriladi. Bu tizim operatorlarni yanada xushmuomala va tezkor bo'lishga majbur qiladi.

## 3. 🌐 Marketing va Foydalanuvchi Sayti (Public Facing)
Ushbu bo'lim yangi foydalanuvchilarni tarmoqqa jalb qilish uchun optimallashtirilgan.
*   **Landing Page**: Zamonaviy dizayn (Dark Mode), 3D effektlar va ko’ngilochar navigatsiya bilan foydalanuvchini jalb qiladi.
*   **Help Center (Knowledge Base)**: Foydalanuvchi ilovani qanday o’rnatish, pul ishlash va balansni yechish bo'yicha barcha savollariga javob topadi.
*   **Trust Indicators**: Platformaning litsenziyalari, maxfiylik siyosati va foydalanish shartlari ochiq ko'rsatilgan.

## 📱 UI/UX Texnologiyalari
*   **Vanilla CSS**: Hech qanday og'ir kutubxonalarsiz (masalan, Tailwind'siz) yozilgan, bu esa sahifalarning millisekundlarda ochilishini ta'minlaydi.
*   **Inter Font Family**: Google fontlardan foydalanilgan bo'lib, barcha qurilmalarda bir xil chiroyli va o’qilishi oson ko'rinadi.
*   **Hash Routing**: Brauzer tarixini (Back/Forward) to'g'ri boshqaruvchi va URL orqali navigatsiya qiluvchi tizim.

## 🛡️ Ruxsatlar Tizimi (RBAC)
Tizimda rollar qat'iy ajratilgan:
*   **SuperAdmin**: To'liq huquqlar.
*   **Manager**: Foydalanuvchilarni ko'rish va hisobotlarni olish huquqi.
*   **Support/Operator**: Faqat chat va mijozlar bilan muloqot qilish huquqi.
