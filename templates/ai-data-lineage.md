---
template: ai-data-lineage
language: tr,en
legal_basis:
  - eu_ai_act_art_10
  - eu_ai_act_art_53
  - kvkk_md_5_1
  - kvkk_md_9
  - gdpr_art_5_1
  - gdpr_art_44
jurisdiction: TR,EU
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "Training data lineage + RAG indexing + fine-tuning — kullanıcı verisinin AI'ya 'sızması' riski"
---

# AI Data Lineage Şablonu

> Kullanıcı verisinin **AI sistemine nasıl aktığı** ve **nerede saklandığı** şeffaflığı. KVKK md. 5/1 + GDPR Art. 5(1)(a) (transparency principle) + AI Act Art. 10 (data governance) ile uyumlu.

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{AI_VERI_AKISI_TABLO}}` — kullanıcı verisinin AI'ya akışı (Claude doldurur)
- `{{FINE_TUNE_DURUMU}}` — kullanılıyor mu?
- `{{RAG_DURUMU}}` — kullanılıyor mu?
- `{{LANG}}`

---

## Aydınlatma metnine eklenecek bölüm (Türkçe)

`/aydinlatma` sayfasındaki ilgili bölümün **yerine** veya **ekine**:

```markdown
## X. Yapay Zeka ve Veri Akışı

### X.1. Verileriniz AI Sistemine Nasıl İletilir?

Kullanıcı girdileriniz (örn. soru/mesaj/yüklediğiniz dosya) {{PROJE_ADI}} sunucularından
şu AI sağlayıcılarına iletilir:

| Veri tipi | Sağlayıcı | Konum | Saklama | Amaç |
|---|---|---|---|---|
| {{ORNEK_VERI_1}} | {{ORNEK_PROVIDER_1}} | {{ORNEK_KONUM_1}} | {{ORNEK_SAKLAMA_1}} | {{ORNEK_AMAC_1}} |
| {{ORNEK_VERI_2}} | {{ORNEK_PROVIDER_2}} | {{ORNEK_KONUM_2}} | {{ORNEK_SAKLAMA_2}} | {{ORNEK_AMAC_2}} |
{/* Claude doldurur — package.json + .env + kullanım analizinden */}

### X.2. Verileriniz AI Modelin Eğitimi İçin Kullanılıyor mu?

**Kısa cevap**: {{TRAINING_USE_SUMMARY}}

Detay:

- **Anthropic Claude API** (commercial agreement): API'ye gönderdiğiniz veri
  Claude'un genel eğitiminde **kullanılmaz**. Anthropic 30 gün boyunca güvenlik/
  abuse detection için saklar, sonra siler.
- **OpenAI API** (default): Mart 2023'ten itibaren API verisi training için
  **kullanılmaz** (opt-out gerekmez, default opt-out'tur). 30 gün saklanır.
- **Google Generative AI** (paid tier): API verisi training için kullanılmaz.
- {{DIGER_PROVIDER_SATIRLARI}}

Önemli: **Free/consumer chatbot** (claude.ai, chatgpt.com, gemini.google.com) ile
API kullanımı **farklıdır** — biz API kullanıyoruz.

### X.3. Fine-tuning / Custom Model Eğitimi

**{{FINE_TUNE_DURUMU}}**

{/* Eğer FINE_TUNE_DURUMU == "yapılıyor": */}
{{PROJE_ADI}} kendi modelini eğitiyor veya fine-tune ediyor. Bu eğitim için
kullanılan veriler:

- **Kapsam**: {{FINE_TUNE_VERI_KAPSAMI}}
- **Anonymize/pseudonymize**: {{FINE_TUNE_ANONIMIZE_DURUMU}}
- **Kullanıcı rızası**: {{FINE_TUNE_RIZA_DURUMU}} — eğer rıza alınmadan
  kullanıcı verisi training'e giriyorsa **KVKK md. 5/1 ihlali** riski

{/* Eğer FINE_TUNE_DURUMU == "yapılmıyor": */}
{{PROJE_ADI}} kendi AI modeli eğitmez veya fine-tune etmez. Sadece sağlayıcının
hazır modellerini API üzerinden kullanır.

### X.4. RAG (Retrieval-Augmented Generation) — Vector Store

**{{RAG_DURUMU}}**

{/* Eğer RAG kullanılıyor: */}
{{PROJE_ADI}} RAG sistemi kullanır. Bu, AI cevap üretirken aşağıdaki kaynaklardan
**bağlam çeker**:

| Kaynak | İçerik | Saklama | Kullanıcı verisi içeriyor mu? |
|---|---|---|---|
| {{RAG_KAYNAK_1}} | {{RAG_ICERIK_1}} | {{RAG_SAKLAMA_1}} | {{RAG_USER_DATA_1}} |
| {{RAG_KAYNAK_2}} | {{RAG_ICERIK_2}} | {{RAG_SAKLAMA_2}} | {{RAG_USER_DATA_2}} |

**Önemli**: Eğer kullanıcı verisi vector store'a indekslenmişse, bu **veri saklama**
KVKK md. 7 (gerekli süre) ve md. 5/1 (açık rıza) kapsamına girer.

### X.5. Kullanıcı İçerik → AI Eğitimi Riski

Bazı projelerde kullanıcı içeriği (kayıtlar, mesajlar, dosyalar) **dolaylı
yollarla** AI training'e sızabilir:

- ✅ **Bizim önlemimiz**: Kullanıcı verisi hiçbir koşulda AI sağlayıcısının
  training'ine yansımıyor (API agreement gereği).
- ⚠️ **Sınır**: Kullanıcı verisi sadece **session içinde** AI'ya gönderiliyor;
  RAG/fine-tune **yapılmıyorsa** (yukarıdaki X.3 + X.4 bölümlerine bakın).
- 🔴 **Risk**: Eğer kullanıcı verisi vector store'a + fine-tune'a gidiyorsa,
  bu mutlaka **açık rıza** ile yapılmalı.

### X.6. Yurt Dışı Veri Aktarımı (KVKK md. 9)

AI sağlayıcıların büyük çoğunluğu **ABD merkezli**. Bu aktarım için:

- **KVKK md. 9**: Yeterli koruma sağlanan ülkeler listesinde ABD **yok**;
  yurt dışı aktarım için **açık rıza** veya **bağlayıcı taahhüt** gerekir.
- **GDPR Art. 44+**: AB'den ABD'ye veri aktarımı için **EU-US Data Privacy
  Framework** (2023+) altında belirli sağlayıcılar belgesi var. Diğerleri için
  SCC (Standard Contractual Clauses) imzalanması gerekir.
- **Bizim yaklaşımımız**: Açık rıza checkbox'ında **veri aktarımının ülke
  ve sağlayıcı bilgisi** açıkça belirtilmiştir.

### X.7. AI Sağlayıcıların DPA (Data Processing Agreement) Durumu

| Sağlayıcı | DPA | Imzalanma | Link |
|---|---|---|---|
| Anthropic | ✓ Var | {{ANTHROPIC_DPA_DATE}} | anthropic.com/legal/dpa |
| OpenAI | ✓ Var | {{OPENAI_DPA_DATE}} | openai.com/policies/data-processing-addendum |
| Google | ✓ Var | {{GOOGLE_DPA_DATE}} | cloud.google.com/terms/data-processing-addendum |
| {{DIGER_VENDOR}} | {{DIGER_DPA}} | {{DIGER_DPA_DATE}} | {{DIGER_DPA_LINK}} |

KVKK md. 12 (veri işleyen ile veri sorumlusu arasındaki sözleşme) için bu DPA'lar imzalanmıştır.

### X.8. Veri Sahibi Olarak Haklarınız

AI sistemiyle ilgili veri sahibi hakları için (KVKK md. 11 / GDPR Art. 15-22):

- **Verilerime erişim**: <a href="/hesabim">Hesabım</a> → Verilerimi İndir
- **Veri silme**: <a href="/hesabim">Hesabım</a> → Hesabımı Sil (AI sağlayıcıdaki
  30 gün buffer sonrası tamamen silinir)
- **Otomatik karar verme itiraz** (Art. 22): <a href="mailto:{{ILETISIM_EMAIL}}">İletişim</a>
- **AI bias raporu**: <a href="/ai-bias">Bias Sayfası</a>'nda bilgi
```

---

## English variant — key sections

```markdown
## X. AI and Data Flow

### X.1. How Your Data Reaches AI Systems

Your inputs are transmitted from {{PROJECT_NAME}} servers to:
{/* table */}

### X.2. Is Your Data Used for AI Model Training?

**Short answer**: {{TRAINING_USE_SUMMARY}}

- Anthropic Claude API (commercial agreement): API data is NOT used for general training. Anthropic retains it for 30 days for security/abuse detection.
- OpenAI API (default): Since March 2023, API data is NOT used for training (default opt-out).
- Google Generative AI (paid tier): API data not used for training.
{/* per-provider details */}

Important: **Free/consumer chatbots** (claude.ai, chatgpt.com, gemini.google.com) DIFFER from API usage — we use APIs.

### X.3. Fine-tuning / Custom Model Training
{/* per status */}

### X.4. RAG (Retrieval-Augmented Generation) — Vector Stores
{/* per status */}

### X.5. User Content → AI Training Risk
{/* mitigations + caveats */}

### X.6. International Data Transfer (GDPR Art. 44+)

Most AI providers are **US-based**. For this transfer:
- **GDPR Art. 44+**: EU-US Data Privacy Framework (2023+) coverage for certified providers
- **For others**: SCC (Standard Contractual Clauses) signed
- **Our practice**: Consent checkbox explicitly states the country and provider

### X.7. AI Provider DPA Status

{/* table of DPAs */}

### X.8. Your Rights as Data Subject

For AI-related rights (GDPR Art. 15-22):
- **Access**: /account → Export My Data
- **Erasure**: /account → Delete My Account (fully deleted after 30-day provider buffer)
- **Automated decision-making objection** (Art. 22): contact us
- **AI bias report**: see /ai-bias
```

---

## Karar matrisi — Eklenecek bölümler proje türüne göre

| Proje özelliği | X.3 (Fine-tune) | X.4 (RAG) | X.5 (Risk uyarısı) |
|---|---|---|---|
| Sadece API call (Anthropic/OpenAI), no storage | "yapılmıyor" | "yapılmıyor" | Yeşil — risk düşük |
| RAG indeksli (vector store kullanıcı içeriği indekslemiyor) | "yapılmıyor" | "var, ama kullanıcı verisi içermez" | Sarı — vector store içeriği denetlenmeli |
| RAG indeksli + kullanıcı içeriği indekslenmiş | "yapılmıyor" | "var, kullanıcı içeriği içerir" | 🔴 — açık rıza zorunlu, X.4 dolu |
| Custom fine-tune yapılmış | "yapılmış" | possibly var | 🔴 — fine-tune verisi nereden, rıza nasıl alındı detaylı yaz |
| RLHF / preference data collected | "kullanıcı feedback'i ile" | — | 🟡 — feedback'in training'e gitmesi rıza ile |

---

## Senin projelerine örnek

**Yıldızname**:
- Sadece Anthropic API call, no RAG, no fine-tune → X.3 "yapılmıyor", X.4 "yapılmıyor", X.5 yeşil

**VAL-NSPE LM**:
- Symbolic wrapper around Claude API → X.3 "yapılmıyor"; ama "evaluation data" topluyorsa X.4 minimal RAG (test queries) — açıkla
- Phase 2 fine-tune'a geçecek: X.3 "planlı, kullanıcı verisi içermeyen synthetic dataset üzerinde"

**Philosophy Research**:
- Eğer literatür RAG indekslemiş ise X.4 dolu; kullanıcı sorgusu indekse eklemiyorsa risk düşük
- Anthropic API call → X.3 yapılmıyor

**TargetMind AI**:
- Otomatik karar verme + customer profiling → **Art. 22 / md. 5/2-d uyarısı kritik**
- Embedding-based customer modeling = X.4 RAG dolu, X.5 sarı/kırmızı (kullanıcı segmentasyonu için)

**Bloom Within**:
- AI yorumlama → X.3 yapılmıyor, X.4 minimal (chakra knowledge base RAG olabilir)

---

## Önemli notlar

1. **Versioning kritik**: API agreement değişirse (sağlayıcı 30 gün → 60 gün retention'a geçerse) sayfa **derhal** güncellenir.
2. **Vendor lock-in transparency**: Hangi vendor'a bağımlı olduğun açıkça gözükmeli.
3. **Çoklu vendor**: Bir proje birden çok AI sağlayıcısı kullanıyorsa her birinin satırı ayrı olmalı.
4. **Free vs paid distinction**: Free tier'da training opt-in genellikle default; bu kritik bir ayrım, kullanıcıya açıklanmalı.
