---
template: etbis-bilgi-formu
language: tr
legal_basis:
  - 6563_e_ticaret_k
  - etbis_yon
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 12_months
version: "0.3"
notes: "ETBİS kayıt zorunlu; kayıt için: https://etbis.ticaret.gov.tr"
---

# ETBİS Bilgi Formu Şablonu

> **6563 sayılı Elektronik Ticaret Hakkında Kanun** kapsamında T.C. Ticaret Bakanlığı ETBİS (Elektronik Ticaret Bilgi Sistemi) kaydı bilgi sayfası ve footer notu.

**Placeholder'lar**:
- `{{SATICI_AD}}`, `{{SATICI_VERGI_NO}}`, `{{SATICI_MERSIS}}`
- `{{ETBIS_NO}}` — kayıt tamamlandıktan sonra alınan numara

---

## Next.js — `app/etbis-bilgi/page.tsx` (opsiyonel, footer'a koyman da yeter)

```tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "ETBİS Bilgileri · {{PROJE_ADI}}",
  description: "Elektronik Ticaret Bilgi Sistemi kayıt bilgilerimiz",
};

export default function EtbisBilgiPage() {
  return (
    <main className="prose max-w-2xl mx-auto py-12 px-4">
      <h1>ETBİS Bilgi Formu</h1>
      <p className="lead">
        T.C. Ticaret Bakanlığı Elektronik Ticaret Bilgi Sistemi (ETBİS)
        kapsamındaki kayıt bilgilerimiz.
      </p>

      <h2>İşletme Bilgileri</h2>
      <table>
        <tbody>
          <tr>
            <th>Ticari Unvan</th>
            <td>{{SATICI_AD}}</td>
          </tr>
          <tr>
            <th>Vergi/TC Kimlik Numarası</th>
            <td>{{SATICI_VERGI_NO}}</td>
          </tr>
          <tr>
            <th>Mersis Numarası</th>
            <td>{{SATICI_MERSIS}}</td>
          </tr>
          <tr>
            <th>ETBİS Kayıt Numarası</th>
            <td>{{ETBIS_NO}}</td>
          </tr>
          <tr>
            <th>Vergi Dairesi</th>
            <td>{{VERGI_DAIRESI}}</td>
          </tr>
          <tr>
            <th>Adres</th>
            <td>{{SATICI_ADRES}}</td>
          </tr>
          <tr>
            <th>İletişim</th>
            <td>
              <a href="mailto:{{SATICI_EMAIL}}">{{SATICI_EMAIL}}</a><br />
              {{SATICI_TELEFON}}
            </td>
          </tr>
          <tr>
            <th>Faaliyet Alanı</th>
            <td>{{FAALIYET_ALANI}}</td>
          </tr>
        </tbody>
      </table>

      <h2>ETBİS Nedir?</h2>
      <p>
        Elektronik Ticaret Bilgi Sistemi (ETBİS), T.C. Ticaret Bakanlığı tarafından
        yönetilen, e-ticaret faaliyeti yürüten gerçek ve tüzel kişilerin kayıt
        ve raporlama yaptığı bir sistemdir.
      </p>
      <p>
        Türkiye'de e-ticaret yapan tüm işletmeler, <strong>6563 sayılı Elektronik
        Ticaret Hakkında Kanun</strong> kapsamında ETBİS'e kayıt olmak zorundadır.
      </p>
      <p>
        ETBİS Resmi Sitesi:{" "}
        <a href="https://etbis.ticaret.gov.tr" target="_blank" rel="noopener noreferrer">
          etbis.ticaret.gov.tr
        </a>
      </p>

      <h2>Yasal Yükümlülüklerimiz</h2>
      <ul>
        <li>
          <strong>Mesafeli Satış Sözleşmesi</strong>:{" "}
          <a href="/mesafeli-satis-sozlesmesi">/mesafeli-satis-sozlesmesi</a>
        </li>
        <li>
          <strong>Cayma Hakkı (14 gün)</strong>:{" "}
          <a href="/cayma-hakki">/cayma-hakki</a>
        </li>
        <li>
          <strong>İade ve İptal</strong>:{" "}
          <a href="/iade-iptal">/iade-iptal</a>
        </li>
        <li>
          <strong>KVKK Aydınlatma</strong>:{" "}
          <a href="/aydinlatma">/aydinlatma</a>
        </li>
      </ul>

      <h2>Tüketici Şikayet Mekanizmaları</h2>
      <ul>
        <li>
          <strong>İlk Başvuru</strong>: Tarafımızla iletişime geçin —
          <a href="mailto:{{SATICI_EMAIL}}">{{SATICI_EMAIL}}</a>
        </li>
        <li>
          <strong>Tüketici Hakem Heyeti</strong> (2024 itibariyle ~57.000 TL altı uyuşmazlıklar)
        </li>
        <li>
          <strong>Tüketici Mahkemesi</strong> (yukarıdaki sınır üstü)
        </li>
        <li>
          <strong>e-Devlet Tüketici Şikayetleri Sistemi</strong>
        </li>
      </ul>

      <hr />

      <p className="text-sm opacity-70">
        Son güncelleme: <time dateTime="{{TARIH}}">{{TARIH}}</time>
      </p>
    </main>
  );
}
```

---

## Footer Notu — Daha Hafif Alternatif

ETBİS bilgi sayfası yerine sadece footer'a bilgi yazmak da yeterlidir. `Footer.tsx`'e ekleme:

```tsx
<footer>
  {/* Mevcut linkler */}
  <nav>
    <a href="/mesafeli-satis-sozlesmesi">Mesafeli Satış Sözleşmesi</a>
    <a href="/cayma-hakki">Cayma Hakkı</a>
    <a href="/iade-iptal">İade & İptal</a>
    <a href="/aydinlatma">Aydınlatma</a>
    <a href="/kosullar">Kullanım Koşulları</a>
  </nav>

  {/* ETBİS notu */}
  <div className="etbis-bilgi">
    <p className="text-xs opacity-70">
      <strong>{{SATICI_AD}}</strong> · Vergi/TC: {{SATICI_VERGI_NO}}
      {{SATICI_MERSIS && ` · Mersis: ${SATICI_MERSIS}`}}
      · ETBİS Kayıt No: {{ETBIS_NO}}
      · <a href="/etbis-bilgi">ETBİS Bilgileri</a>
    </p>
  </div>
</footer>
```

---

## Eğer ETBİS Kayıt Yoksa — Süreç Bilgisi

ETBİS kayıt **yapılmadıysa** kullanıcıya **mutlaka** kayıt sürecine yönlendir:

```markdown
# ETBİS Kayıt Süreci — Adım Adım

## 1. Ön Koşullar (öncesinde tamamlanması gereken)

- Şahıs şirketi veya sermaye şirketi (limited/anonim) tescilli olmalı
- Vergi mükellefiyeti açılmış olmalı (vergi dairesinde)
- e-Devlet hesabı + Elektronik İmza (e-İmza) hazır olmalı

## 2. Kayıt Adımları

1. https://etbis.ticaret.gov.tr adresine git
2. "Kayıt" → işletme türünüze göre seçim (Mağaza / Pazar Yeri / Hizmet Sağlayıcı)
3. e-Devlet ile giriş yap
4. İşletme bilgilerini doldur (otomatik MERSIS entegrasyonu)
5. Faaliyet alanı/kapsam seçimi
6. E-İmza ile başvuruyu onayla
7. Onay süreci 1-3 iş günü
8. Onay sonrası ETBİS Kayıt Numarası verilir → bu numarayı footer'a ve mesafeli satış sözleşmesine ekle

## 3. Yıllık Yenileme

- ETBİS kaydı yıllık olarak yenilenmelidir
- Yıllık cironuza göre kademeli raporlama yükümlülükleri vardır

## 4. Yardım

- ETBİS Help Desk: helpdesk@ticaret.gov.tr
- Mali müşavir / SMMM: bu süreçte profesyonel destek alınması önerilir
- Avukat: ETBİS kayıt + KVKK + e-Ticaret K. uyum için bütünsel review
```

---

## Önemli notlar

1. **Mali müşavir/SMMM zorunlu** — vergisel yönleri için
2. **Avukat review önerilir** — ETBİS + KVKK + Tüketici K. uyum için
3. **ETBİS kayıt numarası footer'da bulunmalı** — yasal şart; eksikse kullanıcı şikayetinde delil olarak gösterilebilir
4. **Yıllık ciro raporu** — belirli ciro üstü işletmeler için yıllık raporlama zorunlu
5. **Bu şablon kayıt süreci için klavuzdur** — gerçek başvuru ETBİS sisteminden yapılır
