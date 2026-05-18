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

## 11. E-Ticaret / Ödeme

### Tarama
```bash
grep -rn "stripe\|iyzico\|paytr\|paddle\|@paypal" --include="*.tsx" --include="*.ts" --include="package.json"
```

### Risk
Ödeme alınıyorsa Türkiye'de:
- **ETBİS kaydı** (Elektronik Ticaret Bilgi Sistemi)
- **Mesafeli Satış Sözleşmesi**
- **Cayma Hakkı** (14 gün)
- **KDV mükellefiyeti**
- **Tüketici Hakem Heyeti** bilgisi
- **İade/iptal politikası**

### Mitigasyon
- Bu skill'in kapsamı dışı, faz 3'te eklenebilir
- Şimdilik: "e-ticaret tespit edildi → avukatla görüş" flag'i

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
