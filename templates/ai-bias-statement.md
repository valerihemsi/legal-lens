---
template: ai-bias-statement
language: tr,en
legal_basis:
  - eu_ai_act_art_10
  - eu_ai_act_art_13
  - eu_ai_act_art_50
  - kvkk_md_5_2_d
  - gdpr_art_22
jurisdiction: TR,EU
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "VAL-NSPE LM benzeri bias-aware tasarım örneği bu şablonun ardındaki yaklaşımdır"
---

# AI Bias Statement Şablonu

> AI sisteminin **bilinen bias'larını** ve **bias-aware tasarım yaklaşımını** kullanıcıya şeffaf olarak sunar. EU AI Act Art. 10 (data governance) + Art. 13 (transparency) + KVKK md. 5/2-d ile uyumlu.

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{MODEL_LISTESI}}` — kullanılan modeller
- `{{BIAS_KATEGORILERI_TABLO}}` — proje türüne göre relevant bias kategorileri
- `{{LANG}}`

---

## Next.js — `app/ai-bias/page.tsx` (Türkçe)

```tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "AI Bias Beyanı · {{PROJE_ADI}}",
  description: "Yapay zeka modellerinin taşıdığı bilinen önyargı kategorileri ve nasıl ele aldığımız",
};

export default function AIBiasPage() {
  return (
    <main className="prose max-w-3xl mx-auto py-12 px-4">
      <h1>AI Bias Beyanı</h1>
      <p className="lead">
        Yapay zeka modelleri **tarafsız değildir**. Eğitim verisindeki yapısal önyargıları
        çıktıya taşıyabilir. {{PROJE_ADI}} olarak bu önyargıları **gizlemek yerine
        görünür kılıyoruz** — kullanıcının kendi muhakemesini kurabilmesi için.
      </p>

      <h2>1. Niçin Bu Sayfa?</h2>
      <p>
        Modern AI sistemleri eğitildikleri devasa veri kümelerinden — internet, kitaplar,
        kod tabanları — **yapısal kalıpları** öğrenir. Bu kalıplar arasında bazıları
        **önyargılıdır** (bias): belirli grupları/perspektifleri sistematik olarak
        diğerlerinden farklı temsil eder.
      </p>
      <p>
        Bu bias'lar **gizli kaldıkça** zarar verir. {{PROJE_ADI}}'nin yaklaşımı:
        bias'ı **görünür kılmak**, çıktıyı kullanıcının **dış-bilgi olarak** değil
        **filtreli bilgi olarak** okumasını sağlamak.
      </p>

      <h2>2. Bilinen Bias Kategorileri</h2>
      <p>
        Kullandığımız modellerde (Claude, GPT, Gemini, vb.) gözlemlenen ve
        açık-erişim araştırmalarla belgelenmiş bias kategorileri:
      </p>

      <table>
        <thead>
          <tr>
            <th>Kategori</th>
            <th>Tezahür</th>
            <th>Etkilenebilecek alanlar</th>
          </tr>
        </thead>
        <tbody>
          {/* {{BIAS_KATEGORILERI_TABLO}} — Claude proje türüne göre doldurur */}
          <tr>
            <td><strong>Coğrafi</strong></td>
            <td>"Default reality" Batı/ABD merkezli — başka kültürlerin gerçekliği "egzotik" olarak işaretlenir</td>
            <td>Yorum, öneri, kullanıcı modelleme</td>
          </tr>
          <tr>
            <td><strong>Dilsel</strong></td>
            <td>İngilizce için performans Türkçe'ye göre daha yüksek; Türkçe nüans kaybı görülebilir</td>
            <td>Çeviri, anlam yorumlama, sembol açıklama</td>
          </tr>
          <tr>
            <td><strong>Sosyoekonomik</strong></td>
            <td>Orta-üst sınıf perspektifi default; ekonomik dezavantaj görmezden gelinebilir</td>
            <td>Tavsiye, kararlar, kaynak önerileri</td>
          </tr>
          <tr>
            <td><strong>Cinsiyet</strong></td>
            <td>Geleneksel cinsiyet rolleri pekiştirilebilir; ikili cinsiyet varsayımı default</td>
            <td>Kullanıcı modelleme, öneri</td>
          </tr>
          <tr>
            <td><strong>Yaş</strong></td>
            <td>"Genç yetişkin" (25-40) perspektifi default; yaşlı/genç deneyimi marjinalize</td>
            <td>UX kararları, öneri</td>
          </tr>
          <tr>
            <td><strong>Dini/sekular</strong></td>
            <td>"Sekular Batı" perspektifi default; dini deneyim "kültürel öğe" olarak indirgenir</td>
            <td>Sembolik yorum, anlam çıkarma</td>
          </tr>
          <tr>
            <td><strong>İlerleme/gelişmecilik</strong></td>
            <td>"Modernleşme = ilerleme" varsayımı; geleneksel bilgi türleri "eski" olarak işaretlenir</td>
            <td>Bilgi türü değerlendirme, kültürel atıf</td>
          </tr>
          <tr>
            <td><strong>Techno-determinizm</strong></td>
            <td>"Teknoloji kaderi belirler" varsayımı; eleştiri yumuşatılır</td>
            <td>AI değerlendirme, sistem kritiği</td>
          </tr>
          <tr>
            <td><strong>Politik</strong></td>
            <td>"Merkez-liberal" pozisyon default; uç pozisyonlar (sağ veya sol) "marjinal" işaretlenir</td>
            <td>Politik metin yorumu, tartışma</td>
          </tr>
          <tr>
            <td><strong>Kültürel</strong></td>
            <td>WEIRD (Western, Educated, Industrialized, Rich, Democratic) kültür default</td>
            <td>Tüm semantic alanlar</td>
          </tr>
        </tbody>
      </table>

      <h2>3. Bizim Yaklaşımımız</h2>
      <ul>
        <li>
          <strong>Bias-as-feature, bias-as-bug değil</strong>: Bias'ı gizlemek
          değil görünür kılmak.
        </li>
        <li>
          <strong>Referans çerçevesi sorgu</strong>: Her AI çıktısı için
          "kim'in default reality'sinden konuşuyor?" sorusu sorulur.
        </li>
        <li>
          <strong>Caveat ≠ objectivity</strong>: Yapay belirsizlik
          (aşırı caveat) "tarafsızlık" değildir; gerçek epistemic özürlülük
          + somut sınırlama isimlendirme tercih edilir.
        </li>
        <li>
          <strong>Çoklu perspektif zorlaması</strong>: Tek-perspektif çökmesi
          (örn. sadece Batı bakışı) yapısal olarak bias işaretidir.
        </li>
        <li>
          <strong>İnsan supervision</strong>: AI çıktısı kullanıcıya
          **yorumlanabilir bilgi** olarak sunulur, **otorite** olarak değil.
        </li>
      </ul>

      <h2>4. Kullanıcı İçin Pratik Tavsiyeler</h2>
      <ol>
        <li>
          <strong>AI çıktısını dış-bilgi gibi okuma</strong>: Kim tarafından,
          hangi referans çerçeveden üretildiğini düşün.
        </li>
        <li>
          <strong>Kendine sor: bu çıktı kimin gerçekliğini öncelikleştiriyor?</strong>
        </li>
        <li>
          <strong>Önemli kararlar için tek kaynak yapma</strong>: Uzman görüşü +
          alternatif kaynaklar + kendi muhakemen birleşmeli.
        </li>
        <li>
          <strong>Bias örneği gördüysen bildir</strong>: <a href="mailto:{{ILETISIM_EMAIL}}?subject=AI bias raporu">{{ILETISIM_EMAIL}}</a>
        </li>
      </ol>

      <h2>5. Bias Tespit + Düzeltme Süreçlerimiz</h2>
      <ul>
        <li>
          <strong>Periyodik audit</strong>: 6 aylık periyotlarla AI çıktılarının
          bias kategorilerine göre değerlendirilmesi.
        </li>
        <li>
          <strong>Prompt-engineering kalibrasyonu</strong>: Sistem prompt'larında
          referans çerçevesi sorgu, çoklu perspektif teşviki.
        </li>
        <li>
          <strong>Symbolic reasoning layer</strong> (VAL-NSPE LM yaklaşımı): Bazı
          projelerde AI çıktısı bir "bias-auditing" katmanından geçer.
        </li>
        <li>
          <strong>User feedback loop</strong>: Bias raporları sistem güncelleme
          döngüsüne girer.
        </li>
      </ul>

      <h2>6. EU AI Act Uyumu</h2>
      <p>
        Bu beyan EU AI Act Art. 10 (data governance), Art. 13 (transparency),
        Art. 50 (transparency obligations) kapsamındaki yükümlülükleri taşır.
      </p>

      <h2>7. Sınırlamalar</h2>
      <p>
        <strong>Bu beyan eksik olabilir.</strong> Bias kategorileri AI araştırma
        literatürü ile genişler; yukarıdaki liste 2026 mevcut bilgisidir.
        Yeni bias tipleri keşfedildiğinde sayfa güncellenir.
      </p>

      <h2>8. Güncellemeler</h2>
      <p>Son güncelleme: <time dateTime="{{TARIH}}">{{TARIH}}</time></p>

      <p className="mt-12 text-sm opacity-70">
        İlgili: <a href="/model-card">Model Card</a> · <a href="/aydinlatma">Aydınlatma Metni</a>
      </p>
    </main>
  );
}
```

---

## English variant — key strings

```tsx
<h1>AI Bias Statement</h1>
<p>AI models are <strong>not neutral</strong>. They carry structural biases from training data into their outputs. At {{PROJECT_NAME}} we make these biases <strong>visible rather than hidden</strong> — so users can build their own judgment.</p>

<h2>1. Why This Page?</h2>
{/* ... */}

<h2>2. Known Bias Categories</h2>
{/* table with categories: Geographic, Linguistic, Socioeconomic, Gender, Age, Religious/Secular, Progress, Techno-determinism, Political, Cultural */}

<h2>3. Our Approach</h2>
<ul>
  <li><strong>Bias-as-feature, not bias-as-bug</strong>: ...</li>
  <li><strong>Reference frame inquiry</strong>: ...</li>
  <li><strong>Caveats ≠ objectivity</strong>: ...</li>
  <li><strong>Multi-perspective requirement</strong>: ...</li>
  <li><strong>Human oversight</strong>: ...</li>
</ul>

<h2>4. Practical Advice for Users</h2>
<ol>
  <li>Don't read AI output as external knowledge...</li>
  <li>Ask: whose reality is this output prioritizing?</li>
  <li>Don't use as a single source for important decisions...</li>
  <li>Report bias examples you observe...</li>
</ol>

<h2>5. Bias Detection & Mitigation</h2>
<ul>
  <li>Periodic audit (6-month cycles)</li>
  <li>Prompt-engineering calibration</li>
  <li>Symbolic reasoning layer (VAL-NSPE LM approach)</li>
  <li>User feedback loop</li>
</ul>

<h2>6. EU AI Act Compliance</h2>
{/* Art. 10 + 13 + 50 references */}

<h2>7. Limitations</h2>
<p><strong>This statement may be incomplete.</strong> Bias categories expand with research; the above is the 2026 view. We update when new bias types are identified.</p>
```

---

## Aydınlatma metni / Privacy policy entegrasyonu

`/aydinlatma` sayfasının uygun bölümüne ekle:

```markdown
## X. Yapay Zeka ve Önyargı

Bu sistem yapay zeka kullanır ([Model Card](/model-card)). AI modelleri
eğitim verisinden yapısal önyargılar taşır. Bu önyargıları **şeffaflaştırmak**
için ayrı bir [Bias Beyanı sayfamız](/ai-bias) bulunmaktadır.

Kullanıcı haklarınız:
- AI bias örneği bildirmek (`/iletisim` veya {{ILETISIM_EMAIL}})
- Otomatik karar verme sürecine itiraz etmek (KVKK md. 11 / GDPR Art. 22)
- İnsan inceleme talep etmek
```

---

## Önemli notlar

1. **Bu beyan jenerik değil, proje-spesifik olmalı**: Yıldızname için bias kategorileri farklı (kültürel + dini ağırlıklı), TargetMind AI için farklı (sosyoekonomik + demografik), Bloom Within için farklı (kültürel + dini)
2. **Pretend olmamalı**: "Hiç bias yok" demek **kendi bir bias**. Var, isimlendiriliyor.
3. **Update cadence**: Yeni bias araştırması çıktıkça güncelle (6 ay önerilen)
4. **VAL-NSPE LM'ye bağlı projelerde** symbolic reasoning layer'ı bu sayfada açıkça anılabilir — operasyonel bias-mitigation kanıtı
