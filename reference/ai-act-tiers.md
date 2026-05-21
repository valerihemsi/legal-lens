# EU AI Act — Risk Tier Sınıflandırma Rehberi

> **EU AI Act (Regulation 2024/1689)** Ağustos 2024'te yürürlüğe girdi. Risk-tier yaklaşımıyla AI sistemlerini sınıflandırıyor; her tier'ın **farklı obligations**'ı var.
>
> ⚠️ Bu rehber **bilgilendirme amaçlıdır**. Belirli bir AI sisteminin tier'ı için lisanslı bir AI/data protection avukatıyla görüşülmesi gerekir.

---

## Hızlı Karar Akışı

```
Bu AI bireylere zarar verebilir mi?
│
├── HAYIR → Minimal risk (no obligations)
│
└── EVET
    │
    ├── Subliminal manipulation, social scoring,
    │   real-time biometric (public space), vb. mi?
    │   └── EVET → 🚫 PROHIBITED (Art. 5)
    │
    ├── Annex III'te listelenmiş bir use case mi?
    │   (eğitim, istihdam, kritik altyapı, hukuk, vb.)
    │   └── EVET → 🔴 HIGH RISK (Title III)
    │
    ├── Sınırlı şeffaflık riski mi?
    │   (chatbot, deepfake, emotion recognition, vb.)
    │   └── EVET → 🟡 LIMITED RISK (Art. 50)
    │
    └── General-Purpose AI Model mi?
        └── EVET → ⚠️ GPAI (Art. 53-55)
```

---

## Tier 1: 🚫 PROHIBITED (Art. 5)

**Yasak AI uygulamaları** — herhangi bir use case için bunlar yasak:

1. **Subliminal techniques** — bilinçaltı manipülasyon, psikolojik zafiyetleri istismar
2. **Vulnerable groups exploitation** — çocuklar, engelliler üzerinde manipülatif AI
3. **Social scoring** (devlet veya widely-used) — kişilik/davranış puanlama
4. **Predictive policing** — tek kriminolojik profilleme ile suç tahmini
5. **Untargeted scraping** for facial recognition databases
6. **Emotion recognition** — workplace ve eğitim ortamlarında
7. **Biometric categorization** — race, religion, political opinions çıkarma
8. **Real-time remote biometric identification** in public spaces (sınırlı istisnalar)

→ Ürün bu kategorideyse **deploy etme**. Avukatla görüş.

---

## Tier 2: 🔴 HIGH RISK (Title III)

**Yüksek riskli AI sistemleri** — Annex III'te listelenen alanlar:

### Annex III — High-risk use cases

| Kategori | Alt-örnekler |
|---|---|
| 1. **Biometrics** | Remote biometric ID, biometric categorization (yasak haller dışında), emotion recognition (yasak alanlar dışında) |
| 2. **Critical infrastructure** | Trafik, elektrik, su, gaz, dijital altyapı yönetimi |
| 3. **Education** | Erişim kararları, sınav değerlendirme, öğrenci yerleştirme |
| 4. **Employment** | İşe alım, performans değerlendirme, terfi/işten çıkarma kararları, **iş ilanı targeting** |
| 5. **Essential services** | Kredi puanı, sigorta risk değerlendirme, sağlık önceliklendirme |
| 6. **Law enforcement** | Bireysel risk değerlendirme, deepfake tespit, kanıt güvenilirliği |
| 7. **Migration/asylum** | Vize/sığınma kararları, risk değerlendirme |
| 8. **Justice/democracy** | Mahkeme karar destek, seçim sonucu etki |

### High-risk obligations (Bölüm III, Articles 8-15)

| Yükümlülük | Detay |
|---|---|
| **Risk management system** (Art. 9) | Continuous risk identification + mitigation throughout lifecycle |
| **Data governance** (Art. 10) | Training data — quality, relevance, bias examination |
| **Technical documentation** (Art. 11) | Pre-market documentation, model card, dataset card |
| **Record keeping** (Art. 12) | Automatic logging of outputs |
| **Transparency** (Art. 13) | Provider → user instructions (intended purpose, limitations, supervision needs) |
| **Human oversight** (Art. 14) | Meaningful human oversight design |
| **Accuracy & cybersecurity** (Art. 15) | Robustness + cybersecurity measures |

→ High-risk ise: Conformity assessment (CE marking), EU database registration, post-market monitoring.

---

## Tier 3: 🟡 LIMITED RISK (Art. 50 — Transparency Obligations)

**Sınırlı şeffaflık yükümlülükleri** — kullanıcının AI ile etkileşim halinde olduğunu bilmesi gereken sistemler:

| Sistem türü | Şeffaflık yükümlülüğü |
|---|---|
| **AI chatbot / conversational AI** | Kullanıcı AI ile konuştuğunu bilmeli (apaçık değilse) |
| **GenAI content** (text, image, audio, video) | Output **AI tarafından üretildi** olarak işaretlenmeli (machine-readable + user-visible) |
| **Deepfake** (gerçek kişilerin AI ile yapılmış simülasyonu) | Açıkça "AI generated" disclosure |
| **Emotion recognition** (yasak haller dışında) | Kullanıcıya bilgilendirme |
| **Biometric categorization** (yasak haller dışında) | Kullanıcıya bilgilendirme |

### Senin projelerine örnek

| Proje | Tier | Obligation |
|---|---|---|
| **Yıldızname** | Limited risk | "AI tarafından üretildi" + "kehanet değildir" disclaimer |
| **VAL-NSPE LM (kullanıcı-facing)** | Limited risk | AI output marking |
| **Philosophy Research (AI yorumlama)** | Limited risk | "AI generated interpretation" |
| **Bloom Within (chakra interpretation)** | Limited risk | AI generated + "tıbbi tavsiye değil" |
| **TargetMind AI** | **High risk** (Annex III #4 employment-adjacent) | Customer targeting → potansiyel high-risk; lawyer review |

---

## Tier 4: ⚠️ GPAI (General-Purpose AI Models, Art. 53-55)

**Genel-amaçlı AI modelleri** — örn. GPT, Claude, Gemini, Llama gibi foundation modeller:

### Tüm GPAI'ler için (Art. 53)

1. **Technical documentation** — Model details, training data, performance metrics
2. **Information to downstream providers** — Model capabilities + limitations
3. **Copyright compliance** — Training data'da copyright respect
4. **Training data summary** — Public summary (template by AI Office)

### Systemic Risk GPAI (Art. 55) — büyük modeller (10^25 FLOPS+)

Ek olarak:
- Model evaluations (red-teaming dahil)
- Systemic risk assessment
- Serious incident reporting
- Cybersecurity protection

→ Sen GPAI **provider** değilsin (Anthropic, OpenAI sağlıyor) ama **downstream user** olarak Art. 53 kapsamındaki bilgilere erişmen + projende belirtmen gerekiyor.

---

## Tier 5: 🟢 MINIMAL RISK

Yukarıdaki kategorilerden hiçbirine girmeyen AI uygulamaları — spam filtreleri, basit recommendation, content moderation tools (limited scope).

**Yükümlülük yok** ama **best practice**: gönüllü transparency.

---

## Coğrafi Kapsam

EU AI Act **outside EU** uygulanır:
- AI sağlayıcısı EU'da yerleşik → kapsam içi
- AI sağlayıcısı EU'da değil ama **EU'da hizmet sunuyor** → kapsam içi
- AI sistemi EU'da yerleşik birey üzerinde **etki** yapıyor → kapsam içi

→ TR'den EU kullanıcısı erişebilir bir AI ürün geliştiriyorsan: **AI Act kapsamı içindesin.**

---

## Zaman Çizelgesi

| Tarih | Yürürlük |
|---|---|
| 1 Ağustos 2024 | AI Act yürürlüğe girdi |
| 2 Şubat 2025 | Prohibited AI use cases yasaklandı |
| 2 Ağustos 2025 | GPAI rules + governance rules |
| 2 Ağustos 2026 | High-risk AI obligations (Annex III için) tam yürürlük |
| 2 Ağustos 2027 | Annex I high-risk + tam compliance |

---

## Türkiye Eşdeğeri Durumu

Türkiye'de AI'a özel kapsamlı kanun **henüz yok** (2026-05 itibariyle). Mevcut çerçeve:

- **KVKK md. 5/2-d**: Otomatik karar verme + özel nitelikli veri (kanunda öngörülen istisnalar dışında **rıza zorunlu**)
- **Türkiye Yapay Zeka Strateji Belgesi**: 2024+ taslaklar — risk-tabanlı yaklaşım önerileri ama henüz yasalaşmadı
- **Kişisel Veri Komisyonu Kararları**: AI/profilleme ile ilgili spesifik kararlar (örn. otomatik kredi skoru kararı)
- **6502 Tüketici K.**: Yanıltıcı reklam → AI tarafından üretildi göstermemek yanıltıcı

→ Türkiye'de operatörsen, **EU AI Act'i benchmark olarak takip etmek** + KVKK kararlarına uymak en sağlam strateji.

---

## Kaynak

- **Resmi metin**: [EUR-Lex Regulation 2024/1689](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
- **EU AI Office**: https://digital-strategy.ec.europa.eu/en/policies/ai-office
- **EDPB Guidelines**: AI ile veri koruma kesişimi

---

*Bu rehber bilgilendirme amaçlıdır. Belirli bir AI sisteminin sınıflandırması için lisanslı uzmanla görüşülmesi önerilir.*
