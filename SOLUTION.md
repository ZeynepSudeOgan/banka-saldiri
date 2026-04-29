🔍 Çözüm Yolu (Spoiler — Sadece Sıkıştığında Aç)
<details>
<summary><b>İpucu 1 — Nereden başlamalı?</b></summary>
Bir bankacılık uygulamasında en kritik işlem nedir? Para transferi. O isteğin nasıl korunduğuna bak. Form'un HTML kaynağında bir gizli alan var. Onu bir kere not et, oturumu kapat, tekrar gir. Aynı mı, farklı mı?
</details>
<details>
<summary><b>İpucu 2 — Cookie'ler ne diyor?</b></summary>
Login olduktan sonra Burp HTTP history'de response'a bak. Set-Cookie header'larında SameSite değeri ne? Hangi cookie hangi durumlarda gönderilir? Modern tarayıcıların varsayılan davranışı nedir, bu uygulamada ne override edilmiş?
</details>
<details>
<summary><b>İpucu 3 — Saldırı vektörü</b></summary>
Mağduru kandırıp uygulamanın bir endpoint'ine cross-origin bir istek attırabilir misin? HTML'de otomatik submit olan bir form, doğru cookie politikasıyla birleşince ne olur?
</details>
<details>
<summary><b>Tam Çözüm</b></summary>
Zafiyet 1 — Statik CSRF Token
app.py içinde token sabit kodlanmış:
pythonCSRF_TOKEN = "a1b2c3d4e5f6789012345678abcdef00"
Her kullanıcıda, her oturumda aynı değer. Saldırgan bu token'ı bir kez öğrenince (örn. kendi hesabıyla giriş yapıp HTML'i inceleyerek) tüm kullanıcılar için kullanabilir. Token, per-session ve rastgele olmalıydı.
Zafiyet 2 — Cookie SameSite Yapılandırması
session_id cookie'si SameSite=None olarak set ediliyor. Bu, cookie'nin cross-site isteklerde otomatik gönderilmesi anlamına gelir — CSRF saldırısının ön koşulu. Üstelik Secure=False, yani HTTPS bile gerekmiyor.
Zafiyet 3 — Origin/Referer Doğrulaması Yok
Sunucu, /transfer endpoint'ine gelen POST isteğinin nereden tetiklendiğine bakmıyor. Origin başka bir site olsa bile isteği işliyor.
Saldırı PoC
html<!DOCTYPE html>
<html><body>
  <form id="x" action="http://localhost:5001/transfer" method="POST">
    <input type="hidden" name="csrf_token" value="a1b2c3d4e5f6789012345678abcdef00" />
    <input type="hidden" name="recipient" value="attacker_account" />
    <input type="hidden" name="amount" value="5000" />
  </form>
  <script>document.getElementById('x').submit();</script>
</body></html>
Bu dosyayı farklı bir origin'de host et (python3 -m http.server 9090). Mağdur Northfield'a giriş yapmışken bu sayfayı açtığı an, $5,000 saldırgan hesabına geçer. Mağdurun yaptığı tek şey: bir linke tıklamak.
Saldırganın Kendini Ele Verdiği Yerler
Burp HTTP history'de yakalanan istekte:

Origin: http://127.0.0.1:9090 — bankanın kendi domain'i değil
Referer: http://127.0.0.1:9090/csrf_poc.html — kötü niyetli kaynak açık
Cookie: header'ında oturum cookie'leri var ama istek cross-site
Transfer zamanı, kullanıcının normal davranış profiliyle uyuşmuyor

Bu üç nokta, blue team'in tespitte kullanabileceği imzalardır.
Düzeltmeler

CSRF token'ı secrets.token_hex(32) ile session başına üret
SESSION_COOKIE_SAMESITE = "Strict" (kritik işlemler için)
SESSION_COOKIE_SECURE = True ve HTTPS zorunlu
Sunucu tarafında Origin/Referer header doğrulaması
Yüksek meblağlı transferler için ek doğrulama (2FA, e-mail confirm)

</details>