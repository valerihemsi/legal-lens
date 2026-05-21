---
template: kvkk-md11-rights
language: tr
legal_basis:
  - kvkk_md_11
  - kvkk_md_13
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "Md. 13 — başvurulara 30 gün içinde cevap zorunlu"
---

# KVKK Md. 11 — Veri Sahibi Hakları İçin Kod Şablonları

Veri saklayan her uygulamada, KVKK md. 11 hakları için **çalışan bir mekanizma** gerekli. Sadece "iletişim email'imize yazın" çoğu durumda yetmez; kullanıcının kendi hesabından erişim/silme yapabilmesi en az dirençli yol.

Bu şablon:
1. **`/hesabim` (Account) sayfası** — kullanıcının kendi verisini görüp yönetebileceği panel
2. **Veri dışa aktarma** (JSON export) — taşınabilirlik + bilgi talebi (md. 11/b)
3. **Hesap silme** — silme talebi (md. 11/e)
4. **Düzeltme** — kullanıcı profilinden inline (md. 11/d)

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{ILETISIM_EMAIL}}`
- `{{KULLANICI_TABLOLARI}}` — silinmesi/aktarılması gereken tablo isimleri (Claude tespit eder)

---

## 1. `/hesabim` sayfası — Generic React + Supabase

`app/hesabim/page.tsx`:

```tsx
"use client";

import { useEffect, useState } from "react";
import Link from "next/link";
import { createClient } from "@/lib/supabase/client"; // veya projenizdeki supabase wrapper

interface KullaniciProfili {
  id: string;
  email: string;
  username?: string;
  display_name?: string;
  created_at: string;
}

export default function HesabimPage() {
  const [profil, setProfil] = useState<KullaniciProfili | null>(null);
  const [yukleniyor, setYukleniyor] = useState(true);
  const [silmeOnayi, setSilmeOnayi] = useState(false);
  const [silmeMetni, setSilmeMetni] = useState("");
  const [silmeIslemde, setSilmeIslemde] = useState(false);
  const [hata, setHata] = useState<string | null>(null);
  const supabase = createClient();

  useEffect(() => {
    const yukle = async () => {
      const { data: { user } } = await supabase.auth.getUser();
      if (!user) {
        window.location.href = "/giris";
        return;
      }
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("id", user.id)
        .single();
      if (error) {
        setHata("Profil yüklenemedi");
      } else {
        setProfil({ ...data, email: user.email ?? "" });
      }
      setYukleniyor(false);
    };
    yukle();
  }, [supabase]);

  const veriExport = async () => {
    setHata(null);
    try {
      const res = await fetch("/api/me/export");
      if (!res.ok) throw new Error("Dışa aktarma başarısız");
      const blob = await res.blob();
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = `{{PROJE_ADI}}-verilerim-${new Date().toISOString().split("T")[0]}.json`;
      a.click();
      URL.revokeObjectURL(url);
    } catch (err) {
      setHata(err instanceof Error ? err.message : "Bir hata oluştu");
    }
  };

  const hesapSil = async () => {
    if (silmeMetni !== "SİL") {
      setHata("Onay metni eşleşmiyor");
      return;
    }
    setSilmeIslemde(true);
    setHata(null);
    try {
      const res = await fetch("/api/me", { method: "DELETE" });
      if (!res.ok) {
        const body = await res.json().catch(() => ({}));
        throw new Error(body.error ?? "Silme başarısız");
      }
      await supabase.auth.signOut();
      window.location.href = "/?silindi=1";
    } catch (err) {
      setHata(err instanceof Error ? err.message : "Bir hata oluştu");
      setSilmeIslemde(false);
    }
  };

  if (yukleniyor) {
    return <main className="min-h-screen flex items-center justify-center">Yükleniyor...</main>;
  }

  if (!profil) {
    return <main className="min-h-screen flex items-center justify-center">Profil bulunamadı</main>;
  }

  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-2xl mx-auto space-y-12">
        <header>
          <h1 className="text-3xl font-light tracking-wide">Hesabım</h1>
          <p className="text-sm opacity-70 mt-2">
            KVKK md. 11 kapsamındaki haklarını buradan kullanabilirsin.
          </p>
        </header>

        {/* Bilgi görüntüleme */}
        <section className="space-y-3">
          <h2 className="text-lg font-medium">Hesap bilgileri</h2>
          <div className="bilgi-satiri">
            <span className="bilgi-etiket">Email</span>
            <span>{profil.email}</span>
          </div>
          {profil.username && (
            <div className="bilgi-satiri">
              <span className="bilgi-etiket">Kullanıcı adı</span>
              <span>{profil.username}</span>
            </div>
          )}
          <div className="bilgi-satiri">
            <span className="bilgi-etiket">Kayıt tarihi</span>
            <span>{new Date(profil.created_at).toLocaleDateString("tr-TR")}</span>
          </div>
        </section>

        {/* Veri dışa aktarma */}
        <section className="border-t pt-8">
          <h2 className="text-lg font-medium mb-2">Verilerimi indir</h2>
          <p className="text-sm opacity-75 mb-4">
            Hesabınla ilişkili tüm verileri JSON formatında indirebilirsin. Bu
            dosya başka bir platforma taşımak için kullanılabilir veya kişisel
            arşivin olabilir.
          </p>
          <button
            type="button"
            onClick={veriExport}
            className="haklar-buton"
          >
            JSON olarak indir
          </button>
        </section>

        {/* Düzeltme talebi */}
        <section className="border-t pt-8">
          <h2 className="text-lg font-medium mb-2">Bilgilerimi düzelt</h2>
          <p className="text-sm opacity-75 mb-4">
            Eksik veya yanlış bilgi varsa{" "}
            <Link href="/hesabim/duzenle" className="link-altCizgi">
              profil düzenleme sayfasından
            </Link>{" "}
            güncelleyebilirsin. Eğer email'in değişmesi gerekiyorsa lütfen{" "}
            <a href="mailto:{{ILETISIM_EMAIL}}" className="link-altCizgi">
              {{ILETISIM_EMAIL}}
            </a>{" "}
            adresine yazarak iletişime geç.
          </p>
        </section>

        {/* Hesap silme — TEHLİKELİ BÖLGE */}
        <section className="border-t pt-8 tehlike-bolge">
          <h2 className="text-lg font-medium mb-2 text-rose-300">Hesabımı sil</h2>
          <p className="text-sm opacity-75 mb-4">
            Hesabını ve hesabınla ilişkili tüm verileri kalıcı olarak silersin.
            Bu işlem <strong>geri alınamaz</strong>. Hesabın silindikten sonra:
          </p>
          <ul className="text-sm opacity-75 mb-4 space-y-1 list-disc list-inside">
            <li>Email/şifre ile giriş yapamazsın</li>
            <li>Paylaşımların, oyların, yorumların kalıcı silinir</li>
            <li>Diğer kullanıcıların gönderdiği mesajlar (sizden gelen) silinir</li>
            <li>30 gün içinde aynı email ile yeniden kayıt olabilirsin</li>
          </ul>

          {!silmeOnayi ? (
            <button
              type="button"
              onClick={() => setSilmeOnayi(true)}
              className="haklar-buton haklar-buton-tehlike"
            >
              Hesabımı silmek istiyorum
            </button>
          ) : (
            <div className="silme-onay-kutu space-y-3">
              <p className="text-sm">
                Onaylamak için aşağıdaki kutucuğa{" "}
                <strong className="text-rose-300">SİL</strong> yaz:
              </p>
              <input
                type="text"
                value={silmeMetni}
                onChange={(e) => setSilmeMetni(e.target.value)}
                placeholder="SİL"
                disabled={silmeIslemde}
                className="silme-input"
              />
              <div className="flex gap-3">
                <button
                  type="button"
                  onClick={() => {
                    setSilmeOnayi(false);
                    setSilmeMetni("");
                    setHata(null);
                  }}
                  disabled={silmeIslemde}
                  className="haklar-buton haklar-buton-soluk"
                >
                  Vazgeç
                </button>
                <button
                  type="button"
                  onClick={hesapSil}
                  disabled={silmeIslemde || silmeMetni !== "SİL"}
                  className="haklar-buton haklar-buton-tehlike"
                >
                  {silmeIslemde ? "Siliniyor..." : "Hesabımı kalıcı olarak sil"}
                </button>
              </div>
            </div>
          )}

          {hata && (
            <p className="text-rose-300 text-sm mt-3 italic">{hata}</p>
          )}
        </section>

        {/* Diğer KVKK hakları */}
        <section className="border-t pt-8">
          <h2 className="text-lg font-medium mb-2">Diğer talepler</h2>
          <p className="text-sm opacity-75">
            Yukarıdaki butonlarla karşılanmayan KVKK md. 11 talepleri için
            (örn. otomatik karara itiraz, tazminat talebi) lütfen{" "}
            <a href="mailto:{{ILETISIM_EMAIL}}" className="link-altCizgi">
              {{ILETISIM_EMAIL}}
            </a>{" "}
            adresine yazarak iletişime geç. Talebine 30 gün içinde yanıt verilir.
          </p>
        </section>
      </div>
    </main>
  );
}
```

### CSS yardımcıları

```css
.bilgi-satiri {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.6rem 0;
  border-bottom: 1px solid rgba(180, 180, 180, 0.08);
}

.bilgi-etiket {
  font-size: 0.7rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  opacity: 0.6;
}

.haklar-buton {
  padding: 0.65rem 1.2rem;
  font-size: 0.78rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  border: 1px solid currentColor;
  border-radius: 0.3rem;
  background: transparent;
  cursor: pointer;
  transition: all 0.3s ease;
}

.haklar-buton:hover {
  background: rgba(255, 255, 255, 0.05);
}

.haklar-buton:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.haklar-buton-tehlike {
  color: rgb(252, 165, 165);
  border-color: rgb(220, 38, 38);
}

.haklar-buton-tehlike:hover {
  background: rgba(220, 38, 38, 0.08);
}

.haklar-buton-soluk {
  opacity: 0.6;
}

.tehlike-bolge {
  border-color: rgba(220, 38, 38, 0.25);
}

.silme-onay-kutu {
  padding: 1rem;
  border: 1px solid rgba(220, 38, 38, 0.3);
  border-radius: 0.5rem;
  background: rgba(220, 38, 38, 0.05);
}

.silme-input {
  width: 100%;
  padding: 0.6rem 0.9rem;
  background: transparent;
  border: 1px solid rgba(180, 180, 180, 0.25);
  border-radius: 0.3rem;
  font-family: inherit;
  font-size: 1rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
}

.silme-input:focus {
  outline: none;
  border-color: rgb(220, 38, 38);
}

.link-altCizgi {
  text-decoration: underline;
  text-underline-offset: 2px;
}
```

---

## 2. Veri Dışa Aktarma API — Supabase

`app/api/me/export/route.ts`:

```ts
import { NextRequest } from "next/server";
import { createClient } from "@/lib/supabase/server";

export async function GET(req: NextRequest) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    return new Response(
      JSON.stringify({ error: "Giriş yapmamışsınız" }),
      { status: 401, headers: { "Content-Type": "application/json" } }
    );
  }

  // Kullanıcının tüm verisini topla
  const [
    profilRes,
    experimentsRes,
    votesRes,
    savesRes,
    followsRes,
    messagesRes,
  ] = await Promise.all([
    supabase.from("profiles").select("*").eq("id", user.id).single(),
    supabase.from("experiments").select("*").eq("user_id", user.id),
    supabase.from("votes").select("*").eq("user_id", user.id),
    supabase.from("saves").select("*").eq("user_id", user.id),
    supabase.from("follows").select("*").or(`follower_id.eq.${user.id},following_id.eq.${user.id}`),
    supabase.from("messages").select("*").or(`sender_id.eq.${user.id},receiver_id.eq.${user.id}`),
  ]);

  const ihracat = {
    meta: {
      proje: "{{PROJE_ADI}}",
      export_tarihi: new Date().toISOString(),
      kullanici_id: user.id,
      email: user.email,
      not: "Bu dosya KVKK md. 11/b kapsamında oluşturulan kişisel veri ihracatınızdır.",
    },
    profil: profilRes.data,
    deneyler: experimentsRes.data ?? [],
    oylar: votesRes.data ?? [],
    kayitlar: savesRes.data ?? [],
    takipler: followsRes.data ?? [],
    mesajlar: messagesRes.data ?? [],
  };

  return new Response(JSON.stringify(ihracat, null, 2), {
    status: 200,
    headers: {
      "Content-Type": "application/json",
      "Content-Disposition": `attachment; filename="verilerim-${new Date().toISOString().split("T")[0]}.json"`,
    },
  });
}
```

`{{KULLANICI_TABLOLARI}}` placeholder'ı için Claude:
- `package.json`'da Supabase görürse `lib/supabase/server.ts` benzeri kütüphaneyi bulur
- `schema.sql` veya migration dosyalarından user_id'li tabloları tespit eder
- Yoksa kullanıcıya sorar

---

## 3. Hesap Silme API — Supabase

`app/api/me/route.ts`:

```ts
import { NextRequest } from "next/server";
import { createClient as createServerClient } from "@/lib/supabase/server";
import { createClient as createAdminClient } from "@supabase/supabase-js";

export async function DELETE(req: NextRequest) {
  const supabase = await createServerClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    return new Response(
      JSON.stringify({ error: "Giriş yapmamışsınız" }),
      { status: 401, headers: { "Content-Type": "application/json" } }
    );
  }

  // Admin client — SERVICE_ROLE_KEY gerekli (env var)
  // BU KEY ASLA CLIENT'A SIZMAMALI!
  const admin = createAdminClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!,
    { auth: { autoRefreshToken: false, persistSession: false } }
  );

  try {
    // Önce kullanıcıya bağlı tüm verileri sil
    // (Eğer FK constraint ON DELETE CASCADE varsa bu otomatik olur,
    //  ama emin olmak için manuel siliyoruz)

    await Promise.all([
      admin.from("messages").delete().or(`sender_id.eq.${user.id},receiver_id.eq.${user.id}`),
      admin.from("follows").delete().or(`follower_id.eq.${user.id},following_id.eq.${user.id}`),
      admin.from("saves").delete().eq("user_id", user.id),
      admin.from("votes").delete().eq("user_id", user.id),
      admin.from("experiments").delete().eq("user_id", user.id),
      admin.from("profiles").delete().eq("id", user.id),
    ]);

    // Son olarak auth.users'tan sil (bu cascade etmez, ayrıca yapılır)
    const { error: authError } = await admin.auth.admin.deleteUser(user.id);
    if (authError) throw authError;

    return new Response(JSON.stringify({ silindi: true }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    });
  } catch (err) {
    console.error("Hesap silme hatası:", err);
    return new Response(
      JSON.stringify({
        error: err instanceof Error ? err.message : "Silme başarısız",
      }),
      { status: 500, headers: { "Content-Type": "application/json" } }
    );
  }
}
```

### Önemli: `.env.local`'a ekle

```
SUPABASE_SERVICE_ROLE_KEY=...
```

**ASLA** client-side'da kullanma. Sadece API route'larda.

---

## 4. UGC için stratejik karar — sil mi, anonimleştir mi?

Sosyal platformlarda (UGC tutan siteler) kullanıcı sildikten sonra:

| Veri tipi | Önerilen strateji | Gerekçe |
|---|---|---|
| Profil | **Sil** | Kişisel veri |
| Email/şifre | **Sil** | Kişisel veri |
| Kullanıcı adı | **Sil veya rezerve** | Kişisel veri; ama "deleted-{id}" placeholder bırakılabilir |
| Posts/Yorumlar | **Sil** veya **anonimleştir** | Karar projeye bağlı, kullanım koşullarında belirtilmeli |
| Oylar | **Sil** | İstatistik etkilenebilir; ama KVKK silme hakkı önceliklidir |
| Gönderilen mesajlar | **Sil sender'ı** veya **anonimleştir** | Alıcı taraf hâlâ konuşmayı görmek isteyebilir |
| Aldığı mesajlar | **Sil** | Kullanıcı silindiğinden gelen mesajlar anlamsız |

**Anonimleştirme alternatifi** (silmek yerine):
```ts
// Posts'u silmek yerine, user_id'yi nullable yap ve "Silinen Kullanıcı" göster
await admin.from("experiments").update({
  user_id: null,
  // veya user_id'yi koru ama profil silindiği için "Silinen Kullanıcı" göster
}).eq("user_id", user.id);
```

Bu yaklaşım UGC'nin "topluluk değeri"ni korur (oy/yorum ilişkileri kopmaz) ama kullanıcının silme talebini yerine getirir (PII ortadan kaldırılır).

Kullanım koşullarında bu yaklaşımı açıkça belirt:
> "Hesabınızı sildiğinizde profil bilginiz ve kişisel verileriniz silinir, ancak yayımladığınız içerikler 'Silinen Kullanıcı' adıyla saklanmaya devam edebilir."

---

## 5. Soft delete (geri alınabilir) — opsiyonel

Hesap silindi dedikten 30 gün sonra hard delete yapmak isterseniz:

```ts
// Profile tablosunda:
ALTER TABLE profiles ADD COLUMN deleted_at TIMESTAMP;

// Silme API'sinde:
await admin.from("profiles").update({
  deleted_at: new Date().toISOString(),
  email: null,  // PII'yi hemen kaldır
  username: `deleted-${user.id}`,
}).eq("id", user.id);

// Auth'tan silme yerine ban et:
await admin.auth.admin.updateUserById(user.id, { ban_duration: "100000h" });

// 30 gün sonra Vercel Cron / Supabase Edge Function ile hard delete:
const eskiSilinenler = await admin.from("profiles")
  .delete()
  .lt("deleted_at", new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString());
```

Avantaj: kullanıcı 30 gün içinde "geri al" diyebilir.
Dezavantaj: PII 30 gün daha sistemde kalır — bunu kullanım koşullarında belirt.

---

## 6. Prisma alternatifi (Supabase yerine)

`prisma.user.delete({ where: { id: userId } })` — cascade etmesi için `schema.prisma`'da `onDelete: Cascade` ayarı olmalı:

```prisma
model Post {
  id     String @id
  userId String
  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

Bu olduğunda `prisma.user.delete` tüm bağımlı kayıtları otomatik temizler.

---

## 7. Generic (DB bağımsız) — sadece email tabanlı

Eğer veri saklamıyorsan ama email haberdarlık formu varsa (örn. newsletter):

```ts
// POST /api/me/silme-talebi
export async function POST(req: NextRequest) {
  const { email } = await req.json();
  // 1. Email'i listeden kaldır (Mailchimp/Substack/whatever API)
  // 2. Kullanıcıya onay maili gönder
  return Response.json({ success: true });
}
```

---

## Aydınlatma metnine ekleme

Bu özellikler eklendikten sonra, aydınlatma metninin "Veri Sahibi Hakları" bölümünü güncelle:

> Yukarıdaki hakların büyük çoğunluğunu hesabınızın{" "}
> <Link href="/hesabim">/hesabim</Link> sayfasından **kendiniz**
> kullanabilirsiniz. Yapamadıklarınız için{" "}
> <a href="mailto:{{ILETISIM_EMAIL}}">{{ILETISIM_EMAIL}}</a>{" "}
> adresine yazın.

---

## Test checklist

Üretim öncesi mutlaka test et:

- [ ] Export butonu çalışıyor → indirilen JSON tüm kullanıcı verisini içeriyor mu?
- [ ] Silme akışı → "SİL" yazılmadan buton disabled
- [ ] Silme sonrası → kullanıcı login olamıyor (banned veya silindi)
- [ ] Silme sonrası → diğer kullanıcılar profil sayfasında 404 görüyor (veya "Silinen Kullanıcı")
- [ ] Silme sonrası → kullanıcının UGC'si (post, comment) silindi veya anonimleşti
- [ ] Service role key sadece server-side kullanılıyor (NEXT_PUBLIC_ prefix YOK)
- [ ] Silinen email tekrar kayıt olabiliyor (Supabase'in default davranışı)

---

*Bu şablon hukuki tavsiye değildir. Üretim öncesi avukat review önerilir.*
