---
template: kvkk-aydinlatma
language: tr
legal_basis:
  - kvkk_md_10
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "Kurul kararları ile güncellenebilir; resmi metin için kvkk.gov.tr"
---

# KVKK Aydınlatma Metni Şablonu

Bu şablon, Next.js App Router için `app/aydinlatma/page.tsx` olarak kullanılır. Diğer framework'lerde HTML/Markdown'a çevir.

**Placeholder'lar** (Claude bunları doldurur):
- `{{PROJE_ADI}}` — site adı (örn. "Yıldızname")
- `{{VERI_SORUMLUSU_AD}}` — gerçek kişi adı veya ticari unvan
- `{{ILETISIM_EMAIL}}` — KVKK için iletişim email
- `{{TOPLANAN_VERILER_LISTE}}` — markdown liste (örn. "- Ad ve soyad\n- Email")
- `{{ISLEME_AMACI_LISTE}}` — markdown liste (proje türüne göre)
- `{{UCUNCU_TARAF_BLOK}}` — her üçüncü taraf için paragraf (Anthropic, Supabase, vs.)
- `{{SAKLAMA_DURUMU}}` — "saklanmaz" veya saklanan veriler + süreler
- `{{CEREZ_DURUMU}}` — "kullanılmaz" veya kullanılan çerez listesi
- `{{TARIH}}` — bugünün tarihi
- `{{SHARED_STYLE}}` — proje tasarımına uygun stil import'u (örn. "import './styles.css'")

---

## `app/aydinlatma/page.tsx`

```tsx
import type { Metadata } from "next";
import Link from "next/link";

export const metadata: Metadata = {
  title: "Aydınlatma Metni — {{PROJE_ADI}}",
  description: "KVKK kapsamında kişisel verilerin işlenmesine ilişkin aydınlatma metni.",
};

export default function AydinlatmaPage() {
  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-3xl mx-auto">
        <header className="text-center mb-12">
          <h1 className="text-3xl md:text-4xl font-light tracking-wide">Aydınlatma Metni</h1>
          <p className="text-sm opacity-70 mt-2">Kişisel Verilerin Korunması Kanunu kapsamında</p>
        </header>

        <article className="prose prose-invert max-w-none space-y-6">
          <p className="border-l-2 pl-4 italic opacity-90">
            6698 sayılı Kişisel Verilerin Korunması Kanunu&apos;nun (&ldquo;KVKK&rdquo;)
            10. maddesi uyarınca, veri sorumlusu sıfatıyla aşağıdaki bilgilendirme
            tarafınıza sunulmaktadır.
          </p>

          <section>
            <h2>1. Veri Sorumlusu</h2>
            <p>
              Bu internet sitesi (&ldquo;{{PROJE_ADI}}&rdquo;) gerçek kişi veri
              sorumlusu <strong>{{VERI_SORUMLUSU_AD}}</strong> tarafından
              işletilmektedir.
            </p>
            <p>
              <strong>İletişim:</strong>{" "}
              <a href="mailto:{{ILETISIM_EMAIL}}">{{ILETISIM_EMAIL}}</a>
            </p>
            <p className="text-sm opacity-70">
              Bu adres yalnızca KVKK kapsamındaki haklara ilişkin talepler için
              kullanılmaktadır.
            </p>
          </section>

          <section>
            <h2>2. İşlenen Kişisel Veriler</h2>
            <p>Sembolik/hizmet amacıyla aşağıdaki kişisel verileriniz işlenir:</p>
            <ul>
{{TOPLANAN_VERILER_LISTE}}
            </ul>
            <p>
              Bu verilerin tümünü siz, kendi rızanızla form/kayıt aracılığıyla
              girersiniz. Site, kullanıcının verilerini başka bir kaynaktan edinmez
              veya zenginleştirmez.
            </p>
          </section>

          <section>
            <h2>3. İşleme Amacı</h2>
            <p>Söz konusu veriler yalnızca aşağıdaki amaçlarla işlenir:</p>
            <ul>
{{ISLEME_AMACI_LISTE}}
            </ul>
            <p>
              Veriler; pazarlama, profilleme, reklam, üçüncü kişilere satış,
              istatistik veya analiz amacıyla <strong>kullanılmaz</strong>.
            </p>
          </section>

          <section>
            <h2>4. İşlemenin Hukuki Sebebi</h2>
            <p>
              Verileriniz KVKK madde 5/1 uyarınca yalnızca{" "}
              <strong>açık rızanız</strong> hukuki sebebiyle işlenir. Formu/kaydı
              tamamlamadan önce işaretlediğiniz onay kutusu, bu açık rızanın
              belgesi niteliğindedir.
            </p>
          </section>

          <section>
            <h2>5. Saklama Süresi</h2>
{{SAKLAMA_DURUMU}}
          </section>

          <section>
            <h2>6. Aktarılan Taraflar — Yurt Dışı Aktarım</h2>
{{UCUNCU_TARAF_BLOK}}
          </section>

          <section>
            <h2>7. Veri Sahibi Hakları (KVKK Md. 11)</h2>
            <p>KVKK&apos;nın 11. maddesi uyarınca her veri sahibi şu haklara sahiptir:</p>
            <ul>
              <li>Kişisel verilerinizin işlenip işlenmediğini öğrenme</li>
              <li>İşlenmişse buna ilişkin bilgi talep etme</li>
              <li>İşlenme amacını ve amacına uygun kullanılıp kullanılmadığını öğrenme</li>
              <li>Yurt içinde veya yurt dışında aktarıldığı üçüncü kişileri bilme</li>
              <li>Eksik veya yanlış işlenmişse düzeltilmesini isteme</li>
              <li>KVKK&apos;da öngörülen şartlar çerçevesinde silinmesini veya yok edilmesini isteme</li>
              <li>İşlemenin yalnızca otomatik sistemlerle analiziyle aleyhinize bir sonuç doğmasına itiraz etme</li>
              <li>Kanuna aykırı işleme sebebiyle zarara uğramışsanız tazminat talep etme</li>
            </ul>
            <p>
              Bu talepleri{" "}
              <a href="mailto:{{ILETISIM_EMAIL}}">{{ILETISIM_EMAIL}}</a> adresine
              yazılı olarak iletebilirsiniz.
            </p>
          </section>

          <section>
            <h2>8. Çerezler</h2>
{{CEREZ_DURUMU}}
          </section>

          <section>
            <h2>9. Açık Rızanın Geri Alınması</h2>
            <p>
              Vermiş olduğunuz açık rızayı dilediğiniz zaman geri alabilirsiniz.
              Geri alma, ileriye dönük etki doğurur.
            </p>
          </section>

          <section>
            <h2>10. Değişiklikler</h2>
            <p>
              Bu metnin yürürlükteki sürümü her zaman bu sayfada yayınlanır.
              Önemli değişiklikler olduğunda metnin başına güncelleme tarihi eklenir.
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

## Placeholder doldurma rehberi

### `{{TOPLANAN_VERILER_LISTE}}` örnek

PII tespitine göre Claude doldurur:
```
              <li>Ad ve soyad</li>
              <li>Email adresi</li>
              <li>Doğum tarihi</li>
              <li>Doğum yeri</li>
```

### `{{ISLEME_AMACI_LISTE}}` örnek

Proje türüne göre:
```
              <li>Sembolik yorum metni üretmek</li>
              <li>Kullanıcının taleplerini yanıtlamak</li>
```

### `{{SAKLAMA_DURUMU}}` örnek

Eğer veri saklanmıyorsa:
```html
<p className="border-l-2 pl-4">
  Site, kişisel verilerinizi <strong>hiçbir veritabanında, dosyada veya kalıcı
  bellekte saklamaz</strong>. Veriler yalnızca işlem süresince sunucu belleğinde
  tutulur ve istek tamamlandığında otomatik olarak silinir.
</p>
```

Eğer saklanıyorsa:
```html
<p>
  Verileriniz <strong>{{X gün/ay/yıl}}</strong> süreyle Supabase
  veritabanında saklanır. Bu sürenin sonunda otomatik olarak silinir veya
  silme talebinizle daha önce silinebilir.
</p>
```

### `{{UCUNCU_TARAF_BLOK}}` örnek

Her tespit edilen üçüncü taraf için ayrı paragraf. Örn. Anthropic ve Vercel için:
```html
<p className="border-l-2 pl-4">
  Yorum metnini üretmek için yapay zekâ hizmeti veren
  <strong>Anthropic, PBC</strong> (ABD) adlı şirketin Claude isimli büyük dil
  modeli kullanılmaktadır. Girdiğiniz veriler, yorum talebi sırasında bu
  şirketin sunucularına iletilir. Bu aktarım, KVKK madde 9 kapsamında bir
  yurt dışı aktarımdır. Anthropic, kendi gizlilik politikası çerçevesinde
  verileri 30 güne kadar güvenlik denetimi amacıyla saklayabilir.
  <a href="https://www.anthropic.com/legal/privacy" target="_blank" rel="noopener">
    anthropic.com/legal/privacy
  </a>
</p>

<p>
  Sitenin barındırma altyapısı <strong>Vercel Inc.</strong> (ABD) tarafından
  sağlanmaktadır. Vercel, sunucu erişim kayıtlarında IP adresi ve teknik
  loglar tutar; form içerikleri bu loglara dahil edilmez.
</p>
```

### `{{CEREZ_DURUMU}}` örnek

Eğer çerez yoksa:
```html
<p>
  Site, kullanıcı takibi veya analiz amaçlı çerez <strong>kullanmaz</strong>.
  Form verileri yalnızca tarayıcının geçici oturum belleğinde (sessionStorage)
  tutulur; sekmeyi kapattığınızda silinir.
</p>
```

Eğer çerez varsa: kullanılan çerezlerin listesi + amaçları + süreleri.

---

*Bu şablon hukuki tavsiye değildir. Canlıya çıkmadan önce lisanslı avukatla incelenmesi önerilir.*
