# Risk Weights — Skorlama Sistemi

> Audit raporundaki bulgulara **sayısal ağırlık** verir, toplam **uyum skoru (0-100)** hesaplar. v0.3'te eklendi.

---

## Skor Hesabı

```
final_score = 100 - Σ(finding_severity × surface_weight)

# Severity sabit:
🟢 = 1
🟡 = 4
🔴 = 10

# Surface weight (1-3): aşağıdaki tablo
```

**Kaba aralık yorumu:**
- **90-100** — Düzgün uyum; minor improvement notları
- **70-89** — Orta uyum; 1-3 yüksek bulgu mevcut
- **50-69** — Zayıf uyum; ciddi boşluklar
- **0-49** — Kritik uyumsuzluk; production'a çıkmamalı

---

## Surface Weights (12+ yüzey)

| # | Yüzey | Weight | Rationale |
|---|---|---|---|
| 1 | PII Toplama | **3** | Direk KVKK md.5/1; idari para cezası temel sebebi |
| 2 | Üçüncü Taraf Aktarımı | **3** | KVKK md.9 yurt dışı aktarım; DPA yokluğu da burada |
| 3 | Server-Side Persistence | **2** | Saklama süresi + güvenlik tedbirleri |
| 4 | Riskli İddialar | **2** | Sağlık/finans yüksek; diğerleri orta |
| 5 | Eksik Yasal Sayfalar | **3** | KVKK md.10 + GDPR Art.13 — temel yükümlülük |
| 6 | Cookie/Tracking | **2** | Kurul kararı 2020/216 — açık rıza zorunlu |
| 7 | Deploy Durumu | **1** | Modifier; risk'i artırır ama tek başına neden değil |
| 8 | UGC Platform | **3** | 5651 + telif + DSA — çok katmanlı yükümlülük |
| 9 | Yaş Doğrulama | **2** | KVKK md.5/2-d çocuk; özel rıza akışı |
| 10 | KVKK md.11 Hakları | **3** | Veri sahibi hakları + md.13 30-gün cevap |
| 11 | E-Ticaret / Ödeme | **3** | ETBİS + mesafeli satış + cayma hakkı |
| 12 | Gıda / Sağlık / Wellness | **3** | 6502 Tüketici + sağlık reklam yön. |
| **13** | **AI / GenAI (v0.3 yeni)** | **3** | EU AI Act + transparency + bias disclaimer |

---

## Severity (sabit)

| İşaret | Değer | Tanım |
|---|---|---|
| 🟢 | 1 | Düşük risk — uyarı/iyileştirme önerisi |
| 🟡 | 4 | Orta risk — eylem gerekli, yakın vade |
| 🔴 | 10 | Yüksek risk — production öncesi zorunlu düzeltme |

---

## Hesaplama Örnekleri

### Örnek 1 — Düzgün proje (skor: 92/100)

- 🟢 Cookie banner var ama copy iyileştirilebilir (weight 2 × 1 = 2)
- 🟢 Footer linklerinde tek harf yazım hatası (weight 3 × 1 = 3)
- 🟢 Saklama süresi belirsiz aydınlatma metninde (weight 2 × 1 = 2)
- Düşülen: 7
- **Skor: 93/100** → 🟢 *"Düzgün uyum, küçük iyileştirmeler"*

### Örnek 2 — Orta uyum (skor: 71/100)

- 🟡 Cookie banner yok (weight 2 × 4 = 8)
- 🟡 KVKK md.11 endpoint'leri eksik (weight 3 × 4 = 12)
- 🟢 Footer eksik bir link (weight 3 × 1 = 3)
- 🟢 Saklama süresi belirsiz (weight 2 × 1 = 2)
- 🟢 3rd-party listesi eksik (weight 3 × 1 = 3)
- Düşülen: 28
- **Skor: 72/100** → 🟡 *"Orta uyum; cookie banner + md.11 hakları öncelikli"*

### Örnek 3 — Kritik (skor: 26/100)

- 🔴 PII toplayan form, aydınlatma yok (weight 3 × 10 = 30)
- 🔴 Cookie banner yok, GA aktif (weight 2 × 10 = 20)
- 🔴 KVKK md.11 endpoint yok (weight 3 × 10 = 30)
- 🟡 3rd-party listesi eksik (weight 3 × 4 = 12)
- 🟡 Yaş kontrolü yok (weight 2 × 4 = 8)
- Düşülen: 100 (negatif olamaz, floor 0)
- **Skor: 0/100** → 🔴 *"Kritik uyumsuzluk; production'a çıkma"*

> **Düşüş floor:** Skor negatife düşemez. Maksimum düşüş = 100. **0 = en kritik durum**.

---

## Score Trend (Multi-Run Tracking)

Aynı projede `/legal-lens audit` birden çok kez çalıştırılırsa, önceki `LEGAL_AUDIT.md`'i oku, skor trendi raporla:

```markdown
## Skor Trendi
- 2026-03-15: 42/100 🔴
- 2026-04-02: 68/100 🟡
- 2026-05-21: 89/100 🟢 ← bu run

Δ son 30 gün: +21 puan (cookie banner + md.11 endpoint'leri eklendi)
```

Bu trend kullanıcıya **ilerleme** görünür kılar, motivasyon kaynağı.

---

## Modifier Faktörleri (skor düşüşüne ek)

Bazı durumlarda **toplam skor üstünden** ek düşüş yapılır:

| Modifier | Etki |
|---|---|
| **Halka açık deploy + PII toplama + eksik yasal sayfa** | -10 (zaten yüksek bulgu var ama prod risk'i ek vurgulanır) |
| **Public git repo + .env izleniyor** | -15 (secret leak) |
| **Çocuk verisi + 18+ kontrolü yok** | -10 |
| **Sağlık iddiası + avukat review yapılmamış** | Score capped 50/100 (üst sınır kısıtlı) |
| **AI çıktısı + output disclaimer yok** | -5 (v0.3+) |

---

## Skill İçinde Kullanım

`LEGAL_AUDIT.md` üretirken:

1. Tüm bulgular toplandıktan sonra her bulgu için:
   - Severity (🟢/🟡/🔴) tespit et
   - Surface weight'ı bu tablodan al
   - Düşüş = severity × weight
2. Toplam düşüşü 100'den çıkar
3. Modifier'ları uygula
4. Floor 0
5. Sonucu raporun **Özet** bölümünde göster

```markdown
## Özet

- **Toplam Skor**: 72/100 🟡
- **Severity dağılımı**: 🔴 0 · 🟡 3 · 🟢 5
- **Yüksek ağırlık alanlar**: PII (3), 3rd-party (3), md.11 (3)
- **Skor trendi**: +12 puan (önceki audit'ten)
```

---

*Bu skorlama sistemi v0.3'te eklendi. Geri bildirim için issue aç.*
