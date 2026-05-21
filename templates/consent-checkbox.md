---
template: consent-checkbox
language: tr
legal_basis:
  - kvkk_md_5_1
  - kvkk_kurul_2020_216
jurisdiction: TR
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 6_months
version: "0.3"
notes: "Pre-ticked checkbox yasak (md. 5/1 açık rıza şartları)"
---

# Açık Rıza Checkbox Şablonu

Form öncesi zorunlu rıza kutusu. React + Next.js öncelikli ama HTML/Vue/Svelte için aşağıda alternatif var.

**Placeholder'lar**:
- `{{ISLEME_OZETI}}` — bir cümlede ne yapılıyor (örn. "sembolik yorum üretmek")
- `{{UCUNCU_TARAF_KISA}}` — kısa kısa: "Anthropic (ABD)" gibi, virgülle ayır
- `{{YAS_SINIRI}}` — 18 veya 13
- `{{ASIL_RISK}}` — opsiyonel, hizmet türüne göre "kehanet/teşhis/yatırım tavsiyesi olmadığı"

---

## React/TSX (Next.js)

```tsx
// State (component'in en üstüne ekle)
const [rizaVerildi, setRizaVerildi] = useState(false);

// Form içine, submit butonundan ÖNCE ekle
<label className="riza-kutu">
  <input
    type="checkbox"
    checked={rizaVerildi}
    onChange={(e) => setRizaVerildi(e.target.checked)}
    required
  />
  <span className="riza-kutu-isaret" aria-hidden="true" />
  <span className="riza-kutu-metin">
    {{YAS_SINIRI}} yaşımı doldurdum. Yukarıda girdiğim verilerin{" "}
    <strong>{{ISLEME_OZETI}}</strong> amacıyla işlenmesine ve bu amaçla{" "}
    <strong>{{UCUNCU_TARAF_KISA}}</strong> sunucularına aktarılmasına{" "}
    <strong>açık rıza</strong> veriyorum.{" "}
    {{ASIL_RISK_BLOK}}{" "}
    <Link href="/aydinlatma" target="_blank" className="riza-link">
      Aydınlatma metni
    </Link>
    {" · "}
    <Link href="/kosullar" target="_blank" className="riza-link">
      Kullanım koşulları
    </Link>
  </span>
</label>

// Submit handler'a kontrol ekle
const submitForm = () => {
  if (!rizaVerildi) {
    setHata("Devam etmek için onay kutusunu işaretlemeniz gerekir.");
    return;
  }
  // ... mevcut submit mantığı
};

// Submit button'ı disabled yap
<button type="submit" disabled={submitting || !rizaVerildi}>
  ...
</button>
```

### `{{ASIL_RISK_BLOK}}` örnekleri

AI üretimi varsa:
```
"Bunun yapay zekâ tarafından üretildiğini ve {{ASIL_RISK}} olmadığını anlıyorum."
```

Yoksa boş bırak.

### CSS (proje stiline göre uyarla)

```css
.riza-kutu {
  display: flex;
  gap: 0.85rem;
  align-items: flex-start;
  cursor: pointer;
  padding: 0.85rem 1rem;
  border: 1px solid rgba(180, 180, 180, 0.2);
  border-radius: 0.5rem;
  transition: all 0.3s ease;
}

.riza-kutu:hover {
  border-color: rgba(180, 180, 180, 0.4);
}

.riza-kutu input[type="checkbox"] {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
  pointer-events: none;
}

.riza-kutu-isaret {
  flex-shrink: 0;
  width: 1.05rem;
  height: 1.05rem;
  margin-top: 0.15rem;
  border: 1px solid currentColor;
  border-radius: 0.2rem;
  position: relative;
  transition: all 0.3s ease;
}

.riza-kutu input[type="checkbox"]:checked + .riza-kutu-isaret {
  background: currentColor;
}

.riza-kutu input[type="checkbox"]:checked + .riza-kutu-isaret::after {
  content: "✓";
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--background, #000);
  font-size: 0.7rem;
}

.riza-kutu input[type="checkbox"]:focus-visible + .riza-kutu-isaret {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}

.riza-kutu-metin {
  font-size: 0.78rem;
  line-height: 1.7;
}

.riza-link {
  text-decoration: underline;
  text-underline-offset: 2px;
}
```

---

## Saf HTML alternatifi (Flask, Vanilla JS)

```html
<label class="riza-kutu" for="riza-kutu-input">
  <input type="checkbox" id="riza-kutu-input" name="riza" required />
  <span class="riza-kutu-metin">
    {{YAS_SINIRI}} yaşımı doldurdum. Verilerimin {{ISLEME_OZETI}} amacıyla
    işlenmesine ve {{UCUNCU_TARAF_KISA}} sunucularına aktarılmasına
    <strong>açık rıza</strong> veriyorum.
    <a href="/aydinlatma" target="_blank">Aydınlatma metni</a> ·
    <a href="/kosullar" target="_blank">Kullanım koşulları</a>
  </span>
</label>
```

Backend tarafında `request.form.get('riza') != 'on'` ise hata döndür.

---

## Vue 3 alternatifi

```vue
<script setup>
import { ref } from 'vue';
const rizaVerildi = ref(false);
</script>

<template>
  <label class="riza-kutu">
    <input type="checkbox" v-model="rizaVerildi" required />
    <span class="riza-kutu-metin">
      {{YAS_SINIRI}} yaşımı doldurdum. ...
      <a href="/aydinlatma" target="_blank">Aydınlatma metni</a> ·
      <a href="/kosullar" target="_blank">Kullanım koşulları</a>
    </span>
  </label>

  <button type="submit" :disabled="!rizaVerildi">Devam</button>
</template>
```

---

## Önemli: çift gönderim önlenmeli

Checkbox sadece UX. Server tarafında da kontrol edilmeli — yoksa kullanıcı DevTools'da disabled'ı kaldırıp gönderebilir. KVKK için **server-side enforcement** zorunlu değil ama önerilir (kayıt tutuyorsan açık rıza kanıtı gerekir).

```ts
// API route'ta (Next.js)
if (!body.rizaVerildi) {
  return Response.json(
    { error: "Açık rıza zorunludur" },
    { status: 400 }
  );
}
```

---

*Bu şablon hukuki tavsiye değildir.*
