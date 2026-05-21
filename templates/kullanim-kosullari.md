---
template: kullanim-kosullari
language: tr
legal_basis:
  - tbk_borclar_kanunu
  - 6502_tuketici_korumasi
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 12_months
version: "0.3"
---

# Kullanım Koşulları Şablonu (Türkçe)

Next.js App Router için `app/kosullar/page.tsx`. Diğer framework'lerde uyarlanır.

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{VERI_SORUMLUSU_AD}}`
- `{{ILETISIM_EMAIL}}`
- `{{HIZMET_NITELIGI}}` — proje türüne göre kısa cümle ("yapay zekâ destekli sembolik yorum aracı", "yemek sosyal platformu", vs.)
- `{{AI_BLOK}}` — AI üretimi varsa AI disclaimer paragrafı, yoksa boş
- `{{UGC_BLOK}}` — UGC varsa platform sorumluluğu paragrafı, yoksa boş
- `{{ETICARET_BLOK}}` — ödeme varsa e-ticaret bilgilendirmesi, yoksa boş
- `{{YAS_SINIRI}}` — "18", "13", vb. (default 18)
- `{{RISKLI_ALAN_LISTESI}}` — sağlık, finans, vb. iddialar varsa onlara özgü reddi
- `{{TARIH}}`

---

## `app/kosullar/page.tsx`

```tsx
import type { Metadata } from "next";
import Link from "next/link";

export const metadata: Metadata = {
  title: "Kullanım Koşulları — {{PROJE_ADI}}",
  description: "{{PROJE_ADI}} kullanım koşulları ve sorumluluk reddi.",
};

export default function KosullarPage() {
  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-3xl mx-auto">
        <header className="text-center mb-12">
          <h1 className="text-3xl md:text-4xl font-light tracking-wide">Kullanım Koşulları</h1>
          <p className="text-sm opacity-70 mt-2">ve sorumluluk reddi</p>
        </header>

        <article className="prose prose-invert max-w-none space-y-6">
          <p className="border-l-2 pl-4 italic">
            Bu siteyi kullanarak aşağıdaki koşulları kabul etmiş sayılırsınız.
            Koşulları kabul etmiyorsanız lütfen siteyi kullanmayınız.
          </p>

          <section>
            <h2>1. Hizmetin Niteliği</h2>
            <p>
              {{PROJE_ADI}}, {{HIZMET_NITELIGI}}. Sunulan içerik aşağıdaki
              niteliklere SAHİP DEĞİLDİR:
            </p>
            <ul>
{{RISKLI_ALAN_LISTESI}}
            </ul>
          </section>

{{AI_BLOK}}

          <section>
            <h2>3. Yaş Sınırı</h2>
            <p>
              Bu hizmet yalnızca <strong>{{YAS_SINIRI}} yaşını doldurmuş</strong>{" "}
              bireylerin kullanımına yöneliktir. {{YAS_SINIRI}} yaşından küçük
              kişiler için yasal temsilcilerinin açık rızası gerekir; site bu rızayı
              doğrulayacak bir mekanizma sunmadığı için reşit olmayan kişiler
              siteyi kullanmamalıdır.
            </p>
          </section>

          <section>
            <h2>4. Sorumluluk Reddi</h2>
            <p>
              Site sahibi {{VERI_SORUMLUSU_AD}}; sunulan içeriğe dayanarak alınan
              veya alınmayan kararlardan, bu kararların doğurduğu maddi, manevi,
              sağlık veya ilişki kaynaklı sonuçlardan{" "}
              <strong>sorumlu tutulamaz</strong>. İçeriğin yorumlanması,
              kullanıcının kendi sorumluluğundadır.
            </p>
            <p>
              Site sahibi, sunucu, ağ veya üçüncü taraf hizmetlerinden kaynaklanan
              kesinti, gecikme veya veri kaybından da sorumlu değildir.
            </p>
          </section>

          <section>
            <h2>5. Fikrî Haklar</h2>
            <p>
              Sitenin görsel kimliği, yazı kompozisyonları, mimari ve şablonları
              site sahibine aittir.
            </p>
            <p>
              Kullanıcı için üretilmiş içerik, kullanıcının kişisel kullanımı için
              serbestçe okunabilir, saklanabilir veya atıf vererek paylaşılabilir.
              Ticarî amaçla yeniden yayımlanması ya da yığın halinde otomatik
              üretim amacıyla siteye erişilmesi yasaktır.
            </p>
          </section>

{{UGC_BLOK}}

          <section>
            <h2>6. Kötüye Kullanım</h2>
            <p>Aşağıdaki davranışlar yasaktır:</p>
            <ul>
              <li>Başka bir kişinin bilgilerini, onun rızası olmadan girmek</li>
              <li>Bot, otomatik araç veya yığın istek üreten yöntemlerle siteye yüklenmek</li>
              <li>Hizmeti tersine mühendislikle çoğaltmaya çalışmak</li>
              <li>Hizmeti başkasını manipüle etmek veya zarar vermek amacıyla kullanmak</li>
            </ul>
          </section>

{{ETICARET_BLOK}}

          <section>
            <h2>7. Geri Bildirim ve İletişim</h2>
            <p>
              Teknik sorunlar veya geri bildirim için:{" "}
              <a href="mailto:{{ILETISIM_EMAIL}}">{{ILETISIM_EMAIL}}</a>
            </p>
          </section>

          <section>
            <h2>8. Uygulanacak Hukuk</h2>
            <p>
              Bu koşullara Türkiye Cumhuriyeti hukuku uygulanır. Doğacak
              uyuşmazlıklarda İstanbul Mahkemeleri ve İcra Daireleri yetkilidir.
            </p>
          </section>

          <section>
            <h2>9. Değişiklikler</h2>
            <p>
              Koşullar, ihtiyaç durumunda güncellenebilir; yürürlükteki sürüm her
              zaman bu sayfada bulunur.
            </p>
          </section>

          <p className="text-sm italic opacity-60 text-center mt-12">
            Son güncelleme: {{TARIH}}
          </p>

          <div className="text-center pt-8 border-t opacity-70">
            <Link href="/">← Ana sayfaya dön</Link>
          </div>
        </article>
      </div>
    </main>
  );
}
```

---

## Placeholder örnekleri

### `{{AI_BLOK}}` — AI üretimi varsa

```html
          <section>
            <h2>2. Yapay Zekâ Üretimi</h2>
            <p>
              Hizmet kapsamındaki yorumlar/yanıtlar, üçüncü taraf yapay zekâ
              modelleri tarafından gerçek zamanlı olarak üretilir. Bu tür modeller
              olasılığa dayalı çalışır; aynı veriler için farklı zamanlarda farklı
              çıktılar üretebilir. Üretilen metinde bilgi yanlışları, mantık
              tutarsızlıkları veya beklenmedik ifadeler bulunabilir. Site, üretilen
              içeriğin <strong>doğruluğu, tutarlılığı veya belirli bir niteliği
              taşıması</strong> yönünde hiçbir garanti sunmaz.
            </p>
          </section>
```

### `{{UGC_BLOK}}` — UGC platformuysa

```html
          <section>
            <h2>5a. Kullanıcı Üretimli İçerik</h2>
            <p>
              Kullanıcılar tarafından yüklenen içerikler (yazı, görsel, tarif,
              yorum vb.) yükleyen kullanıcının sorumluluğundadır. {{PROJE_ADI}}
              bu içerikleri önceden inceleme yükümlülüğü altında değildir ancak:
            </p>
            <ul>
              <li>Hakaret, taciz, ayrımcılık içeren içerikleri kaldırma hakkını saklı tutar</li>
              <li>Telif ihlali bildirimleri için <a href="mailto:{{ILETISIM_EMAIL}}">{{ILETISIM_EMAIL}}</a> adresinden iletişim kurulabilir</li>
              <li>Suç teşkil eden içerikler için yasal mercilerle işbirliği yapar</li>
              <li>Kuralları ihlal eden kullanıcıların hesabını uyarısız fesh edebilir</li>
            </ul>
            <p>
              Kullanıcı, yüklediği içeriğin telif haklarına sahip olduğunu veya
              gerekli izinleri aldığını beyan eder. {{PROJE_ADI}}, kullanıcının
              içeriğini hizmet sunmak için gerekli ölçüde işleme, görüntüleme ve
              dağıtma hakkına sahiptir.
            </p>
          </section>
```

### `{{ETICARET_BLOK}}` — ödeme varsa

```html
          <section>
            <h2>6a. Satın Alma ve Cayma Hakkı</h2>
            <p>
              Site üzerinden yapılan satın almalar 6502 sayılı Tüketicinin Korunması
              Hakkında Kanun ve Mesafeli Sözleşmeler Yönetmeliği'ne tabidir.
              Tüketici, dijital içerik teslimatından sonra <strong>14 gün
              içinde</strong> herhangi bir gerekçe göstermeksizin cayma hakkına
              sahiptir. Dijital içerik anında teslim edildiyse cayma hakkı, açık
              rızanız ile düşebilir.
            </p>
            <p>
              <strong>Cayma talebi</strong>:{" "}
              <a href="mailto:{{ILETISIM_EMAIL}}">{{ILETISIM_EMAIL}}</a>
            </p>
            <p>
              <strong>Tüketici Hakem Heyeti</strong>: Uyuşmazlıklarda ilgili
              Tüketici Hakem Heyeti'ne veya Tüketici Mahkemesi'ne başvurabilirsiniz.
            </p>
          </section>
```

### `{{RISKLI_ALAN_LISTESI}}` — riskli iddialara göre

Sağlık alanı varsa ekle:
```html
              <li>Tıbbî, psikolojik veya psikiyatrik teşhis ya da tedavi yerine geçmez</li>
              <li>Sağlık kararlarına dayanak oluşturamaz</li>
```

Kehanet/spirituel varsa ekle:
```html
              <li>Kehanet, fal veya geleceği bildirme iddiası taşımaz</li>
              <li>Dînî hüküm, fetva veya öğreti niteliğinde değildir</li>
```

Finansal varsa ekle:
```html
              <li>Yatırım, hukukî veya finansal tavsiye olarak yorumlanamaz</li>
              <li>Sermaye Piyasası Kurulu (SPK) mevzuatı kapsamında yatırım danışmanlığı değildir</li>
```

Eğitim varsa ekle:
```html
              <li>MEB onaylı bir sertifika programı değildir</li>
              <li>Resmî bir eğitim/akademik dereceye karşılık gelmez</li>
```

---

*Bu şablon hukuki tavsiye değildir. Canlıya çıkmadan önce lisanslı avukatla incelenmesi önerilir.*
