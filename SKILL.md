---
name: legal-lens
description: Mevcut projeyi (Next.js/Vite/Flask/herhangi bir web kodtabanı) hukuki uyum riskleri açısından tarar — KVKK/GDPR yükümlülükleri, eksik aydınlatma metni, açık rıza yokluğu, PII toplayan formlar, üçüncü taraf veri aktarımı, riskli iddialar, eksik footer linkleri. Tarama raporu üretir veya istenirse şablon yasal sayfaları üretir.
allowed-tools: Bash Grep Read Write Edit Glob
disable-model-invocation: true
---

# Legal Lens — Kodtabanı Hukuki Risk Filtresi

> ⚠️ **Bu araç hukuki tavsiye vermez**. KVKK/GDPR şablonlarına dayanan yüzey-seviye bir tarama + boilerplate üreticidir. Üretilen sayfalar canlıya çıkmadan önce lisanslı bir avukatla incelenmelidir. Aracın yanlış/eksik çıktısından kaynaklanan zararlardan yazar sorumlu değildir.

## Çalışma modu

Komut: `/legal-lens` veya `/legal-lens <mod>`

Modlar:
- **(boş veya `audit`)** — Mevcut projeyi tara, `LEGAL_AUDIT.md` raporu üret
- **`generate kvkk`** — `/aydinlatma` ve `/kosullar` sayfalarını + footer'ı + consent checkbox'ı oluştur (Türkçe)
- **`generate gdpr`** — `/privacy` ve `/terms` sayfalarını + footer + consent (İngilizce, Faz 2'de geldi)
- **`generate ai`** — AI projeleri için: model card + output disclaimer + bias statement + data lineage (v0.3'te geldi)
- **`generate ecommerce`** — E-ticaret için: mesafeli satış + cayma hakkı + iade-iptal + ETBİS notu (v0.3'te geldi)
- **`consent`** — Form için açık rıza checkbox snippet'i ekle
- **`footer`** — Yasal linkleri içeren footer component'i ekle
- **`cookies`** — Cookie banner + çerez politikası + ConsentGate altyapısını kur
- **`rights`** — KVKK md. 11 hakları için `/hesabim` paneli + veri export + hesap silme API'leri
- **`check`** — Sadece son değiştirilen dosyaları/git diff'i tara (hızlı pre-commit kontrolü)

`$ARGUMENTS` üzerinden mod al. Mod tanınmazsa kullanıcıya yukarıdaki listeyi göster ve hangisini istediğini sor.

---

## Audit modu (default)

### Adım 1 — Bağlam tespiti

Mevcut çalışma dizinini tara, şunları belirle:

1. **Proje türü**:
   - `package.json` → Next.js / React / Vue / Vite / Express / Node
   - `requirements.txt` veya `pyproject.toml` → Flask / FastAPI / Django / CLI tool
   - `Cargo.toml` → Rust
   - `go.mod` → Go

2. **Jurisdiction tespiti — hibrit** (sadece dil bakma, operatör konumu da kritik):

   **Sinyal A — İçerik dili / hedef kitle**:
   - HTML `<html lang="...">` etiketi
   - `app/layout.tsx`'te metadata (locale, lang)
   - Public sayfa içerikleri: 2-3 paragraf oku, ana dil ne?
   - Mevcut yasal sayfalar (`/aydinlatma` vs `/privacy`) hangi dilde?
   - Para birimi (TRY → TR kitle, EUR → EU kitle, USD → US/global)

   **Sinyal B — Operatör (veri sorumlusu) konumu** — bu sinyali ATLAMA:
   - `git config user.email` → domain TLD (örn. @gmail.com nötr, @firma.com.tr → TR)
   - `package.json` `author` alanı
   - Mevcut aydınlatma metninde `{{VERI_SORUMLUSU_AD}}` veya gerçek isim/şehir
   - **Belirsizse kullanıcıya tek soru sor**: "Veri sorumlusu (sen veya şirketin) hangi ülkede ikamet ediyor?" — bu cevap raporda saklanır

   **Sinyal C — Domain / hosting**:
   - Domain TLD (.com.tr → TR; .de/.fr/.eu → EU; .com nötr)
   - Hosting bölgesi (Vercel/Supabase region — `vercel project ls` ya da Supabase Project Settings'ten)

   **Karar matrisi** (sadece dil değil, operatör + dil + kitle):

   | Operatör | Kitle/Dil | Sonuç |
   |---|---|---|
   | Türkiye | TR | **KVKK birincil** (GDPR'a gerek yok — operatör + kitle aynı ülke) |
   | Türkiye | EN/global | **KVKK birincil + GDPR ek** (operatör KVKK'ya tabi; EU kullanıcısı erişiyorsa GDPR de) |
   | Türkiye | EU dilleri | **KVKK + GDPR ikisi de** |
   | EU | TR | **GDPR birincil + KVKK ek** (TR kullanıcı erişiyorsa) |
   | EU | EU/EN | **GDPR birincil** |
   | Belirsiz | TR | KVKK uygula (en yakın) |
   | Belirsiz | EN/global | **KVKK + GDPR ikisini de uygula** (en sıkı tarafa düş) |

   **Önemli**: KVKK md. 2 — Türkiye'de ikamet eden gerçek/tüzel kişi veri sorumlusu ise hangi dilde site olduğu önemsiz, KVKK uygulanır. Bu yüzden Sinyal B en ağır sinyal.

3. **Deploy durumu** (cross-check — tek sinyale güvenme):
   - **A. Repo işareti**: `.vercel/project.json`, `netlify.toml`, `Procfile`, `wrangler.toml`, `app.yaml` var mı?
   - **B. CLI doğrulaması** (sadece A bir şey buldu veya kullanıcı "deploy ettim" dediyse):
     - Vercel için: `vercel project ls 2>&1 | head -10` — proje gerçekten remote'ta mı?
     - Hata "Not authenticated" verirse: kullanıcıya not düş, "manuel doğrulayın" diye işaretle, rapora yaz
     - Hata "No projects" verirse: lokal `.vercel/project.json` stale olabilir, kullanıcıya sor
   - **C. Production URL probe** (B başarılıysa):
     - `vercel inspect <project> 2>&1` veya `.vercel/project.json`'daki projectId'den prod URL çıkar
     - `curl -sI <url>` → 200/301/302 mi yoksa 401 Vercel Auth mi (deploy protection)?
     - 401 dönüyorsa "deploy korumalı, public erişim yok" notu — bu KVKK risk seviyesini düşürür çünkü halka açık değil
   - **D. Git remote** (deploy yoksa bile paylaşılan repo riski):
     - `git remote -v` → public GitHub/GitLab repo mu?
     - Public repo + `.env*` izlenmiyor mu? (`git ls-files .env* 2>/dev/null`)
   - **Karar matrisi**:
     - A+B+C 200 → "halka açık deploy" (en yüksek risk)
     - A+B+C 401 → "deploy var ama korumalı" (orta risk, raporda belirt)
     - A var, B doğrulanamadı → "deploy iddiası, doğrulanamadı" (kullanıcıya sor)
     - Hiçbiri yok → "sadece yerel"

### Adım 2 — Risk yüzeyi taraması

`reference/risk-checklist.md` dosyasını oku ve oradaki canonical kontrol listesini uygula.

Her bulgu için kaydet:
- **Yüzey adı** (örn. "PII toplama", "Üçüncü taraf veri aktarımı")
- **Yüzey numarası** (1-13, risk-weights.md'den weight için)
- **Konum** (`file:line` formatında, tıklanabilir)
- **Bulgu** (ne tespit ettin, kısa)
- **Risk seviyesi**: 🟢 Düşük / 🟡 Orta / 🔴 Yüksek
- **Mitigasyon** (somut, eyleme dönük: "şu komutu çalıştır", "şu dosyayı oluştur")

### Adım 2.5 — Skor hesabı (v0.3+)

`reference/risk-weights.md` dosyasını oku, skor hesaplama:

```
severity_score = 1 (🟢) | 4 (🟡) | 10 (🔴)
finding_deduction = severity_score × surface_weight (1-3, risk-weights.md tablosundan)
total_deduction = Σ(all findings)
final_score = max(0, 100 - total_deduction)
```

**Modifier'ları uygula** (risk-weights.md'deki ek düşüş tablosu):
- Halka açık deploy + PII + eksik yasal sayfa: -10
- Public git repo + .env izleniyor: -15
- Çocuk verisi + 18+ kontrolü yok: -10
- Sağlık iddiası + avukat review yapılmamış: cap 50/100
- AI çıktısı + output disclaimer yok: -5 (v0.3 yüzey 13)

**Skor trendi** (önceki audit varsa):
- Repo root'ta `LEGAL_AUDIT.md` zaten varsa, "## Özet" bölümünden önceki skoru oku
- Bu run'un skoruyla karşılaştır, delta hesapla
- Trend bölümünde göster: "Δ son audit: +X / -X puan"

### Adım 3 — Rapor üret

Proje root'una `LEGAL_AUDIT.md` yaz. Format:

```markdown
# Legal Audit — <proje adı>

> Tarama tarihi: <ISO date>
> Bu rapor hukuki tavsiye değildir. Detay için skill'in README.md'sine bak.

## Özet

- **🎯 Uyum Skoru**: <N>/100 <emoji> *(90-100 🟢 düzgün · 70-89 🟡 orta · 50-69 🟠 zayıf · 0-49 🔴 kritik)*
- **Proje türü**: ...
- **Jurisdiction tespiti**: KVKK / GDPR / Her ikisi
- **Deploy**: Halka açık (<URL>) / Sadece yerel
- **Toplam yüzey**: <n>
- **🔴 Yüksek risk**: <n> | 🟡 Orta: <n> | 🟢 Düşük: <n>
- **Yüksek ağırlık alanlar** (weight 3): PII, 3rd-party, eksik yasal sayfa, ...

### Skor Trendi (önceki audit varsa)

- <prev_date>: <prev_score>/100 <prev_emoji>
- <this_date>: <this_score>/100 <this_emoji> ← bu run
- Δ: +<X> / -<X> puan (<kısa açıklama: hangi yüzeyler değişti>)

## Acil Eylemler

(🔴 ve 🟡 olan tüm bulguların özetlenmiş eylem listesi, somut komutlarla)

## Detaylı Bulgular

### 🔴 <Yüzey adı> [Yüzey #1, weight 3]
- **Konum**: `app/page.tsx:124`
- **Bulgu**: Email + isim toplayan form, hiç açık rıza yok
- **Risk**: KVKK md. 5/1 ihlali; idari para cezası riski
- **Skor etkisi**: -30 (severity 10 × weight 3)
- **Mitigasyon**:
  1. `/legal-lens generate kvkk` çalıştır → /aydinlatma sayfası oluştur
  2. Form öncesi consent checkbox ekle (`/legal-lens consent`)
  3. Footer'a yasal link ekle (`/legal-lens footer`)

(Tüm bulgular bu formatta — `[Yüzey #N, weight X]` ve `**Skor etkisi**` zorunlu)

## Tarama Dışı Bırakılanlar

(Taranamadı / belirsiz olan şeyler — örn. "Cookie kullanımı belirsiz, dev tools'da Application > Cookies kontrol et")

## Sonraki Adımlar

- En riskli 3 bulguya odaklan
- Şablon üret: `/legal-lens generate kvkk`
- Profesyonel review için: ticari satış, 100+ günlük kullanıcı veya yüksek-risk içerik alanına geçişte avukatla 1-2 saatlik review yaptır

---

*Legal Lens v0.3 · Bu rapor hukuki tavsiye değildir.*
```

### Adım 4 — Kullanıcıya özet sun

Raporu yazdıktan sonra terminalde kısa bir özet ver:
- **🎯 Uyum Skoru: <N>/100 <emoji>** (öne çıkar)
- En kritik 3 bulgu
- Tahmini düzeltme süresi
- Hangi `/legal-lens generate ...` komutuyla otomatik şablon üretebileceği
- Önceki audit varsa: skor trendi (delta)

---

## Generate modu (`kvkk`, `gdpr`)

### `generate kvkk`

1. `templates/kvkk-aydinlatma.md` ve `templates/kullanim-kosullari.md` dosyalarını oku
2. Projenin türüne göre yerleştir:
   - **Next.js App Router**: `app/aydinlatma/page.tsx`, `app/kosullar/page.tsx`
   - **Next.js Pages Router**: `pages/aydinlatma.tsx`, `pages/kosullar.tsx`
   - **Vite/SPA**: `src/pages/Aydinlatma.tsx` veya benzeri
   - **Flask**: `templates/aydinlatma.html`
3. Template placeholder'ları doldur:
   - `{{VERI_SORUMLUSU_AD}}` — kullanıcıya sor (gerçek kişi adı / ticari unvan)
   - `{{ILETISIM_EMAIL}}` — kullanıcıya sor
   - `{{TOPLANAN_VERILER}}` — kodtabanından otomatik tespit ettiklerin
   - `{{ISLEME_AMACI}}` — kullanıcıya sor veya proje türünden tahmin et
   - `{{UCUNCU_TARAF_LISTE}}` — package.json'dan tespit ettiklerin (Anthropic, Supabase, Stripe, vs.)
   - `{{YURT_DISI_AKTARIM_VAR_MI}}` — üçüncü taraf listesinden çıkar
   - `{{SAKLAMA_SURESI}}` — varsayılan "saklanmaz" veya kullanıcıya sor
4. Footer component'ini oluştur (`/legal-lens footer` alt-adımını çalıştır)
5. Mevcut formlara consent checkbox ekle (`/legal-lens consent` alt-adımını çalıştır)
6. Kullanıcıya değiştirilen/oluşturulan dosyaların özetini ver

### `generate gdpr`

İngilizce GDPR şablonlarını kullan. Mantık `generate kvkk` ile aynı, ama farklı template'ler ve farklı route'lar.

**Üretilen dosyalar**:

1. `templates/privacy-policy.md` → `app/privacy/page.tsx` (GDPR Art. 13/14 information notice)
2. `templates/terms-of-service.md` → `app/terms/page.tsx`
3. `templates/footer-en.md` → `components/Footer.tsx` (zaten varsa link'leri günceller)
4. `templates/consent-checkbox-en.md` → mevcut formlara çoğul yerlere snippet ekle
5. Tracking SDK varsa → `templates/cookie-banner-en.md` + `templates/cookie-policy.md` üret
6. Auth+DB varsa → `templates/gdpr-rights.md` üret (`app/account/page.tsx` + export + delete API'leri)

**Placeholder doldurma — KVKK varyantından farklar**:

KVKK templates'inde `{{VERI_SORUMLUSU_AD}}`, `{{ILETISIM_EMAIL}}` gibi placeholder'lar Türkçe yazılmıştı. GDPR templates'inde İngilizce karşılıkları kullanılır:

| KVKK placeholder | GDPR equivalent | Kaynak |
|---|---|---|
| `{{PROJE_ADI}}` | `{{PROJECT_NAME}}` | proje adı |
| `{{VERI_SORUMLUSU_AD}}` | `{{DATA_CONTROLLER_NAME}}` | kullanıcıya sor |
| — | `{{DATA_CONTROLLER_LOCATION}}` | kullanıcıya sor (örn. "Istanbul, Turkey") |
| `{{ILETISIM_EMAIL}}` | `{{CONTACT_EMAIL}}` | kullanıcıya sor |
| `{{TOPLANAN_VERILER_LISTE}}` | `{{COLLECTED_DATA_LIST}}` | kodtabanından tespit |
| `{{ISLEME_AMACI_LISTE}}` | `{{PROCESSING_PURPOSE_LIST}}` | proje türünden tahmin / kullanıcıya sor |
| `{{UCUNCU_TARAF_BLOK}}` | `{{THIRD_PARTY_BLOCK}}` | package.json + servis tespiti |
| `{{SAKLAMA_DURUMU}}` | `{{RETENTION_STATEMENT}}` | DB tespiti + kullanıcıya sor |
| `{{CEREZ_DURUMU}}` | `{{COOKIE_STATEMENT}}` | cookie tarama sonucu |
| — | `{{LAWFUL_BASIS}}` | Art. 6 — consent/contract/legitimate interests; proje türünden tahmin |
| — | `{{TRANSFER_BLOCK}}` | 3rd party tespiti — EEA dışına aktarım var mı |
| — | `{{SUPERVISORY_AUTHORITY_BLOCK}}` | jurisdiction tespiti — sadece EU mu, EU+TR mi |
| `{{TARIH}}` | `{{DATE}}` | bugünün tarihi (ISO) |

**Jurisdiction kararı çıktıyı etkiler**:
- **GDPR birincil + KVKK ek** (TR operatör, EU kitle) → privacy/terms EN üret + footer'da TR aydınlatma'ya cross-link bırak
- **GDPR birincil** (EU operatör + kitle) → sadece EN
- **KVKK birincil + GDPR ek** (TR operatör + kitle, ama EU kullanıcısı erişiyor) → önce `generate kvkk` yap, sonra `generate gdpr` paralel EN sayfalar ekle

**Çoklu jurisdiction senaryosu**:
- TR + EN parallel pages: `/aydinlatma` (TR) + `/privacy` (EN), `/kosullar` (TR) + `/terms` (EN)
- Her sayfanın altında diğer dile cross-link
- Cookie banner i18n: tek banner + locale-based strings (`cookie-banner-en.md`'deki strings tablosu yardımcı)

**Ön kontroller** (üretmeden ÖNCE):
1. Mevcut `/privacy`, `/terms` var mı? → Varsa kullanıcıya sor, üzerine yazma
2. Operatör konumu + iletişim email kullanıcıdan al (sadece 1 kez sor)
3. Lawful basis için kullanıcıya 3 seçenek sun: "form-based consent only", "service contract + consent", "legitimate interests for security + consent for content"

**Üretim sonrası**:
1. Test checklist sun:
   - [ ] `/privacy` ve `/terms` 200 dönüyor mu
   - [ ] Footer'da linkler var mı
   - [ ] Form'larda consent checkbox unchecked default mu (Art. 7 — pre-ticked yasak)
   - [ ] Service role key client'a sızmıyor mu (auth varsa)
2. KVKK varsa cross-link doğrula
3. Production'a deploy etmeden önce avukat review öner (özellikle ticari/ödeme/sağlık ise)

---

## Consent modu

Mevcut form'lara açık rıza checkbox ekle:

1. Form bul: `grep -rn "<form\|onSubmit" --include="*.tsx" --include="*.jsx"`
2. Her form için, submit butonundan önce checkbox HTML'i ekle (`templates/consent-checkbox.tsx` referans al)
3. State management ekle (`rizaVerildi` state, `onChange` handler, submit'i disable et)
4. Kullanıcıya hangi form'ları değiştirdiğini söyle, manuel review iste

---

## Cookies modu

Cookie banner + çerez politikası + onaylı script yükleme altyapısını kur.

### Adım 1 — Tarama

Kullanıcıya yüklemeden ÖNCE projedeki çerez/tracking yüzeyini tespit et:

```bash
# Analytics SDK'ları
grep -rn "@vercel/analytics\|@vercel/speed-insights\|google-analytics\|gtag\|fbq\|posthog\|mixpanel\|@segment" \
  --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js" --include="package.json"

# document.cookie kullanımı
grep -rn "document\.cookie\|cookies()\|setCookie" --include="*.tsx" --include="*.ts"

# 3rd-party iframe (YouTube, Twitter)
grep -rn "youtube\.com\|twitter\.com/widgets\|platform\.twitter\.com" --include="*.tsx" --include="*.jsx"

# localStorage tracking ipuçları
grep -rn "localStorage\.setItem" --include="*.tsx" --include="*.ts" | head -10
```

### Adım 2 — Karar

Tarama sonucuna göre kullanıcıya rapor ver, ÜRETMEDEN ÖNCE onay al:

**Eğer hiç tracking/marketing çerezi yoksa**:
- Banner gerekmez
- Sadece aydınlatma metnine "çerez kullanılmaz" notunu ekle
- Kullanıcıya "Cookie banner gerekli değil çünkü tracking SDK tespit edilmedi" raporu sun

**Eğer tracking SDK tespit edildiyse**:
- Banner + politika + ConsentGate gerekli
- Tespit edilen SDK'ları listele
- Onayla, üretmeye geç

### Adım 3 — Üretim

`templates/cookie-banner.md` ve `templates/cerez-politikasi.md` şablonlarını oku ve şu dosyaları üret:

1. `lib/cookieConsent.ts` — tip + yardımcı fonksiyonlar (loadConsent, saveConsent, acceptAll, rejectAll)
2. `lib/useCookieConsent.ts` — kategori bazlı hook (overload: tüm state veya tek kategori)
3. `components/CookieSettings.tsx` — kategori toggle paneli
4. `components/CookieBanner.tsx` — banner (kompakt + genişletilmiş görünüm)
5. `components/ConsentGate.tsx` — kategori bazlı wrapper (`requires="analytics"` veya `requires={["analytics","marketing"]}`)
6. `app/cerez-politikasi/page.tsx` — politika sayfası
7. `app/cerez-ayarlari/page.tsx` — sonradan tercih değiştirme sayfası
8. `app/globals.css`'e CSS ekle (banner + toggle stilleri)
9. `app/layout.tsx`'e `<CookieBanner />` ekle
10. Footer'a `/cerez-politikasi` ve `/cerez-ayarlari` linklerini ekle (mevcut `Footer.tsx`'i güncelle)

**Kategori seçimi**: 4 default kategori var (`necessary`, `functional`, `analytics`, `marketing`). Tarama sonucuna göre sadece **kullanılan** kategorileri göster — proje sadece analytics kullanıyorsa `marketing` toggle'ını çıkar. CookieSettings'teki `KATEGORILER` listesini bu tespitlere göre filtreleyerek üret.

Şablon placeholder'larını doldur:
- `{{PROJE_ADI}}` — proje adı
- `{{POLITIKA_URL}}` — `/cerez-politikasi`
- `{{ILETISIM_EMAIL}}` — proje email'i
- `{{CEREZ_LISTESI}}` — Adım 1'de tespit edilen SDK'lara göre tablo doldur
- `{{ANALITIK_BLOK}}` — her SDK için ilgili paragraf

### Adım 4 — Mevcut analytics'i sar

Tespit edilen analytics/tracking import'larını uygun kategoriye göre `<ConsentGate>` ile sar:

| SDK | requires kategorisi |
|---|---|
| `@vercel/analytics`, `@vercel/speed-insights` | `analytics` |
| `google-analytics`, `gtag` | `analytics` |
| `posthog-js`, `mixpanel`, `@segment` | `analytics` |
| `fbq` (Facebook Pixel) | `marketing` |
| `linkedin/insight`, `tiktok pixel` | `marketing` |
| `hotjar`, `fullstory`, `clarity` | `analytics` |
| YouTube/Twitter embed | `functional` |

**Önce**:
```tsx
import { Analytics } from "@vercel/analytics/react";
<Analytics />
```

**Sonra**:
```tsx
import { Analytics } from "@vercel/analytics/react";
import ConsentGate from "@/components/ConsentGate";
<ConsentGate requires="analytics">
  <Analytics />
</ConsentGate>
```

Bu dönüşümü her tespit edilen analytics/tracking için yap. Bir component birden fazla SDK barındırıyorsa (örn. hem GA hem FB Pixel) `requires={["analytics","marketing"]}` kullan.

### Adım 5 — Aydınlatma metnini güncelle

Eğer `/aydinlatma` zaten varsa, onun 8. bölümünü (Çerezler) güncelle: önceki "çerez kullanılmaz" notu yerine "Çerez Politikası sayfasına bakın → /cerez-politikasi" linki koy.

---

## Footer modu

`templates/footer.tsx` şablonunu projenin convention'ına göre yerleştir:
- Next.js App Router → `components/Footer.tsx` + `app/layout.tsx`'e import
- Pages Router → benzer
- Vite → `src/components/Footer.tsx`

Linkler: `/aydinlatma`, `/kosullar` (KVKK modu) veya `/privacy`, `/terms` (GDPR modu).

CSS dahil etme — proje kendi tasarım sistemiyle entegre edebilsin diye class adlarını minimal tut.

---

## Rights modu — KVKK md. 11 hakları

Kullanıcının kendi verisini görmesi, indirmesi ve silmesi için akış kur. Veri saklayan her uygulamada (özellikle Supabase/Prisma kullanan, kayıt sistemli olanlar) zorunlu.

### Adım 1 — Tarama (önkoşul)

```bash
# Auth sistemi var mı?
grep -rn "supabase\.auth\|next-auth\|clerk\|auth0\|iron-session" \
  --include="*.tsx" --include="*.ts" --include="package.json"

# Veritabanı kütüphanesi
grep -rn "supabase\|prisma\|drizzle\|@vercel/postgres" --include="package.json"

# Mevcut /hesabim, /account, /profile var mı?
ls app/hesabim app/account app/profile pages/hesabim pages/account 2>/dev/null

# Service role key'i?
grep -rn "SUPABASE_SERVICE_ROLE_KEY\|SERVICE_ROLE" --include="*.ts" --include=".env*"
```

### Adım 2 — Önkoşul kontrolü

Veri saklamayan projeler için bu mod **gereksiz** — kullanıcıya bildir ve çık.

Supabase varsa: `SUPABASE_SERVICE_ROLE_KEY`'in env'de olup olmadığını doğrula. Yoksa kullanıcıya nereden alacağını söyle (Supabase dashboard → Project Settings → API).

### Adım 3 — Üretim

`templates/kvkk-md11-rights.md` şablonunu oku ve şunları üret:

1. **`app/hesabim/page.tsx`** — Kullanıcı paneli (görüntüleme + export + silme butonları)
2. **`app/api/me/export/route.ts`** — JSON veri export endpoint'i
3. **`app/api/me/route.ts`** — DELETE endpoint (hesap silme)
4. **`app/globals.css`'e CSS yardımcıları** — `.bilgi-satiri`, `.haklar-buton`, `.tehlike-bolge`, `.silme-onay-kutu`, vs.

Şablon placeholder'ları:
- `{{PROJE_ADI}}`
- `{{ILETISIM_EMAIL}}`
- `{{KULLANICI_TABLOLARI}}` — **schema parse ile otomatik çıkar, kullanıcıya en son sor**:

  **Adım 3a — Schema dosyalarını bul** (sırayla dene, ilkini kullan):
  ```bash
  # Supabase
  ls supabase/migrations/*.sql 2>/dev/null | sort
  # Prisma
  [ -f prisma/schema.prisma ] && echo "prisma"
  # Drizzle
  find . -name "schema.ts" -path "*/db/*" -o -name "schema.ts" -path "*/drizzle/*" 2>/dev/null | head -3
  # Raw SQL
  find . -name "*.sql" -not -path "*/node_modules/*" | head -10
  ```

  **Adım 3b — `user_id` FK olan tabloları çıkar**:

  - **Supabase migrations** (`*.sql`):
    ```bash
    # CREATE TABLE bloklarını bul, içinde user_id REFERENCES auth.users veya FK varsa al
    grep -B 0 -A 30 "CREATE TABLE" supabase/migrations/*.sql | \
      grep -iE "CREATE TABLE|user_id.*REFERENCES|author_id|sender_id|owner_id"
    ```
    Çıktıyı parse et: her `CREATE TABLE public.X` için içeride `user_id|author_id|sender_id|owner_id` referansı varsa **X** kullanıcı verisi tablosudur.

  - **Prisma** (`schema.prisma`):
    ```bash
    grep -B 1 -A 20 "^model " prisma/schema.prisma | grep -iE "^model |userId|authorId|@relation"
    ```
    `@relation(... references: [id])` ile User'a bağlı tüm model'ler.

  - **Drizzle** (`schema.ts`):
    ```bash
    grep -B 0 -A 15 "pgTable\|sqliteTable\|mysqlTable" db/schema.ts | \
      grep -iE "pgTable|userId|references"
    ```

  **Adım 3c — Storage bucket'larını da ekle** (avatar, upload):
  ```bash
  # Supabase storage referansları
  grep -rn "supabase\.storage\.from(" --include="*.ts" --include="*.tsx" 2>/dev/null | \
    grep -oE "from\(['\"]\w+['\"]" | sort -u
  ```

  **Adım 3d — Tablo listesini kullanıcıya ONAYLATTIR** (sormak değil, onaylatmak):
  ```
  Şemadan çıkardığım kullanıcı verisi tabloları:
  - public.profiles (user_id PK)
  - public.experiments (author_id → auth.users)
  - public.messages (sender_id → auth.users)
  - public.follows (follower_id, following_id → auth.users)
  - storage.avatars/{user_id}/*

  Bu listeyi kullanacağım. Eklemek/çıkarmak istediğin var mı? [Enter = onayla, yoksa düzelt]
  ```

  Hiçbir schema dosyası bulunamazsa: **o zaman** kullanıcıya boş soru sor ("hangi tablolar?"). Bulunduysa daima parse-then-confirm akışı.

### Adım 4 — Strateji kararı (kullanıcıya sor)

UGC varsa (post/comment/message), silme stratejisini sor:

- **Tam silme** — kullanıcının her şeyi gider (basit, yıkıcı)
- **Anonimleştirme** — UGC kalır, ama "Silinen Kullanıcı" gösterilir (topluluk değeri korunur)
- **Soft delete** — 30 gün geri alınabilir, sonra hard delete (kullanıcı dostu, env'de cron gerekli)

Seçtikleri stratejiye göre silme API'sini ona göre üret.

### Adım 5 — Aydınlatma metnini güncelle

Eğer `/aydinlatma` zaten varsa, "Veri Sahibi Hakları" bölümünü güncelle: `/hesabim` paneline yönlendir.

### Adım 6 — Kullanıcıya checklist sun

Üretimden sonra mutlaka şu test checklist'i göster:
- Export çalışıyor mu
- Silme akışı "SİL" yazımdan önce disabled mı
- Service role key client'a sızmıyor mu (`grep -r "SUPABASE_SERVICE_ROLE" app/components/ src/components/`)
- Silinen kullanıcının UGC durumu beklendiği gibi mi

---

## Generate AI modu (v0.3)

AI/GenAI içeren projeler için özel paket — model card + output disclaimer + bias statement + data lineage.

### Adım 1 — AI tespit (önkoşul)

Yüzey 13 (AI/GenAI) tespit edilmiş olmalı. Tespit yoksa kullanıcıya bildir ve çık:
```
Bu projede AI/GenAI kullanımı tespit edilmedi. /legal-lens audit çalıştır
ve önce AI yüzeyini doğrula.
```

`reference/risk-checklist.md` Yüzey 13 tespitlerini kullan:
- Foundation model SDK'ları (Anthropic, OpenAI, Google, Mistral, vb.)
- Vercel AI SDK / LangChain / LlamaIndex
- Self-hosted inference (Ollama, transformers, vb.)

### Adım 2 — Sınıflandırma

`reference/ai-act-tiers.md` rehberine göre AI tier'ını belirle:
- **Minimal risk** — yükümlülük yok
- **Limited risk (Art. 50)** — transparency obligations (en yaygın senaryo)
- **High risk (Annex III)** — full obligations + avukat review zorunlu
- **GPAI** — Art. 53 downstream user

Kullanıcıya tespit ettiğin tier'ı söyle ve onay al.

### Adım 3 — Detay sorular

Şablon placeholder'ları için kullanıcıya:

1. **Kullanım amacı** (`{{KULLANIM_AMACI}}` / `{{INTENDED_USE}}`) — bir cümle
2. **AI çıktısı user-facing mi?** (evet → AI disclaimer'lar görünür eklenecek)
3. **Otomatik karar verme var mı?** (kullanıcı segmentasyonu, scoring, vb.)
4. **Fine-tuning yapılıyor mu?** (yes/no/planned)
5. **RAG / vector store kullanıyor mu?** Kullanıcı verisi indeksleniyor mu?
6. **DPA durumu** — kullanılan vendor'larla DPA imzalı mı? (Anthropic/OpenAI/Google: yes by default; diğerleri için sor)

### Adım 4 — Üretim

Tespit edilen tier ve proje özelliklerine göre şu dosyaları üret:

#### Limited risk (en yaygın)

1. **`components/AIDisclaimer.tsx`** — `templates/ai-output-disclaimer.md`'den, görünür AI işareti
2. **`app/model-card/page.tsx`** — `templates/model-card.md`'den, kullanılan modeller + amaç + sınırlamalar
3. **`app/ai-bias/page.tsx`** — `templates/ai-bias-statement.md`'den, bias categories + tasarım yaklaşımı
4. **`app/aydinlatma/page.tsx`** veya `app/privacy/page.tsx`'e **AI bölümü ekle** — `templates/ai-data-lineage.md`'den (mevcut sayfaya **ekleme**, yeni dosya değil)
5. **Footer'a 3 yeni link**: `/model-card`, `/ai-bias`, ai-disclaimer footer variant
6. **AI çıktısı user-facing component'lere `<AIDisclaimer>` ekle**: `grep -rn "useChat\|useCompletion\|streamText" --include="*.tsx"` ile bul, her bir AI çıktısı render'ını wrap et

#### High risk (Annex III)

Limited risk paketi + ek:
7. **Conformity assessment notu** — kullanıcıya: "high-risk AI sistemi tespit edildi; CE marking + EU database registration gerekebilir; avukat review zorunlu"
8. **`/ai-human-oversight` sayfası** — insan supervision design'ını açıkla
9. **Logging spec** — Art. 12 record keeping için açıklama (kod örneği değil, gereksinim listesi)
10. **Avukat review uyarısı** — explicit, raporun ana özeti

#### GPAI downstream user

7. **`/ai-info` sayfası** — Hangi GPAI sağlayıcı, hangi versiyon, Art. 53 information'a link

### Adım 5 — Placeholder doldur

Otomatik tespit edilebilenler (kullanıcıdan tekrar sorma):
- `{{MODEL_LISTESI}}` — package.json + .env'den (env variable adlarına bak, örn. ANTHROPIC_API_KEY → Claude)
- `{{PROJE_ADI}}` — package.json `name`
- `{{ILETISIM_EMAIL}}` — mevcut aydınlatma metninden veya kullanıcıya sor (sadece eksikse)
- `{{TARIH}}` — bugünün ISO tarihi
- `{{LANG}}` — jurisdiction tespitine göre

Otomatik tespit edilemeyenler (sormak gerekir):
- `{{KULLANIM_AMACI}}` — bir cümle
- `{{FINE_TUNE_DURUMU}}` — yapılıyor / planlı / yapılmıyor
- `{{RAG_DURUMU}}` — kullanılıyor / kullanılmıyor / minimum
- `{{AI_ACT_TIER}}` — Adım 2'de belirlendi

### Adım 6 — Test checklist

Üretimden sonra:

- [ ] `/model-card` 200 dönüyor mu
- [ ] `/ai-bias` 200 dönüyor mu
- [ ] AI çıktısı render edilen her component'te `<AIDisclaimer>` var mı (grep ile doğrula)
- [ ] Footer'da `/model-card` ve `/ai-bias` link'leri var mı
- [ ] Aydınlatma metni veya privacy policy'de AI bölümü dolu mu
- [ ] High-risk ise: avukat review randevusu/notu var mı

### Adım 7 — Skor değişikliği

Bu üretimden sonra `LEGAL_AUDIT.md`'de Yüzey 13 skoru iyileşmeli:
- Daha önce: 🔴 (örn. -30 puan) → şimdi: 🟢 (örn. -3 puan)
- Skor delta'yı kullanıcıya bildir.

---

## Generate Ecommerce modu (v0.3)

E-ticaret içeren projeler için özel paket — mesafeli satış + cayma hakkı + iade-iptal + ETBİS.

### Adım 1 — E-ticaret tespit (önkoşul)

Ödeme SDK'sı tespit edilmiş olmalı:
```bash
grep -rn "stripe\|iyzico\|paytr\|paddle\|@paypal\|shopier" --include="package.json" --include="*.ts"
```

Tespit yoksa kullanıcıya bildir ve çık.

### Adım 2 — Sorular

Kullanıcıya:

1. **Satış konusu** — fiziksel ürün / dijital ürün / hizmet / üyelik / abonelik
2. **Hedef pazar** — TR / EU / global
3. **Vergi mükellefiyeti** — şahıs şirketi / sermaye şirketi / kayıtsız
4. **Ödeme sağlayıcı** — Stripe (yurt dışı) veya iyzico/PayTR (Türkiye)
5. **Teslimat süresi** — ortalama gün

### Adım 3 — Üretim (TR pazarı)

Şu dosyaları üret:

1. **`app/mesafeli-satis-sozlesmesi/page.tsx`** — `templates/mesafeli-satis.md`'den
2. **`app/cayma-hakki/page.tsx`** — `templates/cayma-hakki.md`'den (14 gün, istisnalar)
3. **`app/iade-iptal/page.tsx`** — `templates/iade-iptal.md`'den (akış + form)
4. **`app/etbis-bilgi/page.tsx`** veya footer notu — `templates/etbis-bilgi-formu.md`'den
5. **Footer'a 4 yeni link**
6. **Checkout flow'da consent**: mesafeli satış kabulü checkbox (pre-checkout)

### Adım 4 — Üretim (EU/global pazar)

EU consumer rights directive uyumlu:

1. `app/consumer-rights/page.tsx` — 14-day withdrawal right (EU eşdeğeri)
2. `app/refund-policy/page.tsx`
3. ETBİS yerine VAT MOSS bilgisi (EU operatörse)
4. Checkout consent

### Adım 5 — ETBİS uyarısı (TR operatör)

Tespit edilen pazara göre kullanıcıya bilgi notu:

```
⚠️ ETBİS Bilgi Sistemi Kaydı
T.C. Ticaret Bakanlığı'nın ETBİS sistemine kayıt zorunludur (e-ticaret faaliyeti yapan tüzel/gerçek kişiler için).

Kayıt adresi: https://etbis.ticaret.gov.tr
Belge: ETBİS Bilgilendirme Formu (üretilen sayfada)
```

### Adım 6 — Vergi sorumluluğu uyarısı

Vergi durumuna göre kullanıcıya bilgi notu (avukat/mali müşavir yönlendirmesi):

- **Şahıs şirketi olmadan satış**: 6502 sayılı K. + mevzuat gereği kayıt dışı satış yasak
- **KDV mükellefiyeti**: dijital ürün/hizmet için ayrı düzenleme (2024+ revize edilen mevzuat)
- **Vergisel yükümlülükler**: bu skill kapsamı **dışı**; mali müşavir/SMMM gerekli

### Adım 7 — Test checklist

- [ ] Tüm 4 sayfa 200 dönüyor mu
- [ ] Checkout'ta mesafeli satış kabul checkbox var mı (pre-checked olmadan)
- [ ] İade-iptal formu çalışıyor mu
- [ ] Footer'da yeni link'ler var mı
- [ ] ETBİS kaydı (TR) veya VAT MOSS (EU) bildirimi var mı
- [ ] Mali müşavir review zorunlu hatırlatması verilmiş mi

---

## Check modu (hızlı tarama)

Sadece git diff'i tara:
```bash
git diff HEAD --name-only | xargs ...
```

Yeni form mı eklendi, yeni third-party SDK mı yüklendi, yeni claim'li metin mi geldi — bunları kontrol et. Pre-commit hook gibi düşün ama manuel çalıştırılan.

---

## Önemli notlar

1. **Hiçbir çıktıda "hukuki tavsiye veriyorum" deme**. Her rapor/üretim sonunda "Bu hukuki tavsiye değildir. Profesyonel review için lisanslı avukatla görüşülmesi önerilir." cümlesi olmalı.

2. **Anthropic AUP & TR Avukatlık Kanunu Md. 35** sınırını koru. Belirli bir yasanın belirli bir maddesinin nasıl uygulanacağına dair **kişiselleştirilmiş yorum yapmaktan kaçın**. Bunun yerine "şu maddeyi kontrol et" şeklinde bilgilendir.

3. **Kullanıcı verisi**: Skill kodtabanını okur ama hiçbir kişisel veriyi/kodu dışarı göndermez (Claude API'ye gönderiliyor olabilir ama o zaten Claude Code'un default davranışı, skill ekstra bir gönderim yapmaz).

4. **Şüphede ise daha sıkı tarafa düş**. "Belki gerek yoktur" demek yerine "şunu kontrol et" de.

5. **Mevcut işi yıkma**. Eğer `/aydinlatma` zaten varsa, üzerine yazmadan önce sor. Eğer footer zaten varsa, üzerine yazma.
