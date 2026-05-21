---
template: iade-iptal
language: tr
legal_basis:
  - 6502_tuketici_md_48
  - mesafeli_sozlesmeler_yon
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "Cayma + ayıplı mal + iptal akışı tek sayfa"
---

# İade & İptal Akışı Şablonu

> Cayma hakkı + ayıplı mal + sipariş iptal akışını tek bir formda toplayan operasyonel sayfa.

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{SATICI_EMAIL}}`
- `{{IADE_SURESI_GUN}}` — yasal 14 gün; daha uzun süre tanıyorsan o
- `{{KARGO_FIRMASI}}` — anlaşmalı kargo firmaları (varsa)

---

## Next.js — `app/iade-iptal/page.tsx`

```tsx
"use client";

import { useState } from "react";

type IadeNedeni = "cayma" | "ayipli" | "yanlis_urun" | "diger";

interface IadeFormu {
  siparis_no: string;
  email: string;
  ad: string;
  neden: IadeNedeni;
  aciklama: string;
  iban: string;
  fotograf: File | null;
}

export default function IadeIptalPage() {
  const [form, setForm] = useState<IadeFormu>({
    siparis_no: "",
    email: "",
    ad: "",
    neden: "cayma",
    aciklama: "",
    iban: "",
    fotograf: null,
  });
  const [submitting, setSubmitting] = useState(false);
  const [done, setDone] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setSubmitting(true);
    try {
      const fd = new FormData();
      Object.entries(form).forEach(([k, v]) => {
        if (v instanceof File) fd.append(k, v);
        else fd.append(k, String(v));
      });
      const res = await fetch("/api/iade-iptal", { method: "POST", body: fd });
      if (res.ok) setDone(true);
      else alert("Bir hata oluştu. Lütfen tekrar deneyin veya e-posta ile iletişime geçin.");
    } finally {
      setSubmitting(false);
    }
  }

  if (done) {
    return (
      <main className="prose max-w-3xl mx-auto py-12 px-4">
        <h1>Talebiniz Alındı</h1>
        <p>
          Talebinizi inceleyip 30 gün içinde dönüş yapacağız. Onay e-postası gönderildi.
          Acil durum için: <a href="mailto:{{SATICI_EMAIL}}">{{SATICI_EMAIL}}</a>
        </p>
      </main>
    );
  }

  return (
    <main className="prose max-w-3xl mx-auto py-12 px-4">
      <h1>İade / İptal Talebi</h1>
      <p className="lead">
        Sipariş iadesi, sözleşmeden cayma veya ayıplı mal bildirimi için bu formu doldurun.
        Daha fazla bilgi: <a href="/cayma-hakki">/cayma-hakki</a>
      </p>

      <form onSubmit={handleSubmit} className="iade-form" encType="multipart/form-data">
        <label>
          Sipariş Numarası *
          <input
            type="text"
            required
            value={form.siparis_no}
            onChange={(e) => setForm({ ...form, siparis_no: e.target.value })}
          />
        </label>

        <label>
          E-posta (sipariş anındaki) *
          <input
            type="email"
            required
            value={form.email}
            onChange={(e) => setForm({ ...form, email: e.target.value })}
          />
        </label>

        <label>
          Adınız Soyadınız *
          <input
            type="text"
            required
            value={form.ad}
            onChange={(e) => setForm({ ...form, ad: e.target.value })}
          />
        </label>

        <label>
          Talep Nedeni *
          <select
            value={form.neden}
            onChange={(e) => setForm({ ...form, neden: e.target.value as IadeNedeni })}
          >
            <option value="cayma">Cayma hakkı (14 gün içinde, sebep gerekmez)</option>
            <option value="ayipli">Ayıplı mal (ürün kusurlu/eksik)</option>
            <option value="yanlis_urun">Yanlış ürün gönderildi</option>
            <option value="diger">Diğer (açıklayın)</option>
          </select>
        </label>

        <label>
          Açıklama
          <textarea
            rows={4}
            value={form.aciklama}
            onChange={(e) => setForm({ ...form, aciklama: e.target.value })}
            placeholder={form.neden === "cayma" ? "Cayma için sebep gerekmez" : "Lütfen detay verin"}
          />
        </label>

        {(form.neden === "ayipli" || form.neden === "yanlis_urun") && (
          <label>
            Fotoğraf (kusur/yanlış ürün için)
            <input
              type="file"
              accept="image/*"
              onChange={(e) => setForm({ ...form, fotograf: e.target.files?.[0] || null })}
            />
          </label>
        )}

        <label>
          IBAN (kredi kartı iadeleri için boş bırakın)
          <input
            type="text"
            value={form.iban}
            onChange={(e) => setForm({ ...form, iban: e.target.value })}
            placeholder="TR..."
          />
        </label>

        <button type="submit" disabled={submitting}>
          {submitting ? "Gönderiliyor..." : "Talebi Gönder"}
        </button>
      </form>

      <h2>Süreç Nasıl İşler?</h2>
      <ol>
        <li><strong>Talep alımı</strong> — Formu doldurun, otomatik onay e-postası alın</li>
        <li><strong>İnceleme</strong> — Talebiniz 1-3 iş günü içinde değerlendirilir</li>
        <li><strong>Onay + Kargo</strong> — Onaylandıktan sonra ürünü iade için kargo bilgileri size iletilir (fiziksel ürün için)</li>
        <li><strong>Ürünün ulaşması</strong> — 10 gün içinde ürünü kargo ile gönderin</li>
        <li><strong>Ücret iadesi</strong> — Ürün ulaştıktan sonra <strong>14 gün</strong> içinde iade yapılır</li>
      </ol>

      <h2>Önemli Notlar</h2>
      <ul>
        <li>Cayma hakkı için yasal süre 14 gündür ({{IADE_SURESI_GUN}})</li>
        <li>Ayıplı mal için 2 yıl garanti süresi vardır</li>
        <li>Bu sayfa istisnalar listesini içermez — bkz. <a href="/cayma-hakki">/cayma-hakki</a></li>
        <li>Talebiniz Tüketici Hakem Heyeti/Mahkemesi'ne taşınmadan önce 30 günlük cevap süremiz vardır</li>
      </ul>

      <hr />

      <p className="text-sm opacity-70">
        Form sonuçları KVKK kapsamında işlenir; detay: <a href="/aydinlatma">aydınlatma metni</a>
      </p>
    </main>
  );
}
```

---

## API endpoint — `app/api/iade-iptal/route.ts`

```tsx
import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  const formData = await req.formData();
  const data = {
    siparis_no: String(formData.get("siparis_no") || ""),
    email: String(formData.get("email") || ""),
    ad: String(formData.get("ad") || ""),
    neden: String(formData.get("neden") || ""),
    aciklama: String(formData.get("aciklama") || ""),
    iban: String(formData.get("iban") || ""),
    // fotograf File olarak işlenir, storage'a yüklenir
  };

  // 1. DB'ye kaydet (örneğin Supabase)
  //    iade_iptal_talepleri tablosu: id, siparis_no, email, ad, neden, aciklama, iban, foto_url, durum, created_at
  // 2. Kullanıcıya onay e-postası gönder
  // 3. Admin'e bildirim e-postası gönder

  // Örnek (pseudo):
  // const { data: kayit, error } = await supabase.from("iade_iptal_talepleri").insert(data).select().single();
  // if (error) return NextResponse.json({ error: "Kayıt başarısız" }, { status: 500 });
  // await resendEmail({ to: data.email, subject: "İade talebiniz alındı", ... });
  // await resendEmail({ to: "admin@firma.com", subject: "Yeni iade talebi", ... });

  return NextResponse.json({ success: true });
}
```

**Schema (Supabase migration örneği)**:

```sql
create table public.iade_iptal_talepleri (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id) on delete set null,
  siparis_no text not null,
  email text not null,
  ad text not null,
  neden text not null check (neden in ('cayma','ayipli','yanlis_urun','diger')),
  aciklama text,
  iban text,
  foto_url text,
  durum text default 'inceleniyor' check (durum in ('inceleniyor','onaylandi','reddedildi','tamamlandi')),
  reddedilme_nedeni text,
  yanit_tarihi timestamp with time zone,
  created_at timestamp with time zone default now()
);

-- 30 gün cevap zorunluluğu — view ile takip
create view public.iade_iptal_geciken as
  select * from public.iade_iptal_talepleri
  where durum = 'inceleniyor'
    and created_at < now() - interval '25 days';
```

---

## CSS (globals.css'e ekle)

```css
.iade-form {
  display: grid;
  gap: 1rem;
  max-width: 600px;
}

.iade-form label {
  display: grid;
  gap: 0.25rem;
  font-weight: 500;
}

.iade-form input,
.iade-form select,
.iade-form textarea {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 1rem;
}

.iade-form button {
  background: #2563eb;
  color: white;
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
}

.iade-form button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
```

---

## Önemli notlar

1. **30 gün cevap süresi** — Yasal yükümlülük; geciken talepler için DB view ile takip kurulmalı
2. **Admin paneli** — İade talepleri için admin görüntüleme/yanıt sayfası ayrı (bu şablonun kapsamı dışı)
3. **Kargo entegrasyonu** — Anlaşmalı kargo firmaları varsa onların API'leri bu akışa eklenebilir
4. **Fotograf storage** — Ayıplı mal şikayetinde fotoğraf saklama için ayrı bucket (S3/Supabase Storage)
5. **Tüketici Hakem Heyeti** — 30 gün geçince + olumsuz cevap durumunda kullanıcı bu yola başvurabilir
