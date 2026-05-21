# Risk Checklist — Canonical Tarama Yüzeyleri

Her audit'te bu listedeki her yüzey kontrol edilir. Hiçbiri atlanamaz — atlanırsa raporun "Tarama Dışı Bırakılanlar" bölümüne not edilir.

---

## 1. PII Toplama (Kişisel Veri Toplama)

### Tarama
```bash
# Form elementleri
grep -rn "<form\|<input\|<textarea" --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.html"

# React form state
grep -rn "useState.*{.*\(name\|email\|phone\|isim\|tarih\|telefon\)" --include="*.tsx" --include="*.jsx"

# API endpoint'lerinde gelen alanlar (Next.js route handlers)
grep -rn "body\.\(name\|email\|phone\|isim\|tarih\|kullanici\)" --include="route.ts" --include="route.js"

# Backend (Python Flask)
grep -rn "request\.\(form\|json\|args\).*\(name\|email\|phone\)" --include="*.py"
```

### Risk seviyeleri
- 🟢 **Sadece email** (newsletter/contact için): aydınlatma + açık rıza şart, KVKK md. 5/1
- 🟡 **Email + isim + ek bilgi** (kayıt, profil): tam paket gerekli
- 🔴 **TC kimlik, doğum tarihi, anne adı, sağlık, finansal**: özel nitelikli/hassas veri alanına yaklaşıyor; KVKK md. 6 olası

### Mitigasyon
- `/legal-lens generate kvkk` → aydınlatma + açık rıza
- Form öncesi consent checkbox
- Veri minimizasyonu: gerçekten gerekli olmayan alanları kaldır

---

## 2. Üçüncü Taraf Veri Aktarımı

### Tarama
`package.json` veya `requirements.txt` veya benzeri içinde:

| Bağımlılık | Servis | Veri aktarımı | Coğrafya |
|---|---|---|---|
| `@anthropic-ai/sdk` | Anthropic Claude | Prompt + cevap | ABD |
| `openai` | OpenAI | Prompt + cevap | ABD |
| `@google/generative-ai` | Google Gemini | Prompt + cevap | ABD |
| `@supabase/supabase-js` | Supabase | Auth + DB | EU/ABD (proje seçimine göre) |
| `firebase` | Firebase | Auth + DB + Analytics | ABD |
| `stripe` | Stripe | Ödeme + kart bilgisi | ABD/Avrupa |
| `@vercel/analytics` | Vercel Analytics | Sayfa görüntüleme | ABD |
| `@vercel/speed-insights` | Vercel Insights | Performance | ABD |
| `posthog-js`, `posthog-node` | PostHog | Behavior tracking | ABD/EU |
| `mixpanel-browser` | Mixpanel | Behavior tracking | ABD |
| `google-analytics`, `gtag` | Google Analytics | Behavior tracking | ABD |
| `segment` | Segment | Behavior tracking | ABD |
| `sentry` | Sentry | Error + context | ABD/EU |
| `intercom`, `crisp` | Müşteri sohbet | Kullanıcı mesajları | ABD/EU |
| `mongodb`, `pg`, `mysql2` | DB driver | Tüm veriler | hosting'e bağlı |
| `redis`, `ioredis` | Redis | Session/cache | hosting'e bağlı |

### Risk
- Üçüncü taraf veri işliyorsa → **KVKK md. 9 yurt dışı aktarım** veya yurt içi aktarım kuralları
- ABD'ye giden veri → açık rıza özelinde belirtilmeli
- Analytics SDK'ları → kullanıcı izleme, çerez politikası gerekli

### Mitigasyon
- Aydınlatma metnine her üçüncü tarafı **ad ad** ekle
- Hangi verinin gittiğini ve nedenini açıkla
- Açık rıza checkbox'ında "şu firmaya aktarılacak" demek zorunlu
- Veri işleyen sözleşmesi (DPA) — bu skill kapsamı dışı, avukat işi

---

## 3. Server-Side Veri Kalıcılığı

### Tarama
```bash
# Veritabanı kullanımı
grep -rn "prisma\|drizzle\|sequelize\|mongoose\|knex" --include="*.ts" --include="*.js" --include="*.tsx"

# Vercel KV/Postgres
grep -rn "@vercel/kv\|@vercel/postgres" --include="*.ts"

# Redis
grep -rn "createClient.*redis\|ioredis" --include="*.ts"

# Dosya yazımı
grep -rn "fs\.\(writeFile\|appendFile\|createWriteStream\)" --include="*.ts" --include="*.js"

# Log'larda PII?
grep -rn "console\.\(log\|info\|warn\|error\).*\(body\|form\|email\|isim\|name\)" --include="*.ts"
```

### Risk
- Veri saklanıyorsa → **saklama süresi belirlenmeli**, aydınlatma metnine yazılmalı
- DB'de PII varsa → VERBİS kayıt değerlendirmesi
- Log'larda PII varsa → veri ihlali halinde kapsam genişler

### Mitigasyon
- Saklama süresi politikası belirle (KVKK md. 7 — amaca gerekli süre)
- Kullanıcının silme hakkı için (md. 11) kod desteği — hesap silme akışı
- Log'larda PII'yi maskele (`email: u***@***.com`)

---

## 4. Riskli İddialar / Copy

### Tarama
Ana sayfa, landing copy, hero text, marketing materyalleri, AI çıktı şablonları:

```bash
grep -rn -iE "(tedavi|iyileş|şifa|garanti|kesin sonuç|kazanç|kâr|fal|kehanet|gelecek|astrol|terapi|teşhis|diyet|kilo)" --include="*.tsx" --include="*.jsx" --include="*.md"
```

İngilizce:
```bash
grep -rn -iE "(cure|heal|treatment|guarantee|profit|fortune|prophecy|diagnos|therapy|diet|weight loss)" --include="*.tsx" --include="*.jsx"
```

### Risk alanları
- **Sağlık**: tedavi, iyileşme, tıbbi/psikolojik teşhis → Sağlık Bakanlığı + Tüketici → 🔴
- **Finans**: yatırım tavsiyesi, kazanç vaadi → SPK + BDDK → 🔴
- **Kehanet/spirituel**: garanti edilmiş kehanet → düşük yasal, yüksek sosyal → 🟡
- **Eğitim/sertifika**: sertifika vaadi, MEB onayı iddiası → 🟡
- **Gıda**: alerjen, organik, güvenlik iddiası → Tarım Bakanlığı → 🟡
- **Çocuk**: 18 altı kullanıcı, çocuk içeriği → COPPA + KVKK md. 5/2 → 🔴

### Mitigasyon
- Disclaimer ekle (kullanım koşullarında + ilgili sayfada görünür)
- "Tıbbi tavsiye değildir", "Yatırım tavsiyesi değildir" gibi net ifadeler
- 18 yaş kontrolü zorla (consent checkbox'ta)
- AI çıktıları için ek not: "Yapay zekâ tarafından üretilmiştir, doğruluğu garanti edilmez"

---

## 5. Eksik Yasal Sayfalar

### Tarama
```bash
# Next.js App Router
ls app/aydinlatma 2>/dev/null && echo "KVKK Aydınlatma: ✓" || echo "KVKK Aydınlatma: ✗"
ls app/kosullar 2>/dev/null && echo "Kullanım Koşulları: ✓" || echo "Kullanım Koşulları: ✗"
ls app/privacy 2>/dev/null && echo "Privacy: ✓" || echo "Privacy: ✗"
ls app/terms 2>/dev/null && echo "Terms: ✓" || echo "Terms: ✗"

# Pages Router
ls pages/aydinlatma.* pages/kosullar.* pages/privacy.* pages/terms.* 2>/dev/null

# Generic
find . -iname "privacy*" -o -iname "terms*" -o -iname "aydinlatma*" -o -iname "kosullar*" -o -iname "kvkk*" 2>/dev/null | grep -v node_modules
```

### Beklenen sayfalar (jurisdiction'a göre)

**KVKK (Türkçe odaklı)**:
- `/aydinlatma` — KVKK md. 10 aydınlatma metni
- `/kosullar` — kullanım koşulları + sorumluluk reddi
- (e-ticaret varsa) `/mesafeli-satis-sozlesmesi`, `/cayma-hakki`, `/iletisim`
- (çerez varsa) `/cerez-politikasi`

**GDPR (EU)**:
- `/privacy` veya `/privacy-policy` — Article 13 information notice
- `/terms` veya `/terms-of-service`
- (çerez varsa) `/cookie-policy` + cookie banner

### Footer kontrolü
```bash
grep -rn "Footer\|<footer" --include="*.tsx" --include="*.jsx" | head -5
```

Footer var ama yasal linkler eksik mi?
```bash
grep -A 20 "<footer\|Footer()" --include="*.tsx" -rh | grep -iE "(aydinlatma|kosullar|privacy|terms)"
```

### Mitigasyon
- `/legal-lens generate kvkk` veya `/legal-lens generate gdpr`
- `/legal-lens footer`

---

## 6. Cookie ve Tracking

### Tarama — kategori bazlı

**A. Kesinlikle gerekli çerezler** (banner gerekmez):
```bash
# Auth oturum çerezi
grep -rn "supabase.*auth\|next-auth\|iron-session\|cookie-session" \
  --include="*.tsx" --include="*.ts" --include="package.json"

# CSRF/security cookies
grep -rn "csrf\|X-CSRF" --include="*.tsx" --include="*.ts"

# Dil/tema localStorage
grep -rn "localStorage\.setItem.*\(theme\|language\|locale\|lang\)" \
  --include="*.tsx" --include="*.ts"
```

**B. Analitik çerezler** (açık rıza gerekli → banner zorunlu):
```bash
grep -rn "@vercel/analytics\|@vercel/speed-insights\|google-analytics\|gtag\|posthog\|mixpanel\|@segment\|plausible\|umami" \
  --include="*.tsx" --include="*.ts" --include="package.json"
```

**C. Pazarlama/Tracking pixel** (açık rıza gerekli → banner zorunlu):
```bash
grep -rn "fbq\|facebook.*pixel\|tiktok\.com\|linkedin\.com/insight\|hotjar\|fullstory\|clarity" \
  --include="*.tsx" --include="*.ts"
```

**D. 3rd-party embed çerezleri** (genelde rıza gerekli):
```bash
grep -rn "youtube\.com/embed\|twitter\.com/widgets\|platform\.twitter\.com\|instagram\.com/embed\|tiktok\.com/embed" \
  --include="*.tsx" --include="*.jsx" --include="*.html"
```

### Karar matrisi

| Tespit | Banner gerekli? | Politika sayfası | Aydınlatma |
|---|---|---|---|
| Sadece A (oturum, tema) | ❌ Hayır | İsteğe bağlı | "Çerez kullanılmaz" notu |
| B veya C var | ✅ EVET | ✅ EVET | Politikaya link |
| D var ama gizli mod (privacy-enhanced) | ❌ Hayır | İsteğe bağlı | "Embed iframe'leri kullanılır" |
| D var, normal mod | ✅ EVET | ✅ EVET | Politikaya link |

### Risk
- 🔴 **B/C tespit edildi + banner yok**: KVKK Kurul kararı (2020/216 ve sonrası) gereği açık rıza zorunlu, ihlali idari para cezası
- 🟡 **A yalnız + politika sayfası yok**: Aydınlatma metninde belirtilirse yeterli
- 🟢 **Hiç çerez/storage yok**: Aksiyon gerekmez

### Mitigasyon
- `/legal-lens cookies` → tarama yapar, gerekiyorsa banner + politika üretir
- Mevcut analytics'i `<ConsentGate>` ile sar (otomatik yapılır)
- A kategorisindekiler için sadece aydınlatma metninde belirt

---

## 7. Deploy Durumu

Tek sinyale güvenme — `.vercel/project.json` lokal-stale olabilir, CLI auth düşmüş olabilir, "deploy protection" 401 dönüyor olabilir. Aşağıdaki 4 katmanı sırayla çalıştır, sonuçları cross-check et.

### Tarama — Katman A (repo işaretleri)
```bash
# Hosting marker dosyaları
[ -f .vercel/project.json ] && echo "Vercel marker: ✓" && cat .vercel/project.json
[ -f netlify.toml ] && echo "Netlify: ✓"
[ -f Procfile ] && echo "Heroku-style: ✓"
[ -f wrangler.toml ] && echo "Cloudflare Workers: ✓"
[ -f app.yaml ] && echo "GCP App Engine: ✓"
[ -f fly.toml ] && echo "Fly.io: ✓"
```

### Tarama — Katman B (CLI doğrulama)
```bash
# Vercel — projenin gerçekten remote'ta var olduğunu doğrula
vercel project ls 2>&1 | head -10
# "Not authenticated" → kullanıcıya "vercel login" gerekir notu
# "No projects" + lokal marker var → marker stale, kullanıcıya sor
```

### Tarama — Katman C (production URL probe)
```bash
# Vercel projeId → URL
PROJECT_ID=$(jq -r .projectId .vercel/project.json 2>/dev/null)
vercel inspect "$PROJECT_ID" 2>&1 | grep -iE "url|name" | head -5

# URL'i probe et — public mi yoksa Vercel Auth 401 mi?
curl -sI "https://<production-url>" | head -3
# 200/301/302 → halka açık
# 401 → deploy protection aktif (sadece auth'lu kullanıcılar erişebilir)
# 404 → URL stale ya da silinmiş
```

### Tarama — Katman D (git remote)
```bash
git remote -v 2>/dev/null
# Public GitHub/GitLab repo + form/PII var → kod paylaşımı zaten "ifşa"
# Bonus: secret leak check
git ls-files .env* 2>/dev/null  # boş olmalı; doluysa 🔴 hassas leak
```

### Karar matrisi

| Katman A | Katman B | Katman C | Sonuç | Risk modifier |
|---|---|---|---|---|
| Marker var | CLI doğruladı | 200 | **Halka açık deploy** | +1 (form + PII varsa 🔴) |
| Marker var | CLI doğruladı | 401 | **Deploy korumalı** (Vercel Auth) | nötr (sadece sen erişiyorsun, raporda belirt) |
| Marker var | CLI doğrulayamadı | — | **Deploy iddiası, doğrulanamadı** | kullanıcıya sor, "manuel doğrula" not düş |
| Marker yok | — | — | **Sadece yerel** | 🟢 (deploy etmeden önce hazırla) |
| — | — | — | Public git repo + PII | Bonus 🟡 (kod paylaşılıyor) |

### Risk
- **Halka açık deploy + form + yasal sayfa yok** → 🔴
- **Deploy korumalı (401)** → KVKK riski daha düşük çünkü public erişim yok; ama prod'a açıldığında hazır olsun → 🟡
- **Sadece yerel** → 🟢 (deploy etmeden önce hazır olsun)
- **Public repo + `.env*` izleniyor** → 🔴 secret leak ayrı bir aksiyon

---

## 8. UGC (Kullanıcı Üretimli İçerik)

### Tarama — çok sinyalli, en az 2 hit UGC platform olduğuna işaret eder

**A. Veri yazma sinyalleri** (DB'ye kullanıcı içeriği gidiyor mu):
```bash
# Supabase insert/upsert
grep -rn "supabase\.from\(.*\)\.insert\|supabase\.from\(.*\)\.upsert" \
  --include="*.tsx" --include="*.ts" | head -20

# Prisma create/createMany
grep -rn "prisma\.\w*\.\(create\|createMany\|upsert\)" --include="*.ts" | head -10

# Storage upload (görsel/dosya UGC)
grep -rn "supabase\.storage\.from\(.*\)\.upload\|formData\.append\(.*[Ff]ile" \
  --include="*.tsx" --include="*.ts" | head -10
```

**B. Form/input sinyalleri** (UGC alanları):
```bash
# Textarea ile içerik yazma
grep -rn "<textarea\|<Textarea" --include="*.tsx" --include="*.jsx" | head -10

# UGC alan adları
grep -rn -iE "useState.*\(content|post|comment|message|caption|description|review|note|recipe)" \
  --include="*.tsx" | head -10

# Dosya/görsel input
grep -rn 'type="file"\|accept="image' --include="*.tsx" | head -5
```

**C. Sosyal ilişki sinyalleri** (sosyal platform iddiası):
```bash
# Tablolar: follows, likes, votes, saves, conversations, messages, comments
grep -rn -iE "from\(.(follows|likes|votes|saves|comments|messages|conversations|reactions)" \
  --include="*.tsx" --include="*.ts" | head -10
```

**D. URL pattern'ları** (kullanıcı profili / paylaşılan içerik):
```bash
ls app/u/ app/user/ app/profile/ app/\[username\] app/\[handle\] 2>/dev/null
find app -type d -name "\[*Id\]" -o -name "\[slug\]" 2>/dev/null | head -10
```

### Karar matrisi

| A | B | C | D | Sonuç |
|---|---|---|---|---|
| ✓ | ✓ | ✓ | ✓ | **Tam UGC platform** — tüm yükümlülükler 🔴 |
| ✓ | ✓ | — | ✓ | **UGC siteyi paylaşıyor ama sosyal değil** (blog, portfolio kayıt) 🟡 |
| ✓ | ✓ | — | — | **Form-only UGC** (contact form, recipe submission) 🟡 hafif |
| — | ✓ | — | — | **Form ama persistence yok** (frontend-only state) 🟢 |
| Hiçbiri | | | | UGC yok 🟢 |

### Risk (UGC platform tespit edildiyse — 🔴 yükümlülükler)

UGC barındıran platform için Türkiye'de **birden fazla yasa kapsamı**:

1. **5651 sayılı Kanun** (İnternet Yayın Düzenlemesi):
   - **Yer sağlayıcı yükümlülüğü** — kullanıcı içeriğini barındırıyorsun
   - **Haberdar edildiğinde içeriği kaldırma** zorunluluğu (24 saat)
   - **İçerikle ilgili iletişim formu/email** zorunlu (kolayca bulunabilir yerde)
   - **Trafik bilgilerini saklama** (6 ay-2 yıl arası, BTK düzenlemesi)

2. **Telif** (5846 sayılı FSEK):
   - Kullanıcı başkasının fotoğrafını/yazısını yüklerse → **uyarı-kaldırma (notice-and-takedown)** mekanizması olmazsa platform müteselsil sorumlu
   - DMCA-style süreç olmazsa "iyi niyetli yer sağlayıcı" savunması zayıflar

3. **Hakaret / kişilik hakları** (Borçlar K. md. 49):
   - Kullanıcı başkasını hakaret ederse şikayet üzerine 24 saat içinde kaldırma

4. **Çocuk koruması**:
   - 13 yaş altı kullanıcı içerik yüklüyorsa KVKK md. 5/2-d (özel rıza)
   - Cinsel sömürü içeriği için **zero-tolerance + ihbar** zorunlu (Türk Ceza Kanunu md. 226)

5. **GDPR (EU kullanıcısı varsa)**:
   - Article 17 — silme hakkı (kullanıcı içeriği)
   - Article 20 — taşınabilirlik (UGC export)
   - DSA (Digital Services Act, 2024) — büyükse içerik moderasyon raporu

### Zorunlu yapı taşları (UGC platformu için)

| Yapı | Var mı diye kontrol | Yoksa risk |
|---|---|---|
| **Kullanım Koşullarında UGC bölümü** | `grep -l "kullanıcı içeriği\|user content" app/kosullar/` | 🔴 |
| **İçerik bildirim/şikayet formu** | `ls app/sikayet app/report app/iletisim 2>/dev/null` | 🔴 |
| **İletişim email (5651)** | Aydınlatma + footer'da email var mı | 🔴 |
| **Yaş kontrolü (consent'te 18+ ya da 13+)** | `grep "18\|13 yaş" app/auth/signup/` | 🟡-🔴 |
| **Hesap fesih politikası** | Koşullarda "ihlal halinde fesih" maddesi | 🟡 |
| **İçerik kaldırma mekanizması** | Admin/moderator panel ya da `delete` endpoint | 🟡 |

### Mitigasyon
- Kullanım koşullarında UGC bölümü (telif beyanı, indemnity, kaldırma hakkı)
- `/iletisim` veya `/sikayet` formu — minimum: email + ihlal türü dropdown + URL + açıklama
- Footer'da yer sağlayıcı bilgisi: "Yer sağlayıcı: [Ad] · İletişim: [email]"
- Hesap silme akışı (`/legal-lens rights` zaten üretir)
- Çocuk istismarı için açık raporlama hattı + zero-tolerance metni

---

## 9. Yaş Doğrulama

### Tarama
```bash
grep -rn "yaş\|age\|18\+\|reşit\|adult" --include="*.tsx" --include="*.jsx"
```

### Risk
- 13 yaş altı kullanıcılar → **COPPA** (ABD) + KVKK md. 5/2-d (Türkiye)
- 16 yaş altı kullanıcılar → **GDPR çocuk verisi** koruması (Article 8)
- Erotik/şiddet içerik → 18+ doğrulama yasal zorunlu

### Mitigasyon
- Consent checkbox'a "18 yaşımı doldurdum" satırı (yıldızname'de yapıldı)
- Çocuk içeriği varsa veli onayı akışı (kapsam dışı, avukat işi)

---

## 10. Veri Sahibi Hakları Endpoint'leri (KVKK md. 11)

Veri saklayan herhangi bir uygulamada (auth sistemi + DB tutuyorsa) kullanıcının kendi hakkını uygulayabilmesi için arayüz/endpoint gerekli.

### Tarama

```bash
# Auth + DB kullanılıyor mu?
AUTH=$(grep -l "supabase\.auth\|next-auth\|clerk\|auth0" --include="*.ts" --include="*.tsx" -r . 2>/dev/null | head -1)
DB=$(grep -l "supabase\|prisma\|@vercel/postgres" --include="package.json" -r . 2>/dev/null | head -1)
[ -n "$AUTH" ] && [ -n "$DB" ] && echo "Auth+DB tespit edildi → md. 11 endpoint'leri gerekli"

# /hesabim, /account, /profile gibi bir sayfa var mı?
ls app/hesabim app/account app/profile app/settings/account \
   pages/hesabim pages/account pages/profile 2>/dev/null

# Veri export endpoint'i?
grep -rn "GET.*\(export\|download\|verilerim\)" app/api/ 2>/dev/null
find . -path "*/api/*export*" -type f 2>/dev/null

# Hesap silme endpoint'i?
grep -rn "DELETE.*\(me\|account\|hesap\)" app/api/ 2>/dev/null
find . -path "*/api/me*" -type f 2>/dev/null
```

### Karar matrisi

| Durum | Risk | Mitigasyon |
|---|---|---|
| Auth+DB var, /hesabim yok | 🔴 Yüksek | `/legal-lens rights` — panel + export + silme üret |
| Auth+DB var, /hesabim var ama export yok | 🟡 Orta | Export endpoint ekle |
| Auth+DB var, silme yok | 🔴 Yüksek | Silme endpoint zorunlu |
| Auth+DB var, silme var ama UGC stratejisi belirsiz | 🟡 Orta | Strateji belirle, kullanım koşullarında açıkla |
| Auth yok / DB yok | 🟢 Düşük | Email tabanlı talep mekanizması yeterli |

### Risk gerekçesi

- **KVKK md. 11/e** — silme/yok etme hakkı zorunlu
- **KVKK md. 13** — başvurulara **30 gün içinde** cevap zorunlu; otomatik mekanizma olmazsa manuel cevap vermek mücbir
- **KVKK md. 18** — bu haklara uyulmaması idari para cezasının ağırlaştırıcı sebebi olabilir

### Mitigasyon

- `/legal-lens rights` → tam paket: panel + export API + silme API
- Service role key'in güvenli yönetimi (sadece server-side)
- UGC için strateji (sil / anonimleştir / soft delete) — kullanım koşullarında belirt

---

## 11. E-Ticaret / Ödeme (v0.3'te genişletildi)

### Tarama — kapsamlı

**A. Ödeme sağlayıcı SDK'ları**:
```bash
# Türkiye odaklı
grep -rn "iyzico\|paytr\|@paytr/\|sipay\|shopier\|payu" \
  --include="package.json" --include="*.ts" --include="*.py"

# Global
grep -rn "@stripe/\|stripe-js\|stripe-node\|@paypal\|paddle\|braintree\|@adyen" \
  --include="package.json" --include="*.ts"

# Mobile/local
grep -rn "lemonsqueezy\|@chargebee\|@recurly\|cryptomus" \
  --include="package.json" --include="*.ts"
```

**B. Ödeme akışı sinyalleri**:
```bash
# Checkout pages
ls app/checkout app/odeme app/cart app/sepet pages/checkout pages/odeme 2>/dev/null

# Order/sipariş tabloları
grep -rn -iE "from\(.(orders|siparis|cart|sepet|payments|odemeler)" \
  --include="*.tsx" --include="*.ts" | head -10

# Stripe checkout/iyzico session
grep -rn "stripe\.checkout\.sessions\|iyzipay.*payment\|payTRRequest" \
  --include="*.ts" | head -10
```

**C. Ürün/hizmet sinyalleri**:
```bash
# Product catalog
grep -rn -iE "(price|fiyat|stock|stok|sku|product|urun|hizmet)" \
  --include="*.tsx" --include="*.ts" | grep -iE "(useState|interface|type )" | head -10

# Subscription patterns
grep -rn -iE "(subscription|abone|plan|tier|recurring)" \
  --include="*.tsx" --include="*.ts" | head -5
```

**D. Satış konusu tespit**:
```bash
# Fiziksel ürün (kargo, shipping address)
grep -rn -iE "(shipping|kargo|delivery|address|adres)" --include="*.tsx" | head -5

# Dijital ürün/hizmet (no shipping, instant delivery)
grep -rn -iE "(download|access|license|abonelik|trial)" --include="*.tsx" | head -5
```

### Karar — Satış türü

| Sinyaller | Alt-tür | Cayma hakkı | Vergisel |
|---|---|---|---|
| Stripe + shipping address + cart | **Fiziksel ürün satışı** | 14 gün (6502 md. 48) | KDV + ETBİS |
| Stripe + instant access + download | **Dijital ürün** | Cayma hakkı **istisnası** (md. 15/1-ğ) | KDV + ETBİS |
| iyzico + abonelik + recurring | **SaaS / abonelik** | 14 gün ilk satınalmada | KDV + ETBİS + abonelik özel hüküm |
| Stripe + tek seferlik hizmet | **Hizmet sunumu** | Hizmet **tamamlanmadıysa** 14 gün; tamamlandıysa istisna | KDV + ETBİS |
| Sadece Stripe Connect / marketplace | **Marketplace platformu** | Karmaşık — satıcı + platform sorumluluğu ayrı | ETBİS aracı sıfatıyla kayıt |

### Risk — Türkiye e-ticaret yükümlülükleri

#### 🔴 ETBİS kayıt zorunluluğu yok
- **Elektronik Ticaret Hakkında Kanun (6563 sayılı)** + 2024 değişiklikleri
- Türkiye'de e-ticaret faaliyeti yapan tüm gerçek/tüzel kişiler **ETBİS sistemine kayıt** zorunlu
- Kayıt: https://etbis.ticaret.gov.tr
- Kayıt **olmadan** ödeme sağlayıcı (iyzico, PayTR vb.) çoğunlukla onay vermez
- **Halka açık üye olmuş işletmeler** — ETBİS kayıt no. footer'da görünür olmalı

#### 🔴 Mesafeli Satış Sözleşmesi yok
- **6502 sayılı Tüketici K. md. 48 + Mesafeli Sözleşmeler Yön.**
- Tüketiciye sözleşme metni **satın alma öncesi** sunulmalı + onay alınmalı
- İçerik: satıcı bilgileri, ürün/hizmet, fiyat, ödeme, teslimat, **cayma hakkı**, şikayet mekanizması, çözüm makamları
- **Eksikse**: yapılan satış **geçersiz** sayılabilir, tüketici lehine

#### 🔴 Cayma Hakkı (14 gün) yok
- **6502 sayılı K. md. 48/4**: Tüketici **14 gün** içinde sebep göstermeksizin sözleşmeden cayabilir
- İstisnalar (md. 15): hızlı bozulan ürünler, ısmarlama, açılmış dijital içerik, vb.
- **Cayma formu** tüketiciye verilmeli + e-ticaret sitesinde **kolay erişilebilir** olmalı
- **Eksikse**: cayma süresi **1 yıl 14 gün'e uzar** (md. 48/5)

#### 🔴 İade/iptal akışı yok
- Kullanıcı cayma hakkını **nasıl** kullanır?
- Form + e-posta + manuel akış kabul edilebilir; ama erişilebilir + ücretsiz olmalı
- 14 gün içinde iade ödemesi yapılmalı (yasal süre)

#### 🟡 Tüketici Hakem Heyeti bilgisi yok
- Belirli para sınırı altında (2024 itibariyle ~57.000 TL) Hakem Heyeti zorunlu yetkili
- Üstü için Tüketici Mahkemesi
- Bu bilgi sözleşmede + sitenin "Çözüm Mekanizmaları" bölümünde yer almalı

#### 🟡 Vergi mükellefiyeti belirsiz
- Şahıs şirketi olmadan satış → mali müşavir review zorunlu
- KDV mükellefi olup olmadığı → KDV faturası gerekli
- Yurt dışı satışta KDV durumu (B2B vs B2C)
- **Bu skill kapsamı dışı**, mali müşavir/SMMM zorunlu

### Risk — EU e-ticaret yükümlülükleri (paralel)

#### 🔴 14-day withdrawal right yok (EU Consumer Rights Directive)
- EU CRD Art. 9-16 — tüketici 14 gün içinde sebep göstermeksizin sözleşmeden çekilebilir
- Form + clear information + 14-day reimbursement

#### 🔴 VAT MOSS / OSS kayıt
- EU'da fiziksel veya dijital satış yapan operatör → VAT One Stop Shop kaydı
- EU dışı operatör + EU müşterisi → IOSS (Import OSS) veya doğrudan ülke kaydı
- **2021+ değişiklik**: B2C dijital satışta vergi alıcının ülkesine göre

#### 🟡 Cookies + tracking için "legitimate interest" ≠ ödeme sayfası
- Ödeme sayfasında bile reklam pixel/tracking olmamalı
- PCI-DSS uyumu için strict cookie policy

### Zorunlu yapı taşları (TR pazarı)

| Yapı | Kontrol | Yoksa risk |
|---|---|---|
| **Mesafeli Satış Sözleşmesi sayfası** | `ls app/mesafeli-satis* pages/mesafeli-satis*` | 🔴 |
| **Cayma Hakkı sayfası + form** | `ls app/cayma-hakki* app/iade-iptal*` | 🔴 |
| **ETBİS kayıt notu (footer veya sayfa)** | `grep -rn "ETBİS\|etbis" --include="*.tsx"` | 🔴 |
| **Tüketici Hakem Heyeti bilgisi** | `grep -rn "Hakem Heyeti\|tüketici hakemi"` | 🟡 |
| **KDV/fatura bilgisi** | Aydınlatma + sözleşmede vergi bilgisi | 🟡 |
| **Çözüm/şikayet mekanizması** | Email + iletişim bilgisi | 🟡 |
| **Pre-checkout consent checkbox** | "Mesafeli satış sözleşmesini okudum, kabul ediyorum" | 🔴 |

### Mitigasyon — Şablonlar (v0.3)

- `/legal-lens generate ecommerce` → şu paketi üretir:
  - `templates/mesafeli-satis.md` → `/mesafeli-satis-sozlesmesi` sayfası
  - `templates/cayma-hakki.md` → `/cayma-hakki` sayfası + form
  - `templates/iade-iptal.md` → `/iade-iptal` akışı
  - `templates/etbis-bilgi-formu.md` → footer notu + bilgi sayfası
  - Checkout flow'a consent checkbox
  - Mali müşavir/avukat review uyarısı

### Mitigasyon — Ekosistem entegrasyonu

- **Ödeme sağlayıcı vendor docs**: iyzico/PayTR/Stripe'ın e-ticaret rehberi
- **ETBİS başvuru süreci**: T.C. Ticaret Bakanlığı sitesinden adım adım
- **SMMM/mali müşavir**: Vergi optimizasyonu için zorunlu danışmanlık

---

## 12. Gıda / Sağlık / Wellness Platformları

Genel "riskli iddialar" (§4) yeterli değil — gıda ve sağlık platformlarında ek yükümlülükler var (alerjen sorumluluğu, tarif doğruluğu, profesyonel tavsiye sınırı, fitness/beslenme tavsiyesi). Recipe sharing, wellness, meditation, fitness gibi platform tiplerinde zorunlu kontrol.

### Tarama — platform türünü tespit et

**A. Gıda/tarif platformu sinyalleri**:
```bash
# Kelime sinyalleri (TR + EN)
grep -rn -iE "(tarif|recipe|malzeme|ingredient|pişir|cook|bake|fırın|oven|mutfak|kitchen)" \
  --include="*.tsx" --include="*.md" 2>/dev/null | head -20

# Tablo isimleri
grep -rn -iE "from\(.(recipes|experiments|dishes|ingredients|cookbook)" \
  --include="*.tsx" --include="*.ts" 2>/dev/null | head -10
```

**B. Sağlık/wellness platformu sinyalleri**:
```bash
# Wellness/spiritüel
grep -rn -iE "(meditasyon|meditation|chakra|nefes|breath|yoga|mindful|terapi|therapy|healing|şifa|iyileşme)" \
  --include="*.tsx" --include="*.md" 2>/dev/null | head -20

# Fitness/beslenme
grep -rn -iE "(diyet|diet|kalori|calorie|kilo|weight|fitness|antrenman|workout|protein|makro|macro)" \
  --include="*.tsx" --include="*.md" 2>/dev/null | head -20

# Tıbbi/teşhis
grep -rn -iE "(teşhis|diagnos|tedavi|treatment|hastalık|disease|ilaç|medicine|semptom|symptom)" \
  --include="*.tsx" --include="*.md" 2>/dev/null | head -20
```

### Karar — platform alt-türü

| Sinyaller | Alt-tür | Kritik risk alanı |
|---|---|---|
| A + UGC (§8) | **Gıda UGC** (recipe sharing platform) | Alerjen, tarif doğruluğu, beslenme tavsiyesi |
| A statik (blog, kişisel tarif) | **Gıda yayını** | Hafif: kaynak referans + "kişisel deneyim" notu |
| B wellness (chakra, meditasyon, breathwork) | **Wellness/spiritüel** | "Tıbbi tavsiye değildir" zorunlu |
| B fitness/beslenme | **Fitness/beslenme** | Profesyonel uzman olmadan birey-spesifik tavsiye yasak |
| B tıbbi (teşhis/tedavi iddiası) | **Sağlık iddiası** | 🔴 Sağlık Bakanlığı + reklam mevzuatı; avukatla görüş zorunlu |

### Risk — Gıda UGC (recipe sharing platforms)

1. **Alerjen sorumluluğu**:
   - Türk Gıda Kodeksi Etiketleme Yönetmeliği (14 alerjen zorunlu beyan)
   - Kullanıcı tarifi paylaşıyorsa platform direkt sorumlu değil ama "alerjen kontrolü size aittir" disclaimer'ı kullanım koşullarında olmalı
2. **Tarif doğruluğu / gıda güvenliği**:
   - Yetersiz pişmiş et, çiğ süt, pastörize edilmemiş içerik → fermentasyon/konserve tarifleri özellikle riskli (botulizm)
   - "Bu tarif sınanmamıştır, kendi sorumluluğunuzda uygulayın" disclaimer'ı zorunlu
3. **Profesyonel beslenme tavsiyesi**:
   - "Kilo verme diyeti", "diyabet tarifi", "celiac güvenli" → diyetisyen olmadan tıbbi/beslenme tavsiyesi yasak (6502 sayılı Tüketici K. + diyetisyen meslek mevzuatı)
4. **Çocuk içerik**:
   - "Bebek maması", "0-1 yaş tarifi" → ek dikkat, sağlık çalışanı onaylı olmadan tavsiye edilmemeli

### Risk — Wellness/Spiritüel (chakra, meditation, breathwork sites)

1. **Tıbbi tavsiye sınırı**:
   - Chakra/meditasyon platformu "anksiyeteyi iyileştirir", "depresyona iyi gelir" gibi ifadeler kullanamaz
   - Türk Tabipleri Birliği Reklam Yönetmeliği + Sağlık Bakanlığı reklam denetimi
   - Doğru ifade: "rahatlama tekniği", "farkındalık pratiği" — etki iddiası YOK
2. **18+ ve gebe uyarısı**:
   - Bazı nefes teknikleri (kapalbhati, holotropik) gebeler/kalp hastaları için kontrendike
   - Wellness disclaimer'ında belirt
3. **Çocuk + ergen**:
   - 16 yaş altı için meditasyon/breathwork → veli onayı önerilir

### Risk — Fitness/Beslenme

1. **Bireye özel program yasak**:
   - "Senin kilon için bu kalori" gibi kişiselleştirilmiş tavsiye için diyetisyen lisansı şart
   - Generic content ("dengeli beslenme nedir") serbest
2. **Sertifika iddiası**:
   - "Diyetisyenler tarafından onaylı" → gerçek diyetisyen + lisans no. eklenmeli

### Risk — Sağlık iddiası (🔴 ağır)

Eğer site teşhis, tedavi, ilaç önerisi, mucize iyileşme iddiası içeriyorsa:
- **6502 sayılı Tüketici K.** — yanıltıcı reklam → para cezası
- **1262 sayılı Kanun** (İspençiyari ve Tıbbi Müstahzarlar) — onaysız tıbbi ürün tanıtımı yasak
- **Sağlık Bakanlığı Reklam Yön.** — sağlık beyanı için Bakanlık onayı zorunlu
- → **Bu skill kapsamı DIŞI** — avukat onayı olmadan deploy etme

### Zorunlu yapı taşları (alt-türe göre)

| Yapı | Gıda UGC | Wellness | Fitness | Sağlık iddiası |
|---|---|---|---|---|
| Genel disclaimer (Kullanım Koşullarında) | ✓ | ✓ | ✓ | ✓ |
| Alerjen sorumluluk maddesi | ✓ | — | — | — |
| "Tıbbi tavsiye değildir" görünür | önerilir | ✓ | ✓ | ✓ |
| Gebe/çocuk kontrendikasyon | — | ✓ | ✓ | ✓ |
| Sertifikalı uzman atfı (varsa) | — | önerilir | ✓ | ✓ |
| 🚨 Avukat review (deploy öncesi) | önerilir | önerilir | ✓ | **zorunlu** |

### Mitigasyon
- Wellness/sağlık iddialı projelerde her tarif/içerik sayfası altında "Bu bir sağlık tavsiyesi değildir" footer notu
- Kullanım Koşullarına gıda/sağlık alt-bölümü ekle (kullanıcı sorumluluğu, indemnity)
- Bireye özel beslenme/sağlık tavsiyesi yasağı belirt
- Sağlık iddiası tespit edilirse → **kullanıcıya açık uyarı**: "bu proje skill kapsamı dışı, avukat görüşmesi al"

---

## 13. AI / GenAI Özel Yükümlülükler (v0.3 yeni)

Yapay zeka ve üretken AI (GenAI) içeren projelerde — chatbot, AI yazı/görsel/ses üreteci, AI yorum/öneri sistemi, AI-powered analitik — klasik yasal yüzeyleri aşan **AI-spesifik** yükümlülükler var:
- **EU AI Act (Regulation 2024/1689)**: 2024 yürürlüğe girdi; risk-tier sınıflandırma + transparency obligations + GPAI rules
- **KVKK md.5/2-d**: Otomatik karar verme ile özel nitelikli veri işleme (yasak haller dışında açık rıza)
- **GDPR Art.22**: Otomatik karar verme + profillemeye karşı veri sahibi hakları
- **Türkiye Yapay Zeka Strateji Belgesi**: 2024+ taslaklarda transparency obligations

### Tarama — AI/GenAI tespit

**A. AI SDK/API kullanımı**:
```bash
# Foundation model API'leri
grep -rn "@anthropic-ai/sdk\|@openai/\|openai\b\|@google/generative-ai\|@google-ai/generativelanguage\|@mistralai/\|cohere-ai\|@huggingface/" \
  --include="package.json" --include="*.ts" --include="*.tsx" --include="*.py" --include="requirements.txt"

# Vercel AI SDK
grep -rn "ai/openai\|ai/anthropic\|@vercel/ai\|^ai$" --include="package.json"

# LangChain/LlamaIndex/orchestration
grep -rn "langchain\|llama-index\|crewai\|@langchain\|autogen" --include="package.json" --include="requirements.txt"

# Self-hosted inference
grep -rn "ollama\|llama-cpp\|transformers\|@xenova/transformers" --include="*.ts" --include="*.py" --include="package.json"
```

**B. AI çıktısı user-facing mi?**:
```bash
# AI generated content → kullanıcıya sunuluyor mu?
grep -rn -iE "(generate|generated|create|completion|chat|interpret|yorum|üret|tahmin|recommendation)" \
  --include="*.tsx" --include="*.ts" 2>/dev/null | grep -iE "(claude|openai|anthropic|gpt|gemini|llm|ai)" | head -10

# Streaming output (kullanıcının doğrudan gördüğü AI çıktısı)
grep -rn "StreamingTextResponse\|useChat\|useCompletion\|stream\b.*completion\|streamText" \
  --include="*.tsx" --include="*.ts" | head -10
```

**C. Otomatik karar verme/profilleme**:
```bash
# Kullanıcı segmentasyonu, scoring, profiling
grep -rn -iE "(score|segment|cohort|category|classify|recommend|target.*user|persona|cluster)" \
  --include="*.ts" --include="*.py" | head -10

# Embedding/vector tabanlı kullanıcı modeli
grep -rn "embedding\|vector.*user\|cosine.*similarity" --include="*.ts" --include="*.py" | head -5
```

**D. Training data / fine-tuning sinyalleri**:
```bash
# Custom fine-tuning veya RAG (user data → AI feedback loop)
grep -rn -iE "(fine.?tun|train.*data|dataset|embedding.*index|vector.*store|pinecone|weaviate|chroma|qdrant)" \
  --include="*.ts" --include="*.py" --include="package.json" | head -10
```

### Karar — AI alt-türü

| Tespit | Alt-tür | AI Act risk tier |
|---|---|---|
| Foundation model API + user-facing chatbot | **AI assistant / chatbot** | Limited risk (transparency) |
| AI yazı/görsel/ses üretimi user-facing | **GenAI content tool** | Limited risk + Art. 50 marking |
| Kullanıcı puanlama/scoring/segmentasyon | **Automated decision-making** | High risk (Annex III) |
| Tıbbi/eğitim/iş/kredi karar AI | **High-risk AI system** | High risk (Annex III) — full obligations |
| Çocuk üzerinde kişiselleştirme | **Vulnerable groups** | High risk + minor data |
| GPAI / open-source genel AI | **General-purpose AI** | GPAI rules (Art. 53-55) |

Detaylı tier rehberi: `reference/ai-act-tiers.md`

### Risk

#### 🔴 Eksik output disclaimer (AI çıktısı kullanıcıya gidiyor)
- AI çıktısının **AI tarafından üretildiği** belirtilmemesi → EU AI Act Art. 50 ihlali (limited risk için bile)
- "Bu garanti edilmiş bilgi" izlenimi → 6502 sayılı Tüketici K. yanıltıcı ticari uygulama
- Halüsinasyon riski açıklanmamış → sağlık/finans/hukuk alanında **çok ağır**

#### 🔴 Bias disclosure yok
- Modelin **eğitim verisi** kaynağı belirsiz → şeffaflık ihlali
- Bias kategorileri (gender, age, geographic, cultural) açıklanmamış
- Bireye özel sonuç verirken bias yansıma uyarısı yok

#### 🔴 Otomatik karar verme — Art. 22 (GDPR) / md. 5/2-d (KVKK)
- Kullanıcı **otomatik** karar veren AI sistemine maruz kaldığını bilmiyor → veri sahibi hakkı ihlali
- "Anlamlı insan müdahalesi" yok → otomatik karara itiraz mekanizması eksik
- Profilleme + sonuç birey üzerinde **hukuki/önemli etki** yapıyor → ek yükümlülükler

#### 🟡 Model card / data sheet yok
- Hangi model (versiyon dahil), nasıl eğitildi, hangi metric'lerle değerlendirildi belirsiz → şeffaflık eksiği
- Audit yapılamıyor, üçüncü taraf inceleyemiyor

#### 🟡 Training data lineage yok
- User input → AI training (RAG embedding store dahil) → kullanıcının verisi modele "kaçıyor mu"?
- KVKK md. 5/1 — açık rıza ile uyumsuz olabilir (rıza alındıktan sonra training kullanımı belirtilmediyse)

#### 🟡 Hassas alan + AI = profesyonel sınır
- Tıbbi teşhis/tedavi AI + lisanslı hekim onayı yok → 1262 sayılı K. + Sağlık Bakanlığı reklam yön. (önceki Yüzey 12 ile bağlantılı)
- Finans tavsiyesi AI + SPK lisansı yok → finansal düzenleme ihlali
- Hukuki tavsiye AI + Avukatlık K. md. 35 → meslek tekeli ihlali

### Mitigasyon — Zorunlu yapı taşları

| Yapı | Kontrol | Yoksa risk |
|---|---|---|
| **AI output disclaimer** (görünür, her sayfada AI içerik varsa) | "AI tarafından üretildi" ifadesi açıkça gösterilir mi | 🔴 |
| **Model card** (kullanılan modeller + versiyonlar) | `/model-card` veya `/ai-info` sayfası var mı | 🟡 |
| **Bias statement** (sınırlamalar) | Aydınlatma metnine AI-spesifik bölüm eklenmiş mi | 🟡 |
| **Otomatik karar verme bildirim** (Art. 22 / md. 5/2-d) | Kullanıcı "AI karar veriyor" bilgisini alıyor mu | 🔴 (varsa) |
| **İtiraz mekanizması** (insan inceleme) | Kullanıcı AI kararına itiraz edebilir mi | 🔴 (varsa) |
| **Training data lineage** | Kullanıcı input'unun training'de kullanılmadığı veya rıza ile kullanıldığı | 🟡 |
| **Hata raporlama** (AI çıktısı yanlış → bildir) | Kullanıcı hatalı çıktıyı raporlayabilir mi | 🟢 |
| **Vendor DPA** (Anthropic/OpenAI/vs.) | Veri işleyen sözleşmesi imzalanmış mı | 🟡 |

### Mitigasyon — Şablonlar (v0.3)

- `/legal-lens generate ai` → şu paketi üretir:
  - `templates/ai-output-disclaimer.md` → görünür AI çıktı işareti
  - `templates/model-card.md` → `/model-card` sayfası
  - `templates/ai-bias-statement.md` → aydınlatma metnine AI bölümü
  - `templates/ai-data-lineage.md` → training/RAG kullanımı disclosure

### Senin projelerine özel notlar

- **VAL-NSPE LM**: Bias-auditing wrapper olarak çalışıyor → kendisi **şeffaflık aracı**; ama yine de paper'da model card + data sheet beklenir
- **Yıldızname**: Symbolic/falcılık AI → "kehanet değildir" + "AI tarafından üretildi" + 18+ kontrolü (zaten var)
- **TargetMind AI**: Game customer targeting → **otomatik karar verme + bias-aware** → Art. 22 ek yükümlülük
- **Philosophy Research**: AI yorumlama → AI generated content marking + bias statement
- **Bloom Within**: Chakra interpretation → "tıbbi tavsiye değildir" + AI generated marking

---

## Raporlama formatı

Her bulgu için ortak yapı:

```
🟡 <Yüzey adı>
**Konum**: `path/to/file.tsx:42-48`
**Bulgu**: <ne tespit ettin, 1-2 cümle>
**Risk**: <hangi yasa/madde/idari para cezası riski>
**Mitigasyon**:
1. <somut komut veya eylem>
2. <somut komut veya eylem>
```

---

*Bu checklist hukuki tavsiye yerine geçmez. KVKK ve GDPR yorumları zaman içinde değişebilir, en güncel için resmi kaynakları kontrol edin: [KVKK Kurumu](https://www.kvkk.gov.tr), [EDPB Guidelines](https://edpb.europa.eu).*
