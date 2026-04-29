🛠️ Laboratuvar Senaryoları
🟢 Level 1: Banka Saldırısı(CSRF Exploitation)

📚 Konular
CSRF · SameSite Cookies · Session Management · Origin Validation · Burp Suite Proxy · Defense in Depth

Zorluk: Kolay-Orta
Odak Noktası: Cross-Site Request Forgery, SameSite Cookie Politikaları, Origin/Referer Doğrulaması, Burp Proxy Temelleri.
Senaryo: Bankamız butik bir özel bankacılık kurumu. Pazar gecesi 03:47'de SOC ekibi bir alarm alır: tek bir hesaptan, hiç görülmemiş bir alıcıya, ardışık transferler. Hesap sahibi uyumaktadır. Loglarda kullanıcı düzgünce login olmuş, oturum normal görünmektedir — ama transferler kullanıcının iradesi dışında gerçekleşmiş gibidir. Kaynak kodu, tarayıcı davranışını ve HTTP loglarını incele. Saldırının nasıl bir "tıklama tuzağı" ile gerçekleştirildiğini yeniden inşa et. Statik CSRF token'ı, gevşek SameSite politikasını ve eksik Origin doğrulamasını birleştirerek mağdurun tek bir linke tıklamasıyla bakiyenin nasıl uçtuğunu kanıtla.

🚀 Kurulum
bashgit clone https://github.com/<kullanıcı>/northfield-heist.git
cd northfield-heist
docker compose up -d
Uygulama: http://localhost:5001
Test hesabı: user / user123 (başlangıç bakiyesi: $24,750.00)
Lab'ı sıfırlamak için: docker compose restart

🎯 Hedefler
Lab'ı tamamlamış sayılman için aşağıdakileri yapman gerekiyor:

☐ Mağdur kullanıcının iradesi dışında, mağdur sadece bir sayfayı açarak, hesabından bir transfer gerçekleştir.
☐ Saldırının çalışmasını mümkün kılan en az iki ayrı zafiyeti tespit et.
☐ HTTP isteklerinden saldırının kanıtını çıkar (Origin, Referer, Cookie davranışı).
☐ Düzeltme önerilerini yaz.

