# Footer Şablonu — Yasal Linkli

**Placeholder'lar**:
- `{{PROJE_ADI}}` — opsiyonel, footer'da gösterilecek site adı/slogan
- `{{LANG}}` — "tr" veya "en" (Türkçe linkler vs İngilizce linkler)

---

## Next.js App Router — `components/Footer.tsx`

```tsx
import Link from "next/link";

export default function Footer() {
  return (
    <footer className="site-footer">
      <nav>
        <Link href="/">Ana sayfa</Link>
        <span className="ayrac" aria-hidden="true" />
        <Link href="/aydinlatma">Aydınlatma metni</Link>
        <span className="ayrac" aria-hidden="true" />
        <Link href="/kosullar">Kullanım koşulları</Link>
      </nav>
      <p className="footer-not">{{PROJE_ADI}}</p>
    </footer>
  );
}
```

### `app/layout.tsx`'e ekle

```tsx
import Footer from "@/components/Footer";

// body içine:
<body>
  {children}
  <Footer />
</body>
```

### CSS (`app/globals.css`)

```css
.site-footer {
  margin-top: 5rem;
  padding: 2rem 1.5rem 2.5rem;
  border-top: 1px solid rgba(180, 180, 180, 0.1);
  text-align: center;
}

.site-footer nav {
  display: inline-flex;
  align-items: center;
  gap: 1.2rem;
  flex-wrap: wrap;
  justify-content: center;
}

.site-footer a {
  text-decoration: none;
  font-size: 0.66rem;
  letter-spacing: 0.4em;
  text-transform: uppercase;
  font-weight: 300;
  opacity: 0.55;
  transition: opacity 0.4s ease;
}

.site-footer a:hover {
  opacity: 1;
}

.site-footer .ayrac {
  width: 1px;
  height: 10px;
  background: rgba(180, 180, 180, 0.18);
}

.site-footer .footer-not {
  margin-top: 1.2rem;
  font-size: 0.62rem;
  opacity: 0.35;
  letter-spacing: 0.2em;
  font-style: italic;
}
```

---

## GDPR varyantı (İngilizce)

`{{LANG}}` "en" ise:

```tsx
<nav>
  <Link href="/">Home</Link>
  <span className="ayrac" aria-hidden="true" />
  <Link href="/privacy">Privacy Policy</Link>
  <span className="ayrac" aria-hidden="true" />
  <Link href="/terms">Terms of Service</Link>
</nav>
```

---

## Çoklu jurisdiction (KVKK + GDPR)

Site hem TR hem EU kitleye sunuluyorsa:

```tsx
<nav>
  <Link href="/">Ana sayfa / Home</Link>
  <span className="ayrac" />
  <Link href="/aydinlatma">Aydınlatma / Privacy</Link>
  <span className="ayrac" />
  <Link href="/kosullar">Koşullar / Terms</Link>
</nav>
```

Veya iki ayrı sayfa: `/aydinlatma` (TR) + `/privacy` (EN), dil-bazlı linkle.

---

## Pages Router (eski Next.js) — `pages/_app.tsx`

```tsx
import Footer from "@/components/Footer";

function App({ Component, pageProps }) {
  return (
    <>
      <Component {...pageProps} />
      <Footer />
    </>
  );
}
```

---

## Vite + React — `src/App.tsx`

```tsx
import Footer from "./components/Footer";

export default function App() {
  return (
    <>
      <Routes>...</Routes>
      <Footer />
    </>
  );
}
```

---

## Flask — `templates/base.html` veya kullanılacak base template

```html
{% block content %}{% endblock %}

<footer class="site-footer">
  <nav>
    <a href="{{ url_for('index') }}">Ana sayfa</a>
    <span class="ayrac"></span>
    <a href="{{ url_for('aydinlatma') }}">Aydınlatma metni</a>
    <span class="ayrac"></span>
    <a href="{{ url_for('kosullar') }}">Kullanım koşulları</a>
  </nav>
</footer>
```

---

*Bu şablon hukuki tavsiye değildir.*
