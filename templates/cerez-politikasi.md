---
template: cerez-politikasi
language: tr
legal_basis:
  - kvkk_md_10
  - kvkk_kurul_2020_216
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
---

# Çerez Politikası Şablonu

`app/cerez-politikasi/page.tsx` olarak. Aydınlatma metninden ayrı bir sayfa — çerezlere özgü detaylar burada.

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{ILETISIM_EMAIL}}`
- `{{CEREZ_LISTESI}}` — taranan/kullanılan çerezlerin tablosu (Claude doldurur)
- `{{ANALITIK_BLOK}}` — analytics SDK kullanılıyorsa o servise özgü paragraf
- `{{TARIH}}`

---

## `app/cerez-politikasi/page.tsx`

```tsx
import type { Metadata } from "next";
import Link from "next/link";

export const metadata: Metadata = {
  title: "Çerez Politikası — {{PROJE_ADI}}",
  description: "Sitede kullanılan çerezler ve tercih yönetimi hakkında bilgi.",
};

export default function CerezPolitikasiPage() {
  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-3xl mx-auto">
        <header className="text-center mb-12">
          <h1 className="text-3xl md:text-4xl font-light tracking-wide">Çerez Politikası</h1>
          <p className="text-sm opacity-70 mt-2">çerezler ve tercih yönetimi</p>
        </header>

        <article className="prose prose-invert max-w-none space-y-6">
          <section>
            <h2>Çerez nedir?</h2>
            <p>
              Çerezler (cookies), bir web sitesini ziyaret ettiğinizde tarayıcınız
              tarafından cihazınıza kaydedilen küçük metin dosyalarıdır. Tarayıcının
              yerel depolaması (localStorage, sessionStorage) da yasal düzlemde
              benzer şekilde değerlendirilir. Bu sayfada hepsine birden{" "}
              <strong>&ldquo;çerez&rdquo;</strong> diyeceğiz.
            </p>
          </section>

          <section>
            <h2>Bu sitede hangi çerezler kullanılıyor?</h2>
{{CEREZ_LISTESI}}
          </section>

          <section>
            <h2>Çerez kategorileri</h2>

            <h3>Kesinlikle Gerekli Çerezler</h3>
            <p>
              Sitenin temel işlevleri için zorunlu çerezlerdir. Onay gerektirmez,
              kapatılmaz; aksi takdirde site çalışmaz. Örnek: oturum çerezi, dil
              tercihi, form girdisi geçici depolaması.
            </p>

            <h3>Analitik Çerezler</h3>
            <p>
              Sitenin nasıl kullanıldığını anlamak için anonimleştirilmiş istatistik
              toplar. <strong>Açık rızanız olmadan yüklenmez.</strong> İlk
              ziyaretinizde gösterilen çerez bilgilendirme kutusunda
              &ldquo;Tümünü kabul et&rdquo; seçeneğini işaretlediyseniz aktiftir.
            </p>

{{ANALITIK_BLOK}}

            <h3>Pazarlama Çerezleri</h3>
            <p>
              Şu anda bu sitede pazarlama amaçlı çerez kullanılmamaktadır. İleride
              eklenirse bu sayfa güncellenecek ve onayınız yeniden istenecektir.
            </p>
          </section>

          <section>
            <h2>Tercihinizi nasıl yönetirsiniz?</h2>
            <p>
              <strong>Site içinden:</strong> Siteye ilk girdiğinizde alt köşede
              beliren çerez bilgilendirme kutusundan &ldquo;Tümünü kabul et&rdquo;
              veya &ldquo;Sadece zorunlu çerezler&rdquo; seçeneklerinden birini
              seçebilirsiniz. Tercihinizi sonradan değiştirmek için tarayıcınızın
              yerel depolamasını temizleyebilirsiniz.
            </p>
            <p>
              <strong>Tarayıcı ayarlarından:</strong> Modern tarayıcıların
              ayarlarından çerez yönetimi panelinden (örn. Chrome: Settings →
              Privacy → Cookies) çerezleri silebilir veya engelleyebilirsiniz.
              Bu durumda kesinlikle gerekli çerezler de etkilenebilir ve sitenin
              bazı bölümleri çalışmayabilir.
            </p>
          </section>

          <section>
            <h2>Yurt dışı veri aktarımı</h2>
            <p>
              Bazı çerezler, kullandığımız üçüncü taraf hizmet sağlayıcıların
              sunucularına (ABD veya AB ülkeleri) veri iletebilir. Detay için
              ana <Link href="/aydinlatma">Aydınlatma Metni</Link>&apos;ne bakın.
            </p>
          </section>

          <section>
            <h2>İletişim</h2>
            <p>
              Çerezler hakkında soru veya talepleriniz için:{" "}
              <a href="mailto:{{ILETISIM_EMAIL}}">{{ILETISIM_EMAIL}}</a>
            </p>
          </section>

          <section>
            <h2>Değişiklikler</h2>
            <p>
              Bu politika güncellenebilir. Güncel sürüm her zaman bu sayfada
              yayınlanır.
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

### `{{CEREZ_LISTESI}}`

Bir tablo. Tespit edilen her çerez/storage için satır:

```html
<div className="overflow-x-auto">
  <table className="cerez-tablo">
    <thead>
      <tr>
        <th>İsim</th>
        <th>Türü</th>
        <th>Amaç</th>
        <th>Süre</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><code>sb-auth-token</code></td>
        <td>Kesinlikle gerekli</td>
        <td>Supabase oturum yönetimi</td>
        <td>Oturum süresi</td>
      </tr>
      <tr>
        <td><code>cookie-consent</code></td>
        <td>Kesinlikle gerekli</td>
        <td>Çerez tercihi (localStorage)</td>
        <td>Kalıcı (kullanıcı silene kadar)</td>
      </tr>
      <tr>
        <td><code>_vercel_*</code></td>
        <td>Analitik</td>
        <td>Vercel Analytics — sayfa görüntüleme istatistiği</td>
        <td>1 yıl</td>
      </tr>
    </tbody>
  </table>
</div>
```

CSS:
```css
.cerez-tablo {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
}
.cerez-tablo th,
.cerez-tablo td {
  text-align: left;
  padding: 0.6rem 0.8rem;
  border-bottom: 1px solid rgba(180, 180, 180, 0.15);
}
.cerez-tablo th {
  font-weight: 500;
  opacity: 0.75;
  font-size: 0.7rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}
.cerez-tablo code {
  font-family: var(--font-mono, ui-monospace, monospace);
  font-size: 0.78rem;
  opacity: 0.9;
}
```

### `{{ANALITIK_BLOK}}` örnekleri

Vercel Analytics varsa:
```html
<p>
  <strong>Vercel Analytics</strong> — sayfa görüntüleme sayısı, ortalama
  süre gibi anonim metrikleri toplar. Kullanıcıyı tanımlayan bir veri
  toplamaz. Vercel Inc. (ABD) tarafından işletilir.{" "}
  <a href="https://vercel.com/legal/privacy-policy" target="_blank" rel="noopener">
    Vercel gizlilik politikası
  </a>
</p>
```

Google Analytics varsa:
```html
<p>
  <strong>Google Analytics</strong> — ziyaret istatistiklerini analiz etmek
  için Google LLC (ABD) tarafından sunulan hizmet. IP adresinizi (kısaltılmış
  olarak), tarayıcı türünüzü, ziyaret süresini toplar. Google'ın gizlilik
  politikası ve verilerin işlenmesine ilişkin detaylar:{" "}
  <a href="https://policies.google.com/privacy" target="_blank" rel="noopener">
    policies.google.com/privacy
  </a>. Bu çerezler ABD'ye aktarılır (KVKK md. 9 kapsamında yurt dışı
  aktarım), açık rızanızla.
</p>
```

PostHog varsa:
```html
<p>
  <strong>PostHog</strong> — kullanıcı davranış analizi için. AB veya ABD
  sunucularında saklanır (proje yapılandırmasına göre).{" "}
  <a href="https://posthog.com/privacy" target="_blank" rel="noopener">
    posthog.com/privacy
  </a>
</p>
```

---

*Bu şablon hukuki tavsiye değildir. Canlıya çıkmadan önce avukatla incelenmesi önerilir.*
