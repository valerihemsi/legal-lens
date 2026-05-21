---
template: model-card
language: tr,en
legal_basis:
  - eu_ai_act_art_53
  - eu_ai_act_art_13
  - kvkk_md_10
jurisdiction: TR,EU
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 3_months
version: "0.3"
notes: "Model versiyon değiştikçe sayfa güncellenmelidir; Hugging Face model card pattern'ı + EU AI Act technical documentation gerekleri"
---

# Model Card Şablonu

> Projende kullanılan AI modeller hakkında **şeffaf bilgi sayfası**. EU AI Act Art. 53 (GPAI downstream user) + Art. 13 (high-risk transparency) gerekleriyle uyumlu. Hugging Face model card pattern'ı temel alınmış.

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{MODEL_LISTESI_TABLO}}` — kullanılan modeller (Claude doldurur, package.json + .env analizi)
- `{{KULLANIM_AMACI}}` — bir cümle
- `{{LANG}}` — "tr" veya "en"

---

## Next.js — `app/model-card/page.tsx` (Türkçe)

```tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "Model Card · {{PROJE_ADI}}",
  description: "Bu uygulamada kullanılan yapay zeka modelleri hakkında şeffaflık bilgisi",
};

export default function ModelCardPage() {
  return (
    <main className="prose max-w-3xl mx-auto py-12 px-4">
      <h1>Model Card</h1>
      <p className="lead">
        {{PROJE_ADI}} uygulamasında kullanılan yapay zeka modelleri, kullanım amacı,
        sınırlamaları ve şeffaflık bilgileri.
      </p>

      <h2>1. Kullanılan Modeller</h2>
      <table>
        <thead>
          <tr>
            <th>Model</th>
            <th>Sağlayıcı</th>
            <th>Versiyon</th>
            <th>Amaç</th>
            <th>Veri Akışı</th>
          </tr>
        </thead>
        <tbody>
          {/* Claude doldurur — package.json + .env'den tespit */}
          <tr>
            <td>Claude Sonnet 4.6</td>
            <td>Anthropic (ABD)</td>
            <td>20250122</td>
            <td>Doğal dil yorumlama</td>
            <td>Kullanıcı promptu → Anthropic API → cevap</td>
          </tr>
          {/* {{MODEL_LISTESI_TABLO}} buraya gelecek */}
        </tbody>
      </table>

      <h2>2. Kullanım Amacı</h2>
      <p>
        Modellerin bu uygulamadaki **temel amacı**: {{KULLANIM_AMACI}}.
      </p>
      <p>
        Bu modeller şunlar için **uygun değildir**:
      </p>
      <ul>
        <li>Tıbbi teşhis veya tedavi tavsiyesi</li>
        <li>Hukuki tavsiye (lisanslı avukat yerine geçmez)</li>
        <li>Finansal yatırım kararları</li>
        <li>Acil durumlar (kriz hattı, intihar müdahalesi, vb.)</li>
        <li>Hassas kişisel veriler için karar verme</li>
      </ul>

      <h2>3. Eğitim Verisi (Sağlayıcı bildirimi)</h2>
      <p>
        Foundation modelleri (Claude, GPT, Gemini, vb.) sağlayıcı tarafından
        geniş kapsamlı internet veri kümeleri, kitaplar, kod ve diğer kaynaklarla
        eğitilmiştir. Eğitim verisinin detayları sağlayıcının yayımladığı
        materyallere göre değişir:
      </p>
      <ul>
        <li>
          <strong>Claude</strong>: Anthropic eğitim verisi açıklaması →{" "}
          <a href="https://www.anthropic.com/news/claudes-constitution" target="_blank" rel="noopener noreferrer">
            anthropic.com
          </a>
        </li>
        <li>
          <strong>GPT</strong>: OpenAI model cards →{" "}
          <a href="https://platform.openai.com/docs/models" target="_blank" rel="noopener noreferrer">
            platform.openai.com
          </a>
        </li>
        {/* {{DIGER_MODEL_LINKLERI}} */}
      </ul>

      <h2>4. Bilinen Sınırlamalar</h2>
      <ul>
        <li>
          <strong>Halüsinasyon riski</strong>: Modeller bazen yanlış bilgiyi
          özgüvenli şekilde sunabilir. Hayatî/yasal/finansal kararlar için
          doğrulama yapın.
        </li>
        <li>
          <strong>Eğitim verisi kesilme tarihi</strong>: Modeller belirli bir
          tarihten sonraki olayları bilmez (genellikle 6-12 ay öncesi).
        </li>
        <li>
          <strong>Dil performansı</strong>: Türkçe için performans İngilizce
          kadar yüksek olmayabilir; nüans kaybı görülebilir.
        </li>
        <li>
          <strong>Kültürel bias</strong>: Eğitim verisi ağırlıklı olarak
          İngilizce ve Batı kaynaklı olduğundan kültürel bias taşıyabilir.
        </li>
        <li>
          <strong>Bias kategorileri</strong>: Demografik (yaş, cinsiyet),
          coğrafi, sosyoekonomik, dini bias yansıtabilir.
          Detay: <a href="/ai-bias">/ai-bias</a>
        </li>
      </ul>

      <h2>5. Veri Akışı ve Gizlilik</h2>
      <p>
        Kullanıcı promptları AI sağlayıcısına iletildikten sonra:
      </p>
      <ul>
        <li>Anthropic: 30 gün saklanır, training için kullanılmaz (commercial agreement gereği)</li>
        <li>OpenAI: 30 gün saklanır, varsayılan olarak training için kullanılmaz (opt-in)</li>
        <li>{{DIGER_PROVIDER_RETENTION}}</li>
      </ul>
      <p>
        Detay aydınlatma metnimizdedir: <a href="/aydinlatma">/aydinlatma</a>
      </p>

      <h2>6. İnsan Denetimi</h2>
      <p>
        AI çıktıları **insan tarafından otomatik olarak gözden geçirilmez**.
        Önemli kararlar için kullanıcının kendi muhakemesi + uzman görüşü gerekir.
      </p>
      <p>
        Yanlış veya zararlı çıktı tespit ederseniz lütfen bildirin:{" "}
        <a href="mailto:{{ILETISIM_EMAIL}}?subject=AI çıktı hatası">
          {{ILETISIM_EMAIL}}
        </a>
      </p>

      <h2>7. EU AI Act Sınıflandırması</h2>
      <p>
        Bu uygulama **{{AI_ACT_TIER}}** kapsamındadır:
      </p>
      <ul>
        <li>{{TIER_AÇIKLAMA}}</li>
      </ul>

      <h2>8. Güncellemeler</h2>
      <p>Son güncelleme: <time dateTime="{{TARIH}}">{{TARIH}}</time></p>
      <p>
        Bu sayfa, kullanılan modeller veya yükümlülükler değiştiğinde güncellenir.
        Önemli değişiklikler aydınlatma metninde duyurulur.
      </p>
    </main>
  );
}
```

---

## Next.js — `app/model-card/page.tsx` (English)

Same structure, English strings (key sections):

```tsx
<h1>Model Card</h1>
<p>Transparent information about AI models used in {{PROJECT_NAME}}.</p>

<h2>1. Models in Use</h2>
{/* table */}

<h2>2. Intended Use</h2>
<p>Primary purpose in this application: {{INTENDED_USE}}.</p>
<p>These models are <strong>not appropriate</strong> for:</p>
<ul>
  <li>Medical diagnosis or treatment advice</li>
  <li>Legal advice (does not replace a licensed lawyer)</li>
  <li>Financial investment decisions</li>
  <li>Emergencies (crisis hotline, suicide intervention, etc.)</li>
  <li>Decisions involving sensitive personal data</li>
</ul>

<h2>3. Training Data</h2>
{/* per-provider info + links */}

<h2>4. Known Limitations</h2>
<ul>
  <li><strong>Hallucination risk</strong>: ...</li>
  <li><strong>Training data cutoff</strong>: ...</li>
  <li><strong>Language performance</strong>: ...</li>
  <li><strong>Cultural bias</strong>: ...</li>
  <li><strong>Bias categories</strong>: ... See <a href="/ai-bias">/ai-bias</a></li>
</ul>

<h2>5. Data Flow & Privacy</h2>
{/* per-provider retention */}

<h2>6. Human Oversight</h2>
<p>AI outputs are <strong>not automatically reviewed by a human</strong>...</p>

<h2>7. EU AI Act Classification</h2>
<p>This application falls under <strong>{{AI_ACT_TIER}}</strong>:</p>

<h2>8. Updates</h2>
<p>Last updated: <time dateTime="{{DATE}}">{{DATE}}</time></p>
```

---

## Footer'a link ekle

```tsx
<footer>
  <nav>
    <a href="/aydinlatma">Aydınlatma</a>
    <a href="/kosullar">Kullanım Koşulları</a>
    <a href="/model-card">Model Card</a> {/* YENİ */}
    <a href="/ai-bias">AI Bias</a> {/* YENİ */}
    {/* diğer linkler */}
  </nav>
</footer>
```

---

## Doldurulan placeholder örneği (Yıldızname)

```yaml
PROJE_ADI: Yıldızname
MODEL_LISTESI_TABLO:
  - { name: "Claude Sonnet 4.6", provider: "Anthropic (ABD)", version: "20250122", amac: "Sembolik yıldızname yorumu üretimi", veri_akisi: "Doğum tarihi + adı → Anthropic API → yıldızname yorumu" }
KULLANIM_AMACI: "Geleneksel Türk yıldızname yorumlama geleneğini modernleştirerek sembolik kişilik açıklaması üretmek (eğlence + öz-keşif amaçlı)"
AI_ACT_TIER: "Limited Risk (Art. 50)"
TIER_AÇIKLAMA: "Kullanıcı AI ile etkileşim halinde olduğunu bilir, çıktı 'AI tarafından üretildi' olarak işaretlidir."
ILETISIM_EMAIL: "iletisim@yildizname.com"
```

---

## Önemli notlar

1. **Versioning**: Model versiyonu (Claude Sonnet 4.6 → 4.7) değiştiğinde sayfa **mecburen** güncellenir
2. **Sağlayıcı bildirimleri** — Anthropic/OpenAI vb. eğitim verisi açıklamaları zaman içinde değişebilir, link güncel tutulmalı
3. **Çoklu model kullanan projeler** — Her modelin amacı + veri akışı ayrı tabloda
4. **Bu sayfa SEO-friendly olmalı** — robots.txt'de izin ver, sitemap.xml'e ekle
