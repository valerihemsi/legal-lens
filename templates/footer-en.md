---
template: footer-en
language: en
legal_basis:
  - gdpr_art_13
jurisdiction: EU
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 12_months
version: "0.3"
---

# Footer Template — English / GDPR

The TR `footer.md` already includes a brief EN variant. This file is the **full GDPR-aware footer** with cookie + account links, used when `/legal-lens generate gdpr` runs.

**Placeholders**:
- `{{PROJECT_NAME}}` — site name/tagline shown in the footer
- `{{HAS_COOKIES}}` — boolean: include cookie-policy/cookie-settings links?
- `{{HAS_AUTH}}` — boolean: include /account link (visible when logged in)?

---

## Next.js App Router — `components/Footer.tsx`

```tsx
import Link from "next/link";

export default function Footer() {
  return (
    <footer className="site-footer">
      <nav aria-label="Legal navigation">
        <Link href="/">Home</Link>
        <span className="separator" aria-hidden="true" />
        <Link href="/privacy">Privacy Policy</Link>
        <span className="separator" aria-hidden="true" />
        <Link href="/terms">Terms of Service</Link>
        {/* If cookies are used: */}
        {/* <span className="separator" aria-hidden="true" />
        <Link href="/cookie-policy">Cookie Policy</Link>
        <span className="separator" aria-hidden="true" />
        <Link href="/cookie-settings">Cookie Settings</Link> */}
        {/* If auth is available: */}
        {/* <span className="separator" aria-hidden="true" />
        <Link href="/account">My Account</Link> */}
      </nav>
      <p className="footer-note">{{PROJECT_NAME}}</p>
    </footer>
  );
}
```

### `app/layout.tsx`

```tsx
import Footer from "@/components/Footer";

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

.site-footer .separator {
  width: 1px;
  height: 10px;
  background: rgba(180, 180, 180, 0.18);
}

.site-footer .footer-note {
  margin-top: 1.2rem;
  font-size: 0.62rem;
  opacity: 0.35;
  letter-spacing: 0.2em;
  font-style: italic;
}
```

---

## Conditional link inclusion

Generate the appropriate set based on detection:

| Detected | Include |
|---|---|
| Auth + DB (Supabase, Prisma, etc.) | `/account` (when logged in) |
| Tracking SDK (analytics, marketing) | `/cookie-policy` + `/cookie-settings` |
| UGC + EU users | `/contact` or `/report` for content reports (DSA / 5651) |
| No cookies, no tracking | Just `/privacy` + `/terms` |

For UGC platforms with EU users, add a "Report content" link visible in footer:
```tsx
<span className="separator" aria-hidden="true" />
<Link href="/report">Report content</Link>
```

---

## Bilingual sites (TR + EN with locale switching)

If the project uses route-based locales (`/tr/...` and `/en/...`):

```tsx
import { useLocale } from "next-intl"; // or your i18n lib

export default function Footer() {
  const locale = useLocale();
  const isEN = locale === "en";

  return (
    <footer className="site-footer">
      <nav>
        <Link href={`/${locale}`}>{isEN ? "Home" : "Ana sayfa"}</Link>
        <span className="separator" aria-hidden="true" />
        <Link href={`/${locale}/${isEN ? "privacy" : "aydinlatma"}`}>
          {isEN ? "Privacy Policy" : "Aydınlatma metni"}
        </Link>
        <span className="separator" aria-hidden="true" />
        <Link href={`/${locale}/${isEN ? "terms" : "kosullar"}`}>
          {isEN ? "Terms of Service" : "Kullanım koşulları"}
        </Link>
      </nav>
    </footer>
  );
}
```

For sites serving both jurisdictions but in a single language, keep two parallel pages:
- `/privacy` (EN, GDPR primary) and `/aydinlatma` (TR, KVKK primary)
- Each page cross-links to the other at the bottom

---

## Other frameworks

### Vite + React — `src/App.tsx`
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

### Flask — `templates/base.html`
```html
{% block content %}{% endblock %}

<footer class="site-footer">
  <nav>
    <a href="{{ url_for('index') }}">Home</a>
    <span class="separator"></span>
    <a href="{{ url_for('privacy') }}">Privacy Policy</a>
    <span class="separator"></span>
    <a href="{{ url_for('terms') }}">Terms of Service</a>
  </nav>
</footer>
```

---

*This template is not legal advice.*
