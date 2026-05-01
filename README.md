# 🎯 Banka Saldırısı


Bir CSRF (Cross-Site Request Forgery) zafiyet laboratuvarı. Burp Suite ile çözülmek üzere tasarlanmış, gerçekçi bir bankacılık arayüzü üzerinde çalışan, izole bir Docker ortamı.

---

## 📜 Senaryo

Bankamız butik bir özel bankacılık kurumu. Müşteri portföyünde devlet adamları, miras vakıfları, isimsiz bağışçılar var. Kurumun mottosu yıllardır aynı: *"Discretion is our currency."*

Ama o gece bir şey kırıldı.

Hafta sonu güvenlik nöbetinde olan SOC ekibi, sabah 03:47'de bir alarm aldı: tek bir hesaptan, yarım saat içinde, daha önce hiç işlem görmemiş bir alıcıya **art arda transferler**. Hesap sahibi uyuyordu. Telefonuna dokunmamıştı. Cihazından şikâyet gelmemişti. Loglarda kullanıcı düzgünce login olmuş, oturum normal görünüyor — ama transferler kullanıcının iradesi dışında gerçekleşmiş gibi.

Yönetim panik halinde. Senin elinde sadece şunlar var:

- Uygulamanın kaynak kodu
- Tarayıcının davranışı
- Sunucunun loglarına erişim
- Ve birkaç saatlik zaman

Saldırgan iz bırakmadığını sanıyor. Ama her saldırgan bir yerde hata yapar — ya tasarımda, ya isteğin kendisinde, ya da güvendiği bir varsayımda.

> **Görevin:** Saldırının nasıl gerçekleştirildiğini yeniden inşa et. Mağdurun tek bir tıklaması bu kadar parayı nasıl uçurabildi? Kanıtları topla, zafiyet zincirini çıkar.
>
> **Saldırgan kendini nerede ele verdi? Bul.**

---

## 🛠️ Laboratuvar Senaryoları

### 🟢 Level 1: Banka Saldırısı (CSRF Exploitation)

**📚 Konular:** `CSRF` · `SameSite Cookies` · `Session Management` · `Origin Validation` · `Burp Suite Proxy` · `Defense in Depth`

| Özellik | Değer |
|---------|-------|
| **Zorluk** | ⭐ Kolay|
| **Odak Noktası** | Cross-Site Request Forgery, SameSite Cookie Politikaları, Origin/Referer Doğrulaması, Burp Proxy Temelleri |
| **Süre** | 30–45 dakika |
| **Stack** | Flask 3.0 · Python · Docker |
| **Gerekli Araçlar** | Burp Suite (Community yeterli), Docker, modern bir tarayıcı |

**Senaryo:** Bankamız butik bir özel bankacılık kurumu. Pazar gecesi 03:47'de SOC ekibi bir alarm alır: tek bir hesaptan, hiç görülmemiş bir alıcıya, ardışık transferler. Hesap sahibi uyumaktadır. Loglarda kullanıcı düzgünce login olmuş, oturum normal görünmektedir — ama transferler kullanıcının iradesi dışında gerçekleşmiş gibidir. Kaynak kodu, tarayıcı davranışını ve HTTP loglarını incele. Saldırının nasıl bir "tıklama tuzağı" ile gerçekleştirildiğini yeniden inşa et. **Statik CSRF token'ı, gevşek SameSite politikasını ve eksik Origin doğrulamasını** birleştirerek mağdurun tek bir linke tıklamasıyla bakiyenin nasıl uçtuğunu kanıtla.

---

## 🚀 Kurulum

```bash
git clone https://github.com/ZeynepSudeOgan/banka-saldiri.git
cd banka-saldiri
docker compose up -d --build
```

**Uygulama:** `http://localhost:5001`

**Test hesabı:** `user` / `user123` (başlangıç bakiyesi: $24,750.00)

Lab'ı sıfırlamak için:
```bash
docker compose restart
```

Tamamen kapatmak için:
```bash
docker compose down
```

---

## 🎯 Hedefler

Lab'ı tamamlamış sayılman için aşağıdakileri yapman gerekiyor:

- ☐ Mağdur kullanıcının iradesi dışında, **mağdur sadece bir sayfayı açarak**, hesabından bir transfer gerçekleştir.
- ☐ Saldırının çalışmasını mümkün kılan **en az iki ayrı zafiyeti** tespit et.
- ☐ HTTP isteklerinden saldırının **kanıtını** çıkar (Origin, Referer, Cookie davranışı).
---

> **Not:** Çözüme bakmadan önce kendin denemeni öneririm. Sıkıştığında `SOLUTION.md` dosyasında adım adım açıklama bulabilirsin.

---
