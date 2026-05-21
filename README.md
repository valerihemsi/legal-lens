# Legal Lens

> **EN**: A Claude Code skill that audits codebases for KVKK/GDPR/EU-AI-Act compliance gaps and generates jurisdiction-aware legal templates. **13 risk surfaces** · hybrid jurisdiction detection (operator + language + domain) · **18 boilerplate templates** · automatic schema parsing for data-subject-rights endpoints · **0-100 compliance scoring** · AI/GenAI special obligations · e-commerce package.

Kodtabanındaki hukuki risk yüzeylerini yüzeye çıkaran, KVKK/GDPR/EU AI Act şablonları üreten bir Claude Code skill'i.

## ⚠️ Bu hukuki tavsiye değildir

Legal Lens **lisanslı avukatlık hizmeti vermez**. Yaptığı şey:

- ✅ Kodtabanındaki yaygın hukuki risk yüzeylerini (PII formları, eksik aydınlatma metni, üçüncü taraf veri aktarımı, vs.) **bilgilendirici amaçla** yüzeye çıkarmak
- ✅ KVKK md. 10 ve GDPR md. 13 şablon dilinde **boilerplate** sayfa metinleri üretmek
- ✅ Açık rıza checkbox, footer ve cookie banner snippet'leri sağlamak

Yapmadığı şey:

- ❌ Belirli bir mahkemede savunma garantisi
- ❌ Belirli bir senin durumun için "şu yasal kararı al" demek
- ❌ Yorumların doğruluğu için garanti
- ❌ Türkiye Avukatlık Kanunu Md. 35 kapsamındaki hukuki danışmanlık

**Üretilen şablonlar boilerplate'tir.** Canlıya çıkmadan önce — özellikle ticari faaliyet, ücretli hizmet, çocuk verisi, sağlık verisi söz konusuysa — lisanslı bir KVKK/dijital hukuk avukatıyla 1-2 saatlik review yaptırılması önerilir.

Aracın yanlış/eksik çıktısından kaynaklanan idari para cezası, dava veya başka zararlardan yazar hiçbir sorumluluk kabul etmez.

---

## Kurulum

Repo'yu klonla ve Claude Code skills dizinine kopyala:

```bash
git clone https://github.com/valerihemsi/legal-lens.git
mkdir -p ~/.claude/skills
cp -r legal-lens ~/.claude/skills/

# Doğrula
ls ~/.claude/skills/legal-lens/SKILL.md
```

Sonra Claude Code'da `/legal-lens` yazarak çalıştır.

---

## Kullanım

```
/legal-lens             # tam tarama yap, LEGAL_AUDIT.md raporu üret (0-100 skor ile)
/legal-lens audit       # (aynısı)
/legal-lens generate kvkk    # /aydinlatma + /kosullar + footer + consent oluştur (Türkçe)
/legal-lens generate gdpr    # /privacy + /terms + cookie-policy + /account + footer + consent (İngilizce)
/legal-lens generate ai      # AI projeleri: model card + output disclaimer + bias statement + data lineage (v0.3)
/legal-lens generate ecommerce # E-ticaret: mesafeli satış + cayma hakkı + iade-iptal + ETBİS (v0.3)
/legal-lens consent     # mevcut form'lara açık rıza checkbox ekle (TR veya EN, jurisdiction tespitine göre)
/legal-lens footer      # yasal linkli footer component'i ekle
/legal-lens cookies     # cookie banner + ConsentGate + cookie policy kur (jurisdiction tespitine göre)
/legal-lens rights      # KVKK md. 11 / GDPR Art. 15-22 paneli + export + delete (Supabase varyantı)
/legal-lens check       # sadece son değişiklikleri (git diff) tara
```

---

## Ne tarıyor

`reference/risk-checklist.md` dosyasındaki canonical listeye göre **13 yüzey**:

1. **PII toplama** (weight 3): form'lar (email, isim, telefon, doğum tarihi, vs.) ve API endpoint'lerinde gelen alanlar
2. **Üçüncü taraf veri aktarımı** (weight 3): `@anthropic-ai/sdk`, `@supabase`, `openai`, `stripe`, `firebase`, analytics SDK'ları
3. **Server-side kalıcılık** (weight 2): DB bağlantıları, dosya yazımı, log'lar
4. **Riskli iddialar** (weight 2): sağlık, finans, kehanet, eğitim, çocuk alanlarında ürettiği copy
5. **Eksik yasal sayfalar** (weight 3): `/aydinlatma`, `/kosullar`, `/privacy`, `/terms`
6. **Cookie ve tracking** (weight 2): tracking SDK + storage + 3rd-party embed
7. **Deploy durumu** (weight 1): 4-katmanlı cross-check (repo marker + CLI + URL probe + git remote)
8. **UGC platform** (weight 3): post/comment/upload sosyal sinyalleri + 5651 yer sağlayıcı yükümlülüğü
9. **Yaş doğrulama** (weight 2): 13/16/18+ kontrolleri (COPPA, KVKK md. 5/2-d, GDPR Art. 8)
10. **KVKK md. 11 endpoint'leri** (weight 3): `/hesabim` + export + silme (schema parse ile otomatik tablo tespiti)
11. **E-ticaret / ödeme** (weight 3): ETBİS + mesafeli satış + cayma hakkı + iade-iptal (v0.3'te genişletildi)
12. **Gıda / sağlık / wellness** (weight 3): alerjen, tarif doğruluğu, tıbbi tavsiye sınırı, sağlık iddiası
13. **AI / GenAI** (weight 3, v0.3 yeni): EU AI Act + bias disclosure + model card + Art. 50 transparency + Art. 22 otomatik karar verme

**Skorlama**: Her bulgu severity (🟢=1, 🟡=4, 🔴=10) × yüzey ağırlığı (1-3). Toplam düşüş 100'den çıkarılır. Trend tracking (multi-run audit). Detay: `reference/risk-weights.md`.

**Jurisdiction tespiti hibrit**: içerik dili + operatör konumu + domain — sadece dile bakmaz, operatör Türkiye'deyse İngilizce site bile KVKK'ya tabi.

---

## Faz durumu

- ✅ Cookie banner + çerez politikası + ConsentGate (Faz 1.1)
- ✅ KVKK md. 11 hakları — /hesabim paneli + veri export + hesap silme (Faz 1.2)
- ✅ Cookie banner granular kategori toggle'ları (Faz 1.3)
- ✅ Deploy tespiti 4-katmanlı, jurisdiction hibrit, UGC 4-sinyal, schema parse, gıda/sağlık/wellness (Faz 1.4)
- ✅ **GDPR İngilizce şablonları** — privacy-policy + terms + cookie-policy + cookie-banner-en + gdpr-rights + consent-checkbox-en + footer-en (Faz 2.0)
- ✅ **v0.3 (2026-05-21)**:
  - **0-100 uyum skorlama** (severity × weight + modifier'lar + multi-run trend)
  - **14 şablona versiyon metadata** (last_updated, legal_basis, review_cadence)
  - **AI/GenAI yüzeyi (Yüzey 13)** + 4 yeni şablon (output-disclaimer, model-card, bias-statement, data-lineage) + `reference/ai-act-tiers.md` EU AI Act 2024 rehberi + `/legal-lens generate ai` modu
  - **E-ticaret modülü** — Yüzey 11 detaylı genişletme + 4 yeni şablon (mesafeli-satis, cayma-hakki, iade-iptal, etbis-bilgi-formu) + `/legal-lens generate ecommerce` modu
- 🚧 KEP adresi / VERBİS kayıt rehberi (Faz 3.1)
- 🚧 DPA (Data Processing Agreement) şablonu (Faz 3.2)
- 🚧 CI/CD entegrasyonu (GitHub Actions + pre-commit hook) (Faz 3.3)
- 🚧 İhlal bildirimi (KVKK md. 12) 72-saat akışı (Faz 3.4)
- 🚧 Veri akış haritası (görsel) (Faz 3.5)
- 🚧 Çoklu jurisdiction (TR + EN paralel) için next-intl entegrasyonu (Faz 3.6)

İlerleyen sürümlerde eklenecek. Bu skill'i kullandıkça çıkan boşlukları not al.

---

## Katkı / Geri bildirim

Issue veya PR açabilirsin. Skill kendi projelerimde test edilerek gelişiyor — bug report, eksik risk yüzeyi, yanlış şablon dili için issue çok değerli.

**Önemli**: KVKK/GDPR yorumları zaman içinde değişebilir (Kurul kararları, EDPB rehberleri). Şablonlardaki dil için sürekli güncelleme gerekir — bu kapsamda PR'lara açığım.

---

*v0.3.0 · Mayıs 2026 · Bu skill hukuki tavsiye vermez.*
