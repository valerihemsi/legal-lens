---
template: mesafeli-satis
language: tr
legal_basis:
  - 6502_tuketici_md_48
  - mesafeli_sozlesmeler_yon
  - 6563_e_ticaret_k
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "Şablon boilerplate'dir. Ticari yapıya göre mali müşavir + avukat review zorunlu"
---

# Mesafeli Satış Sözleşmesi Şablonu

> 6502 sayılı Tüketici K. md. 48 + Mesafeli Sözleşmeler Yönetmeliği kapsamında zorunlu sözleşme. Pre-checkout consent checkbox ile birlikte kullanılır.

**Placeholder'lar**:
- `{{SATICI_AD}}` — ticari unvan veya gerçek kişi adı
- `{{SATICI_ADRES}}` — tam adres
- `{{SATICI_VERGI_NO}}` — Vergi/TC numarası
- `{{SATICI_MERSIS}}` — Mersis numarası (varsa)
- `{{SATICI_TELEFON}}`
- `{{SATICI_EMAIL}}`
- `{{ETBIS_NO}}` — ETBİS kayıt numarası
- `{{URUN_TURU}}` — fiziksel / dijital / hizmet
- `{{TESLIMAT_SURESI}}` — ortalama gün

---

## Next.js — `app/mesafeli-satis-sozlesmesi/page.tsx`

```tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "Mesafeli Satış Sözleşmesi · {{PROJE_ADI}}",
  description: "6502 sayılı Tüketici Kanunu kapsamında mesafeli satış sözleşmesi",
};

export default function MesafeliSatisPage() {
  return (
    <main className="prose max-w-3xl mx-auto py-12 px-4">
      <h1>Mesafeli Satış Sözleşmesi</h1>
      <p>Son güncelleme: <time dateTime="{{TARIH}}">{{TARIH}}</time></p>

      <h2>1. Taraflar</h2>

      <h3>1.1. Satıcı</h3>
      <ul>
        <li><strong>Unvan</strong>: {{SATICI_AD}}</li>
        <li><strong>Adres</strong>: {{SATICI_ADRES}}</li>
        <li><strong>Vergi/TC</strong>: {{SATICI_VERGI_NO}}</li>
        <li><strong>Mersis</strong>: {{SATICI_MERSIS}}</li>
        <li><strong>Telefon</strong>: {{SATICI_TELEFON}}</li>
        <li><strong>E-posta</strong>: <a href="mailto:{{SATICI_EMAIL}}">{{SATICI_EMAIL}}</a></li>
        <li><strong>ETBİS Kayıt No</strong>: {{ETBIS_NO}}</li>
      </ul>

      <h3>1.2. Alıcı (Tüketici)</h3>
      <p>
        İşbu sözleşmeyi kabul ederek mesafeli satış işlemini onaylayan,
        satın alma sırasında bilgileri kaydedilen gerçek kişi.
      </p>

      <h2>2. Sözleşmenin Konusu</h2>
      <p>
        Alıcının Satıcıdan elektronik ortamda satın aldığı, aşağıda nitelikleri ve
        satış fiyatı belirtilen ürün/hizmetin satışı ve teslimi ile tarafların
        hak ve yükümlülüklerinin saptanmasıdır.
      </p>

      <h2>3. Ürün/Hizmet Bilgileri</h2>
      <p>
        Sipariş anında görüntülenen ürün/hizmet bilgileri, sipariş onay e-postasında
        ve hesap panelindeki "Siparişlerim" bölümünde mevcuttur.
      </p>

      <h2>4. Satış Bedeli ve Ödeme</h2>
      <p>
        Satış bedeli, vergiler dahil, sipariş özetinde gösterilen tutardır. Ödeme:
      </p>
      <ul>
        <li>Kredi/banka kartı ile güvenli ödeme sağlayıcı üzerinden ({{ODEME_SAGLAYICI}})</li>
        <li>{{DIGER_ODEME_YONTEMLERI}}</li>
      </ul>

      <h2>5. Teslimat</h2>
      <p>
        {/* Fiziksel ürün için: */}
        Sipariş onaylandıktan sonra ortalama <strong>{{TESLIMAT_SURESI}} iş günü</strong>
        içinde Alıcının belirttiği adrese kargo ile teslim edilir.
        Teslimat süresi, kargo şirketinden kaynaklanan gecikmelerden etkilenebilir.

        {/* Dijital ürün için: */}
        Sipariş onaylandıktan sonra ürüne erişim **anında** açılır;
        hesap panelinden veya e-posta ile gönderilen link üzerinden erişebilirsiniz.

        {/* Hizmet için: */}
        Hizmet, sipariş tarihinden itibaren {{HIZMET_BASLAMA_SURESI}} içinde başlatılır.
      </p>

      <h2>6. Cayma Hakkı (KRİTİK — Aşağıdaki istisnalar dışında)</h2>
      <p>
        Tüketici, sözleşmenin imzalandığı tarihten itibaren <strong>14 (on dört) gün</strong>
        içinde herhangi bir gerekçe göstermeksizin ve cezai şart ödemeksizin sözleşmeden
        cayma hakkına sahiptir (6502 sayılı K. md. 48/4).
      </p>
      <p>
        Detay ve cayma akışı: <a href="/cayma-hakki">/cayma-hakki</a>
      </p>

      <h3>6.1. Cayma hakkı istisnaları (6502 sayılı K. md. 15)</h3>
      <p>Aşağıdaki ürünlerde cayma hakkı kullanılamaz:</p>
      <ul>
        <li>Tüketicinin istekleri/kişisel ihtiyaçları doğrultusunda hazırlanan, özel ısmarlama ürünler</li>
        <li>Hızla bozulabilen veya son kullanma tarihi geçecek ürünler</li>
        <li>Açılması durumunda iadesi sağlık ve hijyen açısından uygun olmayan ürünler (örn: kişisel hijyen)</li>
        <li>Açılmış ses ve görüntü kayıtları, yazılım programları (CD/DVD vb.)</li>
        <li>{/* Dijital içerik için: */} <strong>Açılmış/erişim sağlanmış dijital içerik</strong> (Md. 15/1-ğ) — Alıcı sipariş onayında bu hakkını kullanmayı kabul etmiş sayılır</li>
        <li>Tarihi gazete, dergi ve süreli yayınlar</li>
        <li>Belirli bir tarihte verilmesi gereken hizmetler ({{HIZMET_TARIHI_VARSA}})</li>
      </ul>

      <h2>7. Tüketicinin Şikayet ve Çözüm Mekanizmaları</h2>
      <ul>
        <li>
          <strong>İlk başvuru</strong>: <a href="mailto:{{SATICI_EMAIL}}?subject=Şikayet">{{SATICI_EMAIL}}</a>
          — 30 gün içinde cevap verilir
        </li>
        <li>
          <strong>Tüketici Hakem Heyeti</strong> — Uyuşmazlık tutarı 2024 itibariyle
          ~57.000 TL'nin altında ise yerleşim yerinizdeki İlçe/İl Tüketici Hakem Heyeti
        </li>
        <li>
          <strong>Tüketici Mahkemesi</strong> — Yukarıdaki sınırın üstündeki uyuşmazlıklar için
        </li>
        <li>
          <strong>e-Devlet</strong> üzerinden Tüketici Şikayetleri sistemine başvuru
        </li>
      </ul>

      <h2>8. KVKK Kapsamında Veri İşleme</h2>
      <p>
        Satın alma sürecinde işlenen kişisel veriler için bkz.:{" "}
        <a href="/aydinlatma">Aydınlatma Metni</a>
      </p>

      <h2>9. Genel Hükümler</h2>
      <ul>
        <li>Sözleşme metni Türkçe'dir; uyuşmazlıklarda Türkçe metin esastır</li>
        <li>İşbu sözleşme Türkiye Cumhuriyeti yasalarına tabidir</li>
        <li>Yetkili mahkeme ve icra daireleri: {{SATICI_SEHIR}} mahkemeleri ve icra daireleri</li>
        <li>Sözleşmenin bir hükmünün geçersizliği diğer hükümlerin geçerliliğini etkilemez</li>
        <li>Satıcı işbu sözleşmeyi tek taraflı değiştirme hakkını saklı tutar; değişiklik ileri tarihli işlemler için geçerli olur</li>
      </ul>

      <h2>10. Kabul</h2>
      <p>
        İşbu sözleşmeyi pre-checkout aşamasında kabul ettiğinizi belirten checkbox'ı
        işaretlemekle, Alıcı sözleşme hükümlerini tam, açık ve özgür iradesiyle kabul
        etmiş sayılır.
      </p>

      <p className="text-sm opacity-70 mt-12">
        Şablon: legal-lens v0.3 · Bu metin hukuki tavsiye değildir.
      </p>
    </main>
  );
}
```

### Checkout flow'da kabul checkbox

```tsx
// Pre-checkout consent
import { useState } from "react";

export function CheckoutConsent({ onConsent }: { onConsent: (ok: boolean) => void }) {
  const [accepted, setAccepted] = useState(false);

  return (
    <div className="consent-block">
      <label>
        <input
          type="checkbox"
          checked={accepted}
          onChange={(e) => {
            setAccepted(e.target.checked);
            onConsent(e.target.checked);
          }}
        />
        {" "}
        <a href="/mesafeli-satis-sozlesmesi" target="_blank" rel="noopener noreferrer">
          Mesafeli Satış Sözleşmesi
        </a>'ni ve{" "}
        <a href="/cayma-hakki" target="_blank" rel="noopener noreferrer">
          Cayma Hakkı bilgilerini
        </a>{" "}
        okudum, kabul ediyorum.
      </label>
    </div>
  );
}
```

---

## Önemli notlar

1. **Bu şablon boilerplate'tir** — ticari yapıya (B2C, B2B, marketplace) göre özelleştirilmeli
2. **Mali müşavir/SMMM zorunlu**: vergi mükellefiyeti, KDV durumu, fatura akışı için
3. **Avukat review zorunlu**: yıllık ciro >X (örn. 500K TL) veya uluslararası satış varsa
4. **ETBİS kayıt no.** alanı boş bırakılırsa kullanıcıya **kayıt süreci** bilgisi ver (etbis.ticaret.gov.tr)
5. **Pre-checkout consent** olmadan satış **geçersiz** sayılabilir — checkbox eksiksiz kontrol et
