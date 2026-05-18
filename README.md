# Legal Lens

> **EN**: A Claude Code skill that audits codebases for KVKK/GDPR compliance gaps and generates jurisdiction-aware legal templates. 12 risk surfaces · hybrid jurisdiction detection (operator + language + domain) · 14 boilerplate templates · automatic schema parsing for data-subject-rights endpoints.

Kodtabanındaki hukuki risk yüzeylerini yüzeye çıkaran, KVKK/GDPR şablonları üreten bir Claude Code skill'i.

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
/legal-lens             # tam tarama yap, LEGAL_AUDIT.md raporu üret
/legal-lens audit       # (aynısı)
/legal-lens generate kvkk    # /aydinlatma + /kosullar + footer + consent oluştur (Türkçe)
/legal-lens generate gdpr    # /privacy + /terms + cookie-policy + /account + footer + consent (İngilizce)
/legal-lens consent     # mevcut form'lara açık rıza checkbox ekle (TR veya EN, jurisdiction tespitine göre)
/legal-lens footer      # yasal linkli footer component'i ekle
/legal-lens cookies     # cookie banner + ConsentGate + cookie policy kur (jurisdiction tespitine göre)
/legal-lens rights      # KVKK md. 11 / GDPR Art. 15-22 paneli + export + delete (Supabase varyantı)
/legal-lens check       # sadece son değişiklikleri (git diff) tara
```

---

## Ne tarıyor

`reference/risk-checklist.md` dosyasındaki canonical listeye göre 12 yüzey:

1. **PII toplama**: form'lar (email, isim, telefon, doğum tarihi, vs.) ve API endpoint'lerinde gelen alanlar
2. **Üçüncü taraf veri aktarımı**: `@anthropic-ai/sdk`, `@supabase`, `openai`, `stripe`, `firebase`, analytics SDK'ları
3. **Server-side kalıcılık**: DB bağlantıları, dosya yazımı, log'lar
4. **Riskli iddialar**: sağlık, finans, kehanet, eğitim, çocuk alanlarında ürettiği copy
5. **Eksik yasal sayfalar**: `/aydinlatma`, `/kosullar`, `/privacy`, `/terms`
6. **Cookie ve tracking**: tracking SDK + storage + 3rd-party embed
7. **Deploy durumu**: 4-katmanlı cross-check (repo marker + CLI + URL probe + git remote)
8. **UGC platform**: post/comment/upload sosyal sinyalleri + 5651 yer sağlayıcı yükümlülüğü
9. **Yaş doğrulama**: 13/16/18+ kontrolleri (COPPA, KVKK md. 5/2-d, GDPR Art. 8)
10. **KVKK md. 11 endpoint'leri**: `/hesabim` + export + silme (schema parse ile otomatik tablo tespiti)
11. **E-ticaret / ödeme**: ETBİS / mesafeli satış / cayma hakkı flag'i
12. **Gıda / sağlık / wellness**: alerjen, tarif doğruluğu, tıbbi tavsiye sınırı, sağlık iddiası

**Jurisdiction tespiti hibrit**: içerik dili + operatör konumu + domain — sadece dile bakmaz, operatör Türkiye'deyse İngilizce site bile KVKK'ya tabi.

---

## Faz durumu (bilinen eksikler)

- ✅ Cookie banner + çerez politikası + ConsentGate (Faz 1.1)
- ✅ KVKK md. 11 hakları — /hesabim paneli + veri export + hesap silme (Faz 1.2, Supabase + Prisma varyantları)
- ✅ Cookie banner granular (kategori bazlı): necessary / functional / analytics / marketing toggle'ları + `/cerez-ayarlari` sonradan değiştirme sayfası (Faz 1.3)
- ✅ Deploy tespiti 4-katmanlı cross-check, jurisdiction hibrit tespit (operatör + dil + domain), UGC platform 4-sinyalli auto-detection, schema parse mantığı (Supabase/Prisma/Drizzle), gıda/sağlık/wellness özel risk yüzeyi (Faz 1.4)
- ✅ **GDPR İngilizce şablonları** — privacy-policy, terms-of-service, cookie-policy, cookie-banner-en, gdpr-rights, consent-checkbox-en, footer-en; `/legal-lens generate gdpr` artık çalışıyor (Faz 2.0)
- 🚧 ETBİS / mesafeli satış / cayma hakkı (e-ticaret) şablonları yok (Faz 2.1)
- 🚧 KEP adresi / VERBİS kayıt rehberi yok (Faz 2.2)
- 🚧 Çoklu jurisdiction (TR + EN paralel siteler) için tam i18n şablonları minimum — banner string tablosu var, ama next-intl entegrasyonu manuel

İlerleyen sürümlerde eklenecek. Bu skill'i kullandıkça çıkan boşlukları not al.

---

## Katkı / Geri bildirim

Issue veya PR açabilirsin. Skill kendi projelerimde test edilerek gelişiyor — bug report, eksik risk yüzeyi, yanlış şablon dili için issue çok değerli.

**Önemli**: KVKK/GDPR yorumları zaman içinde değişebilir (Kurul kararları, EDPB rehberleri). Şablonlardaki dil için sürekli güncelleme gerekir — bu kapsamda PR'lara açığım.

---

*v0.2.0 · Mayıs 2026 · Bu skill hukuki tavsiye vermez.*
