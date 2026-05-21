---
template: cayma-hakki
language: tr
legal_basis:
  - 6502_tuketici_md_48_4
  - 6502_tuketici_md_15
  - mesafeli_sozlesmeler_yon
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "14 gün süre + istisnalar listesi + form linki"
---

# Cayma Hakkı Şablonu

> 6502 sayılı Tüketici K. md. 48/4 kapsamında 14 günlük cayma hakkı sayfası + form.

**Placeholder'lar**:
- `{{SATICI_AD}}`, `{{SATICI_ADRES}}`, `{{SATICI_EMAIL}}`
- `{{PROJE_ADI}}`
- `{{URUN_TURU}}` — fiziksel / dijital / hizmet
- `{{IADE_SURESI}}` — yasal 14 gün ama daha uzunsa yaz

---

## Next.js — `app/cayma-hakki/page.tsx`

```tsx
import { Metadata } from "next";
import Link from "next/link";

export const metadata: Metadata = {
  title: "Cayma Hakkı · {{PROJE_ADI}}",
  description: "14 günlük cayma hakkınızı nasıl kullanırsınız",
};

export default function CaymaHakkiPage() {
  return (
    <main className="prose max-w-3xl mx-auto py-12 px-4">
      <h1>Cayma Hakkı</h1>
      <p className="lead">
        6502 sayılı Tüketici Kanunu uyarınca <strong>14 gün</strong> içinde sebep
        göstermeksizin sözleşmeden cayma hakkınız vardır.
      </p>

      <h2>1. Cayma Hakkı Nedir?</h2>
      <p>
        Mesafeli (uzaktan) sözleşme ile satın aldığınız ürünü/hizmeti **14 gün** içinde
        herhangi bir gerekçe göstermeksizin ve cezai şart ödemeksizin iade
        edebilirsiniz. Bu yasal bir haktır; sözleşme ile kısıtlanamaz.
      </p>

      <h2>2. 14 Günlük Süre Ne Zaman Başlar?</h2>
      <ul>
        <li>
          <strong>Fiziksel ürün</strong>: Ürünün size veya gösterdiğiniz kişiye teslim
          edildiği günden itibaren
        </li>
        <li>
          <strong>Hizmet</strong>: Sözleşmenin imzalandığı (ödeme yapıldığı) günden itibaren
        </li>
        <li>
          <strong>Dijital içerik</strong>: Sözleşmenin imzalandığı günden itibaren
          (ama erişim sağlamadan önce — bkz. istisna 5.1)
        </li>
        <li>
          <strong>Abonelik</strong>: İlk satın alma için 14 gün; yenilemelerde
          cayma süresi yenilenmez
        </li>
      </ul>

      <h2>3. Cayma Hakkını Nasıl Kullanırsınız?</h2>
      <p>3 yol var, herhangi birini kullanabilirsiniz:</p>

      <h3>3.1. Bizim Cayma Formumuzu Doldurun (önerilen)</h3>
      <p>
        <Link href="/iade-iptal">
          Online cayma/iade formu →
        </Link>
      </p>

      <h3>3.2. E-posta gönderin</h3>
      <p>
        <a href="mailto:{{SATICI_EMAIL}}?subject=Cayma Bildirimi">
          {{SATICI_EMAIL}}
        </a>
      </p>
      <p>
        E-postanızda şunları belirtin:
      </p>
      <ul>
        <li>Adınız soyadınız</li>
        <li>Sipariş numarası</li>
        <li>"Sözleşmeden cayıyorum" şeklinde net beyan</li>
        <li>İade için banka/kart bilgisi (genelde aynı kart)</li>
      </ul>

      <h3>3.3. Posta ile yazılı bildirim</h3>
      <p>
        {{SATICI_AD}}<br />
        {{SATICI_ADRES}}
      </p>
      <p>
        14 günlük süre, postaya verme tarihinde başlar (PTT alındı belgesi
        delil niteliğindedir).
      </p>

      <h2>4. Cayma Sonrası Süreç</h2>

      <h3>4.1. Ürünü Geri Gönderme</h3>
      <p>
        {/* Fiziksel ürün için: */}
        Cayma bildiriminden itibaren <strong>10 gün</strong> içinde ürünü tarafımıza
        geri göndermeniz gerekir. Kargo ücreti:
      </p>
      <ul>
        <li>{{KARGO_UCRETI_KURALI}} — örn: "Tüketici tarafından karşılanır" veya "Anlaşmalı kargo firmaları ile ücretsiz"</li>
      </ul>

      <h3>4.2. Ücret İadesi</h3>
      <p>
        Cayma bildiriminin alınmasından itibaren <strong>14 gün</strong> içinde,
        ürün/hizmet bedeli ve teslim masrafları **iade edilir**.
        İade, ödeme yapılan yönteme (kart, banka transferi vb.) yapılır.
      </p>
      <ul>
        <li>Kredi kartı iadeleri: Banka tarafından 1-3 iş günü içinde hesabınıza yansır</li>
        <li>Banka transferi iadeleri: Belirttiğiniz IBAN'a 1-2 iş günü içinde</li>
      </ul>

      <h2>5. Cayma Hakkı İstisnaları (6502 sayılı K. md. 15)</h2>

      <p>Aşağıdaki ürün/hizmetlerde cayma hakkı kullanılamaz:</p>

      <h3>5.1. Açılmış Dijital İçerik</h3>
      <p>
        Açılmış/erişim sağlanmış elektronik kitap, abonelik, yazılım, müzik, video,
        dijital indirme — md. 15/1-ğ kapsamında istisnadır.
        <strong>Önemli</strong>: Sipariş anında bu hakkı kullanma onayını verdiğiniz
        için (consent checkbox), erişim açıldıktan sonra cayma mümkün değildir.
      </p>

      <h3>5.2. Diğer İstisnalar</h3>
      <ul>
        <li>Tüketicinin istek/kişisel ihtiyaçlarına göre hazırlanan, ısmarlama ürünler</li>
        <li>Hızla bozulabilen ürünler (gıda, çiçek, vb.)</li>
        <li>Açılması durumunda iadesi sağlık/hijyen açısından uygun olmayan ürünler</li>
        <li>Açılmış ses/görüntü kayıtları, yazılım programları (fiziksel medya)</li>
        <li>Tarihli gazete, dergi, süreli yayınlar</li>
        <li>Belirli tarihte verilmesi gereken hizmetler (örn. konser, etkinlik bileti)</li>
        <li>Bahis ve piyango hizmetleri</li>
      </ul>

      <h2>6. Sıkça Sorulan Sorular</h2>

      <h3>Ürünü iade ederken faturayı göndermem gerekir mi?</h3>
      <p>
        Evet, eğer satış ile birlikte fatura kesildiyse, fatura aslı veya kopyası
        iade ile birlikte gönderilmelidir.
      </p>

      <h3>Ürünü kullandıysam cayabilir miyim?</h3>
      <p>
        Cayma hakkını kullanırken ürünün satılabilirlik kapasitesini kaybetmemiş
        olması gerekir. Aşırı kullanım/yıpranma sonucu değer kaybı oluşmuşsa,
        Satıcı bu kaybı iade tutarından düşebilir (md. 48/5).
      </p>

      <h3>Cayma hakkım için ücret talep edilebilir mi?</h3>
      <p>
        <strong>Hayır</strong>. Cayma hakkını kullandığınız için size hiçbir ücret/cezai
        şart yansıtılamaz. Yalnızca ürünü iade kargo masrafı sözleşmeye göre
        size düşebilir (bkz. yukarıda 4.1).
      </p>

      <h3>14 gün geçtikten sonra ne olur?</h3>
      <p>
        14 gün geçtiyse ve ürün kusurluysa, **ayıplı mal hükümleri** (6502 sayılı K.
        md. 11) devreye girer. Bu kapsamda 2 yıllık garanti süreniz vardır.
      </p>

      <h2>7. Tüketici Şikayet Mekanizmaları</h2>
      <p>
        Cayma sürecinde sorun yaşarsanız:
      </p>
      <ul>
        <li>
          <strong>Tüketici Hakem Heyeti</strong> — Uyuşmazlık tutarı ~57.000 TL (2024) altında
        </li>
        <li>
          <strong>Tüketici Mahkemesi</strong> — Yukarıdaki sınırın üstünde
        </li>
        <li>
          <strong>e-Devlet "Tüketici Şikayetleri"</strong> — online başvuru
        </li>
      </ul>

      <hr />

      <p className="text-sm opacity-70">
        Bu sayfa 6502 sayılı Tüketici Kanunu kapsamında bilgilendirme amaçlıdır.
        Belirli bir durumunuz için hukuki danışmanlık almanız önerilir.
      </p>

      <p className="text-sm opacity-70">
        Son güncelleme: <time dateTime="{{TARIH}}">{{TARIH}}</time> ·
        Şablon: legal-lens v0.3
      </p>
    </main>
  );
}
```

---

## Cayma Bildirimi Form Şablonu (PDF veya printable)

Site dışında basılabilir bir form sunulması gerekir (Yönetmelik gereği):

```markdown
# Cayma Bildirimi Formu

(Bu formu doldurarak posta, e-posta veya iletişim formu ile gönderebilirsiniz)

---

**Alıcı**: {{SATICI_AD}}
**Adres**: {{SATICI_ADRES}}
**E-posta**: {{SATICI_EMAIL}}

---

Sayın yetkili,

Aşağıda bilgileri yer alan tarafımdan ___ tarihinde satın alınan ürün/hizmet
sözleşmesinden 6502 sayılı Tüketici Kanunu md. 48/4 uyarınca cayıyorum.

**Sipariş Numarası**: _________________
**Sipariş Tarihi**: _________________
**Teslim Tarihi**: _________________
**Ürün/Hizmet Adı**: _________________

**Adım Soyadım**: _________________
**Adresim**: _________________
**Tarih**: _________________
**İmza**: _________________

İade için banka bilgileri (kredi kartına iade için boş bırakabilirsiniz):
**IBAN**: _________________
**Adres**: _________________

İyi günler dilerim.
```

Bu form PDF olarak sunulabilir veya online formla doldurulabilir (`templates/iade-iptal.md`).
