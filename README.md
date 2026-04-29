# 🎯 Banka Saldırısı (Northfield Heist)
### *Private Banking, Public Vulnerabilities*

Bir CSRF (Cross-Site Request Forgery) zafiyet laboratuvarı. Burp Suite ile çözülmek üzere tasarlanmış, gerçekçi bir bankacılık arayüzü üzerinde çalışan, izole bir Docker ortamı.

---

## 📜 Senaryo

**Tarih:** 2 Kasım 2025, Pazar — 03:47

Bankamız butik bir özel bankacılık kurumu. Müşteri portföyünde devlet adamları, miras vakıfları, isimsiz bağışçılar var. Kurumun mottosu yıllardır aynı: *"Discretion is our currency."*

Ama o gece bir şey kırıldı.

Hafta sonu güvenlik nöbetinde olan SOC ekibi, sabah 03:47'de bir alarm aldı: tek bir hesaptan, yarım saat içinde, daha önce hiç işlem görmemiş bir alıcıya **art arda transferler**. Hesap sahibi uyuyordu. Telefonuna dokunmamıştı. Cihazından şikâyet gelmemişti. Loglarda kullanıcı düzgünce login olmuş, oturum normal görünüyor — ama transferler kullanıcının iradesi dışında gerçekleşmiş gibi.

Yönetim panik halinde. Basın saat dokuzda kapıya dayanacak. Senin elinde sadece şunlar var:

- Uygulamanın kaynak kodu
- Tarayıcının davranışı
- Sunucunun loglarına erişim
- Ve birkaç saatlik zaman

Saldırgan iz bırakmadığını sanıyor. Ama her saldırgan bir yerde hata yapar — ya tasarımda, ya isteğin kendisinde, ya da güvendiği bir varsayımda.

> **Görevin:** Saldırının nasıl gerçekleştirildiğini yeniden inşa et. Mağdurun tek bir tıklaması bu kadar parayı nasıl uçurabildi? Kanıtları topla, zafiyet zincirini çıkar, raporu yaz.
>
> **Saldırgan kendini nerede ele verdi? Bul.**

---

## 🛠️ Laboratuvar Senaryoları

### 🟢 Level 1: Banka Saldırısı (CSRF Exploitation)

**📚 Konular:** `CSRF` · `SameSite Cookies` · `Session Management` · `Origin Validation` · `Burp Suite Proxy` · `Defense in Depth`

| Özellik | Değer |
|---------|-------|
| **Zorluk** | ⭐⭐ Kolay-Orta |
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
- ☐ Düzeltme önerilerini yaz.

---

## 🛡️ Defansif Bakış

Bu lab bir **red team alıştırması** gibi görünüyor ama esas öğretmek istediği şey şu: bir saldırı başarılı olduktan **sonra** bile, saldırgan log'larda parmak izi bırakır. İyi bir blue team şu soruları sorar:

- İstek **hangi Origin'den** geldi?
- Kullanıcının **olağan saatlerinde mi** tetiklendi?
- Recipient hesabı **daha önce hiç görülmüş mü**?
- Aynı session'da **anormal hızlı** ardışık işlem var mı?

Saldırgan tasarımdaki açığı buldu, evet. Ama bu izleri silemedi. Senin işin: izleri okumak ve hikâyeyi yeniden anlatmak.

---

## 📁 Repo Yapısı

```
banka-saldiri/
├── README.md                ← Bu dosya
├── SOLUTION.md              ← Tam çözüm (spoiler içerir)
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── app.py                   ← Flask uygulaması (zafiyetler burada)
└── templates/
    ├── login.html
    └── transfer.html
```

> **Not:** Çözüme bakmadan önce kendin denemeni öneririm. Sıkıştığında `SOLUTION.md` dosyasında adım adım açıklama bulabilirsin.

---

## ⚠️ Yasal Uyarı

Bu laboratuvar yalnızca **eğitim amaçlı** ve izole ortamlarda çalıştırılmak üzere hazırlanmıştır. Burada öğrenilen tekniklerin, sahibi olmadığınız ya da yazılı izniniz bulunmayan sistemlerde uygulanması yasalara aykırıdır ve ciddi cezai sorumluluk doğurur. Bilginizi yalnızca koruma amaçlı kullanın.

---

> *"Every breach tells a story. Your job is to read it before the press does."*

