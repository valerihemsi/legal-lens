---
template: cookie-banner
language: tr
legal_basis:
  - kvkk_kurul_2020_216
  - kvkk_md_5_1
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "Kurul 2020/216 — kategori bazlı opt-in; reject all = accept all kadar görünür olmalı"
---

# Cookie Banner + Granular Consent Yönetimi

Türkiye (KVKK Kurul 2020/216) ve EU (GDPR/ePrivacy) için **kategori bazlı opt-in** mantığı. Kullanıcı her kategoriyi ayrı ayrı seçebilir; onay verilmeyen kategorideki hiçbir script/çerez yüklenmez.

**Placeholder'lar**:
- `{{PROJE_ADI}}`
- `{{POLITIKA_URL}}` — `/cerez-politikasi`
- `{{AYARLAR_URL}}` — `/cerez-ayarlari` (opsiyonel)

---

## Mimari

5 parça:

1. **`CookieBanner.tsx`** — ilk ziyarette gösterilen panel (kompakt + genişletilmiş görünüm)
2. **`CookieSettings.tsx`** — kategori toggle'ları (banner içinde ve ayarlar sayfasında kullanılır)
3. **`useCookieConsent` hook** — herhangi bir component'tan kategori bazlı onay durumunu okur
4. **`ConsentGate` wrapper** — onay yoksa içeriği render etmez
5. **`/cerez-ayarlari` sayfası** — kullanıcı tercihlerini sonradan değiştirir

`localStorage`'da saklanan onay: JSON-encoded `ConsentState` object.

---

## Kategoriler

| Kategori | Anahtar | Açıklama | Default |
|---|---|---|---|
| Kesinlikle gerekli | `necessary` | Auth, oturum, dil tercihi — kapatılamaz | `true` (her zaman) |
| İşlevsel | `functional` | Tema seçimi, kayıtlı tercihler | `false` (opt-in) |
| Analitik | `analytics` | Vercel Analytics, GA — anonim istatistik | `false` (opt-in) |
| Pazarlama | `marketing` | FB Pixel, ad tracking — hedefli reklam | `false` (opt-in) |

Proje tüm kategorileri kullanmayabilir — sadece kullanılanları göster.

---

## 1. `lib/cookieConsent.ts` — Tip ve yardımcılar

```ts
"use client";

export type CookieCategory = "necessary" | "functional" | "analytics" | "marketing";

export type ConsentState = Record<CookieCategory, boolean>;

export const DEFAULT_CONSENT: ConsentState = {
  necessary: true,  // her zaman true, değiştirilemez
  functional: false,
  analytics: false,
  marketing: false,
};

const STORAGE_KEY = "cookie-consent";

export function loadConsent(): ConsentState | null {
  if (typeof window === "undefined") return null;
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return null;
  // Eski format ile uyumluluk
  if (raw === "accepted") return { ...DEFAULT_CONSENT, functional: true, analytics: true, marketing: true };
  if (raw === "essential") return { ...DEFAULT_CONSENT };
  try {
    const parsed = JSON.parse(raw);
    return { ...DEFAULT_CONSENT, ...parsed, necessary: true };
  } catch {
    return null;
  }
}

export function saveConsent(state: ConsentState): void {
  const toSave = { ...state, necessary: true };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(toSave));
  window.dispatchEvent(new CustomEvent("cookie-consent-changed", { detail: toSave }));
}

export function resetConsent(): void {
  localStorage.removeItem(STORAGE_KEY);
  window.dispatchEvent(new CustomEvent("cookie-consent-changed", { detail: null }));
}

export function acceptAll(): void {
  saveConsent({ necessary: true, functional: true, analytics: true, marketing: true });
}

export function rejectAll(): void {
  saveConsent({ ...DEFAULT_CONSENT });
}
```

---

## 2. `lib/useCookieConsent.ts` — Hook

```ts
"use client";

import { useEffect, useState } from "react";
import type { CookieCategory, ConsentState } from "./cookieConsent";
import { loadConsent } from "./cookieConsent";

// Tüm state'i al
export function useCookieConsent(): ConsentState | null;
// Bir kategoriyi al
export function useCookieConsent(category: CookieCategory): boolean;
// Implementasyon
export function useCookieConsent(category?: CookieCategory): ConsentState | boolean | null {
  const [state, setState] = useState<ConsentState | null>(null);

  useEffect(() => {
    setState(loadConsent());
    const handler = (e: Event) => {
      setState((e as CustomEvent).detail as ConsentState | null);
    };
    window.addEventListener("cookie-consent-changed", handler);
    return () => window.removeEventListener("cookie-consent-changed", handler);
  }, []);

  if (category) {
    return state?.[category] ?? false;
  }
  return state;
}
```

---

## 3. `components/CookieSettings.tsx` — Toggle paneli

Banner içinde ve `/cerez-ayarlari` sayfasında kullanılabilir.

```tsx
"use client";

import { useState } from "react";
import type { CookieCategory, ConsentState } from "@/lib/cookieConsent";

interface KategoriInfo {
  anahtar: CookieCategory;
  baslik: string;
  aciklama: string;
  zorunlu?: boolean;
}

// Proje kullanıyorsa burada listele, kullanmıyorsa kaldır
const KATEGORILER: KategoriInfo[] = [
  {
    anahtar: "necessary",
    baslik: "Kesinlikle Gerekli",
    aciklama: "Oturum, dil tercihi gibi sitenin çalışması için zorunlu çerezler. Kapatılamaz.",
    zorunlu: true,
  },
  {
    anahtar: "functional",
    baslik: "İşlevsel",
    aciklama: "Tema seçimi, kayıtlı tercihleriniz gibi konforu artıran çerezler.",
  },
  {
    anahtar: "analytics",
    baslik: "Analitik",
    aciklama: "Anonim kullanım istatistikleri (sayfa görüntüleme, süre). Sizi tanımlamaz.",
  },
  {
    anahtar: "marketing",
    baslik: "Pazarlama",
    aciklama: "Hedefli reklam ve sosyal medya entegrasyon çerezleri.",
  },
];

interface CookieSettingsProps {
  initial: ConsentState;
  onSave: (state: ConsentState) => void;
  onCancel?: () => void;
  showHeader?: boolean;
}

export default function CookieSettings({
  initial,
  onSave,
  onCancel,
  showHeader = true,
}: CookieSettingsProps) {
  const [state, setState] = useState<ConsentState>(initial);

  const toggle = (kat: CookieCategory) => {
    if (kat === "necessary") return; // değiştirilemez
    setState((s) => ({ ...s, [kat]: !s[kat] }));
  };

  return (
    <div className="cookie-settings">
      {showHeader && (
        <header className="cookie-settings-baslik">
          <h2>Çerez tercihlerini özelleştir</h2>
          <p>Her kategoride istediklerinizi açıp kapatabilirsiniz.</p>
        </header>
      )}

      <div className="cookie-settings-liste">
        {KATEGORILER.map((kat) => (
          <div key={kat.anahtar} className="cookie-kategori">
            <label className="cookie-kategori-satir">
              <span className="cookie-kategori-bilgi">
                <span className="cookie-kategori-baslik">{kat.baslik}</span>
                <span className="cookie-kategori-aciklama">{kat.aciklama}</span>
              </span>
              <span
                className={`cookie-toggle ${state[kat.anahtar] ? "acik" : "kapali"} ${kat.zorunlu ? "zorunlu" : ""}`}
                role="switch"
                aria-checked={state[kat.anahtar]}
                aria-readonly={kat.zorunlu}
                onClick={() => toggle(kat.anahtar)}
                onKeyDown={(e) => {
                  if (e.key === "Enter" || e.key === " ") {
                    e.preventDefault();
                    toggle(kat.anahtar);
                  }
                }}
                tabIndex={kat.zorunlu ? -1 : 0}
              >
                <span className="cookie-toggle-knob" />
              </span>
            </label>
          </div>
        ))}
      </div>

      <div className="cookie-settings-butonlar">
        {onCancel && (
          <button
            type="button"
            onClick={onCancel}
            className="cookie-banner-buton cookie-banner-buton-soluk"
          >
            Vazgeç
          </button>
        )}
        <button
          type="button"
          onClick={() => onSave(state)}
          className="cookie-banner-buton cookie-banner-buton-vurgu"
        >
          Seçimleri kaydet
        </button>
      </div>
    </div>
  );
}
```

---

## 4. `components/CookieBanner.tsx` — Ana banner

```tsx
"use client";

import { useEffect, useState } from "react";
import Link from "next/link";
import CookieSettings from "./CookieSettings";
import {
  DEFAULT_CONSENT,
  loadConsent,
  saveConsent,
  acceptAll,
  rejectAll,
  type ConsentState,
} from "@/lib/cookieConsent";

export default function CookieBanner() {
  const [gosteriliyor, setGosteriliyor] = useState(false);
  const [genisletildi, setGenisletildi] = useState(false);
  const [taslakState, setTaslakState] = useState<ConsentState>(DEFAULT_CONSENT);
  const [hydrated, setHydrated] = useState(false);

  useEffect(() => {
    const mevcut = loadConsent();
    setGosteriliyor(mevcut === null);
    setHydrated(true);
  }, []);

  if (!hydrated || !gosteriliyor) return null;

  const tumunuKabul = () => {
    acceptAll();
    setGosteriliyor(false);
  };

  const reddet = () => {
    rejectAll();
    setGosteriliyor(false);
  };

  const ozelKaydet = (state: ConsentState) => {
    saveConsent(state);
    setGosteriliyor(false);
  };

  return (
    <div
      className={`cookie-banner ${genisletildi ? "genis" : ""}`}
      role="dialog"
      aria-labelledby="cookie-banner-title"
      aria-describedby="cookie-banner-desc"
    >
      {!genisletildi ? (
        <div className="cookie-banner-icerik">
          <h2 id="cookie-banner-title" className="cookie-banner-baslik">
            Çerezler hakkında
          </h2>
          <p id="cookie-banner-desc" className="cookie-banner-aciklama">
            {{PROJE_ADI}} deneyimini geliştirmek için çerez kullanır. Hangi
            çerezleri kabul ettiğini seçebilirsin.{" "}
            <Link href="{{POLITIKA_URL}}" className="cookie-banner-link">
              Çerez politikası
            </Link>
          </p>
          <div className="cookie-banner-butonlar">
            <button
              type="button"
              onClick={reddet}
              className="cookie-banner-buton cookie-banner-buton-soluk"
            >
              Reddet
            </button>
            <button
              type="button"
              onClick={() => setGenisletildi(true)}
              className="cookie-banner-buton cookie-banner-buton-soluk"
            >
              Özelleştir
            </button>
            <button
              type="button"
              onClick={tumunuKabul}
              className="cookie-banner-buton cookie-banner-buton-vurgu"
            >
              Tümünü kabul et
            </button>
          </div>
        </div>
      ) : (
        <CookieSettings
          initial={taslakState}
          onSave={ozelKaydet}
          onCancel={() => setGenisletildi(false)}
          showHeader
        />
      )}
    </div>
  );
}
```

`taslakState`'i her render'da `DEFAULT_CONSENT` olarak başlatmak yerine ister `useEffect` ile mevcut state'ten al ister tasarımına göre değiştir.

---

## 5. `components/ConsentGate.tsx` — Kategori bazlı wrapper

```tsx
"use client";

import type { ReactNode } from "react";
import type { CookieCategory } from "@/lib/cookieConsent";
import { useCookieConsent } from "@/lib/useCookieConsent";

interface ConsentGateProps {
  children: ReactNode;
  /** Hangi kategori(ler) onaylı olmalı? Tüm verilenlerin true olması gerekir. */
  requires: CookieCategory | CookieCategory[];
}

export default function ConsentGate({ children, requires }: ConsentGateProps) {
  const consent = useCookieConsent() as Record<CookieCategory, boolean> | null;
  if (!consent) return null;

  const kategoriler = Array.isArray(requires) ? requires : [requires];
  const hepsiOnayli = kategoriler.every((k) => consent[k]);

  if (!hepsiOnayli) return null;
  return <>{children}</>;
}
```

### Kullanımı

```tsx
// Sadece analytics onaylıysa Vercel Analytics yükle
<ConsentGate requires="analytics">
  <Analytics />
</ConsentGate>

// Hem analytics hem marketing gerekli ise
<ConsentGate requires={["analytics", "marketing"]}>
  <FacebookPixel />
</ConsentGate>

// YouTube embed — functional cookie gerektiriyor
<ConsentGate requires="functional">
  <YouTubeEmbed videoId="..." />
</ConsentGate>
```

---

## 6. `/cerez-ayarlari` sayfası — Sonradan değiştirme

`app/cerez-ayarlari/page.tsx`:

```tsx
"use client";

import { useEffect, useState } from "react";
import Link from "next/link";
import CookieSettings from "@/components/CookieSettings";
import {
  DEFAULT_CONSENT,
  loadConsent,
  saveConsent,
  resetConsent,
  type ConsentState,
} from "@/lib/cookieConsent";

export default function CerezAyarlariPage() {
  const [state, setState] = useState<ConsentState | null>(null);
  const [kaydedildi, setKaydedildi] = useState(false);

  useEffect(() => {
    setState(loadConsent() ?? DEFAULT_CONSENT);
  }, []);

  if (!state) {
    return <main className="min-h-screen flex items-center justify-center">Yükleniyor...</main>;
  }

  const kaydet = (yeni: ConsentState) => {
    saveConsent(yeni);
    setState(yeni);
    setKaydedildi(true);
    setTimeout(() => setKaydedildi(false), 3000);
  };

  const sifirla = () => {
    resetConsent();
    setState(DEFAULT_CONSENT);
  };

  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-2xl mx-auto space-y-8">
        <header>
          <h1 className="text-3xl font-light tracking-wide">Çerez Ayarları</h1>
          <p className="text-sm opacity-70 mt-2">
            Her kategoriyi ayrı ayrı yönetebilirsin. Değişiklikler hemen
            uygulanır.
          </p>
        </header>

        <CookieSettings
          initial={state}
          onSave={kaydet}
          showHeader={false}
        />

        {kaydedildi && (
          <p className="text-sm text-emerald-300/90 italic text-center">
            ✓ Tercihleriniz kaydedildi
          </p>
        )}

        <div className="border-t pt-6 text-sm opacity-75 space-y-3">
          <p>
            Tüm tercihlerinizi sıfırlamak isterseniz aşağıdaki butona basın —
            bir sonraki ziyarette çerez kutusu yeniden gösterilecektir.
          </p>
          <button
            type="button"
            onClick={sifirla}
            className="cookie-banner-buton cookie-banner-buton-soluk"
          >
            Tüm tercihleri sıfırla
          </button>
        </div>

        <div className="border-t pt-6 text-sm">
          <Link href="/cerez-politikasi">→ Çerez Politikası</Link>
        </div>
      </div>
    </main>
  );
}
```

---

## CSS — `app/globals.css`

```css
.cookie-banner {
  position: fixed;
  bottom: 1rem;
  right: 1rem;
  left: 1rem;
  max-width: 32rem;
  margin-left: auto;
  z-index: 9999;
  background: rgba(20, 20, 25, 0.94);
  backdrop-filter: blur(16px);
  border: 1px solid rgba(212, 196, 160, 0.18);
  border-radius: 0.75rem;
  padding: 1.4rem 1.5rem;
  color: rgba(221, 213, 196, 0.92);
  font-size: 0.85rem;
  line-height: 1.55;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
  animation: cookie-banner-giris 0.5s ease-out;
  max-height: calc(100vh - 2rem);
  overflow-y: auto;
}

.cookie-banner.genis {
  max-width: 36rem;
}

@keyframes cookie-banner-giris {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.cookie-banner-baslik {
  font-size: 0.95rem;
  font-weight: 500;
  margin-bottom: 0.6rem;
  color: rgba(212, 196, 160, 0.95);
}

.cookie-banner-aciklama {
  font-size: 0.78rem;
  line-height: 1.7;
  margin-bottom: 1rem;
  opacity: 0.85;
}

.cookie-banner-link {
  color: rgba(212, 196, 160, 1);
  text-decoration: underline;
  text-underline-offset: 2px;
}

.cookie-banner-butonlar {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.cookie-banner-buton {
  flex: 1;
  min-width: 7rem;
  padding: 0.55rem 0.8rem;
  font-size: 0.7rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  border-radius: 0.3rem;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 400;
  border: 1px solid transparent;
}

.cookie-banner-buton-soluk {
  background: transparent;
  color: rgba(221, 213, 196, 0.75);
  border-color: rgba(212, 196, 160, 0.25);
}

.cookie-banner-buton-soluk:hover {
  border-color: rgba(212, 196, 160, 0.5);
  color: rgba(221, 213, 196, 1);
}

.cookie-banner-buton-vurgu {
  background: rgba(212, 196, 160, 0.85);
  color: rgba(20, 20, 25, 1);
  border-color: rgba(212, 196, 160, 0.85);
}

.cookie-banner-buton-vurgu:hover {
  background: rgba(212, 196, 160, 1);
}

/* Settings panel */
.cookie-settings-baslik {
  margin-bottom: 1rem;
}

.cookie-settings-baslik h2 {
  font-size: 0.95rem;
  font-weight: 500;
  margin-bottom: 0.35rem;
  color: rgba(212, 196, 160, 0.95);
}

.cookie-settings-baslik p {
  font-size: 0.75rem;
  opacity: 0.7;
}

.cookie-settings-liste {
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
  margin-bottom: 1.25rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid rgba(180, 180, 180, 0.08);
}

.cookie-kategori-satir {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 1rem;
  cursor: pointer;
  padding: 0.5rem 0;
}

.cookie-kategori-bilgi {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.cookie-kategori-baslik {
  font-size: 0.82rem;
  font-weight: 500;
  color: rgba(221, 213, 196, 0.95);
}

.cookie-kategori-aciklama {
  font-size: 0.7rem;
  line-height: 1.5;
  opacity: 0.65;
}

/* Toggle */
.cookie-toggle {
  position: relative;
  width: 2.4rem;
  height: 1.25rem;
  flex-shrink: 0;
  background: rgba(180, 180, 180, 0.18);
  border-radius: 999px;
  cursor: pointer;
  transition: background 0.3s ease;
  margin-top: 0.15rem;
}

.cookie-toggle.acik {
  background: rgba(34, 197, 94, 0.55);
}

.cookie-toggle.zorunlu {
  background: rgba(212, 196, 160, 0.45);
  cursor: not-allowed;
}

.cookie-toggle-knob {
  position: absolute;
  left: 0.125rem;
  top: 0.125rem;
  width: 1rem;
  height: 1rem;
  background: rgba(240, 237, 232, 0.95);
  border-radius: 50%;
  transition: transform 0.25s ease;
}

.cookie-toggle.acik .cookie-toggle-knob {
  transform: translateX(1.15rem);
}

.cookie-toggle:focus-visible {
  outline: 2px solid rgba(212, 196, 160, 0.8);
  outline-offset: 2px;
}

.cookie-settings-butonlar {
  display: flex;
  gap: 0.5rem;
}

@media (max-width: 480px) {
  .cookie-banner {
    bottom: 0;
    right: 0;
    left: 0;
    max-width: none;
    border-radius: 0.75rem 0.75rem 0 0;
  }
  .cookie-banner-butonlar,
  .cookie-settings-butonlar {
    flex-direction: column;
  }
  .cookie-banner-buton {
    width: 100%;
  }
}
```

---

## Aydınlatma ve Footer entegrasyonu

### Footer'a `/cerez-ayarlari` linki ekle

```tsx
<nav>
  <Link href="/">Ana sayfa</Link>
  <span className="ayrac" />
  <Link href="/aydinlatma">Aydınlatma metni</Link>
  <span className="ayrac" />
  <Link href="/kosullar">Kullanım koşulları</Link>
  <span className="ayrac" />
  <Link href="/cerez-politikasi">Çerez politikası</Link>
  <span className="ayrac" />
  <Link href="/cerez-ayarlari">Çerez ayarları</Link>
</nav>
```

### `app/layout.tsx`'e banner ekle

```tsx
import CookieBanner from "@/components/CookieBanner";

<body>
  {children}
  <Footer />
  <CookieBanner />
</body>
```

### Analytics'leri ConsentGate ile sar

Mevcut analytics import'larını wrap'le. Örn:

```tsx
import { Analytics } from "@vercel/analytics/react";
import ConsentGate from "@/components/ConsentGate";

<ConsentGate requires="analytics">
  <Analytics />
</ConsentGate>
```

---

## Eski (basit) varyant — opsiyonel

Eğer proje çok minimal ise ve sadece tek analitik var, granular karmaşık gelebilir. Bu durumda basit "Tümünü kabul et / Reddet" yeterlidir:

```tsx
// Basit kabul ya da reddet
const [accepted, setAccepted] = useState<boolean | null>(null);
// loadConsent() === null ? göster
// Aksiyon → saveConsent({ necessary: true, analytics: accepted, marketing: accepted, functional: accepted })
```

Granular formattan basit'e geçmek için: 4 toggle yerine 2 buton. Veri yapısı yine aynı kalır.

---

## Test checklist

- [ ] İlk ziyarette banner görünüyor
- [ ] "Reddet" → sadece `necessary: true`, kalanlar false
- [ ] "Tümünü kabul" → hepsi true
- [ ] "Özelleştir" → toggle'lar gözüküyor, `necessary` disabled
- [ ] Toggle'lar yalnızca tıklanıp kaydedince geçerli olur (instant değil — onSave ile)
- [ ] Onay verildikten sonra banner kaybolur
- [ ] `<ConsentGate requires="analytics">` analytics yokken render etmez
- [ ] `/cerez-ayarlari` sayfasında tercihler değiştirilebilir
- [ ] Sıfırlama butonu → çerez tekrar gösterilir
- [ ] Mobile'da banner alta yapışır, butonlar dikey

---

*Bu şablon hukuki tavsiye değildir.*
