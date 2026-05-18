# Cookie Policy Template (English)

Used as `app/cookie-policy/page.tsx`. A separate page from the Privacy Policy — cookie-specific details live here. References ePrivacy Directive Art. 5(3) (the "cookie law") and GDPR Art. 6(1)(a) for consent.

**Placeholders**:
- `{{PROJECT_NAME}}`
- `{{CONTACT_EMAIL}}`
- `{{COOKIE_TABLE}}` — table of detected/used cookies (Claude fills)
- `{{ANALYTICS_BLOCK}}` — per-SDK paragraph if analytics SDKs detected
- `{{DATE}}`

---

## `app/cookie-policy/page.tsx`

```tsx
import type { Metadata } from "next";
import Link from "next/link";

export const metadata: Metadata = {
  title: "Cookie Policy — {{PROJECT_NAME}}",
  description: "Information about cookies used on {{PROJECT_NAME}} and how to manage your preferences.",
};

export default function CookiePolicyPage() {
  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-3xl mx-auto">
        <header className="text-center mb-12">
          <h1 className="text-3xl md:text-4xl font-light tracking-wide">Cookie Policy</h1>
          <p className="text-sm opacity-70 mt-2">cookies and preference management</p>
        </header>

        <article className="prose prose-invert max-w-none space-y-6">
          <section>
            <h2>What is a cookie?</h2>
            <p>
              Cookies are small text files stored on your device by your
              browser when you visit a website. Browser local storage
              (localStorage, sessionStorage) and similar technologies are
              treated equivalently in law. Throughout this page we use the
              word <strong>&ldquo;cookie&rdquo;</strong> to cover all of them.
            </p>
            <p className="text-sm opacity-70">
              The legal basis for cookie use is set out in the ePrivacy
              Directive (2002/58/EC, as amended) Art. 5(3) and the GDPR
              Art. 6(1)(a) where consent is required.
            </p>
          </section>

          <section>
            <h2>Cookies used on this site</h2>
{{COOKIE_TABLE}}
          </section>

          <section>
            <h2>Cookie categories</h2>

            <h3>Strictly necessary cookies</h3>
            <p>
              Required for the basic operation of the service. They do not
              require consent under ePrivacy Art. 5(3) (the "strictly
              necessary" exemption) and cannot be disabled; the service will
              not function without them. Examples: authentication session
              cookie, language preference, temporary form-input storage.
            </p>

            <h3>Functional cookies</h3>
            <p>
              Used to remember preferences (e.g. theme, layout) that improve
              your experience but are not strictly necessary. We request your
              consent for these where they involve identifiers or
              tracking-adjacent storage.
            </p>

            <h3>Analytics cookies</h3>
            <p>
              Used to understand how the service is used by collecting
              aggregated, often anonymised statistics.{" "}
              <strong>Loaded only with your explicit consent.</strong> Active
              if you selected &ldquo;Accept all&rdquo; in the cookie banner.
            </p>

{{ANALYTICS_BLOCK}}

            <h3>Marketing cookies</h3>
            <p>
              No marketing cookies are currently used on this site. If
              introduced later, this policy will be updated and your consent
              re-requested.
            </p>
          </section>

          <section>
            <h2>How to manage your preferences</h2>
            <p>
              <strong>From within the site:</strong> on your first visit, a
              cookie notice appears at the bottom of the page. You may select
              &ldquo;Accept all&rdquo;, &ldquo;Necessary only&rdquo;, or
              customise categories. You can change your choice later via the{" "}
              <Link href="/cookie-settings">cookie settings</Link> page.
            </p>
            <p>
              <strong>From your browser:</strong> modern browsers offer cookie
              management in their settings (e.g. Chrome: Settings → Privacy →
              Cookies; Firefox: Preferences → Privacy & Security). Note that
              blocking strictly necessary cookies may break parts of the site.
            </p>
            <p>
              <strong>Do Not Track:</strong> we honour browser-issued
              Global Privacy Control (GPC) signals as a withdrawal of consent
              to analytics cookies, where supported.
            </p>
          </section>

          <section>
            <h2>International transfers</h2>
            <p>
              Some cookies may transmit data to third-party providers based
              outside the European Economic Area (typically the United
              States). See our <Link href="/privacy">Privacy Policy</Link>{" "}
              section 7 (International transfers) for the safeguards in place
              (Standard Contractual Clauses, EU-US Data Privacy Framework).
            </p>
          </section>

          <section>
            <h2>Contact</h2>
            <p>
              For questions about cookies or to exercise your rights, contact:{" "}
              <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>
            </p>
          </section>

          <section>
            <h2>Changes</h2>
            <p>
              This policy may be updated as new technologies are introduced or
              removed. The current version is always available on this page.
            </p>
          </section>

          <p className="text-sm italic opacity-60 text-center mt-12">
            Last updated: {{DATE}}
          </p>

          <div className="text-center pt-8 border-t opacity-70">
            <Link href="/">← Back to home</Link>
          </div>
        </article>
      </div>
    </main>
  );
}
```

---

## Placeholder fill guide

### `{{COOKIE_TABLE}}`

A table — one row per detected cookie or storage item:

```html
<div className="overflow-x-auto">
  <table className="cookie-table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Category</th>
        <th>Purpose</th>
        <th>Duration</th>
        <th>Provider</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><code>sb-auth-token</code></td>
        <td>Strictly necessary</td>
        <td>Supabase authentication session</td>
        <td>Session</td>
        <td>Supabase</td>
      </tr>
      <tr>
        <td><code>cookie-consent</code></td>
        <td>Strictly necessary</td>
        <td>Stores your cookie preference (localStorage)</td>
        <td>Until cleared by user</td>
        <td>First party</td>
      </tr>
      <tr>
        <td><code>_vercel_*</code></td>
        <td>Analytics</td>
        <td>Vercel Analytics — page view metrics</td>
        <td>1 year</td>
        <td>Vercel</td>
      </tr>
      <tr>
        <td><code>_ga, _gid</code></td>
        <td>Analytics</td>
        <td>Google Analytics — usage statistics</td>
        <td>2 years / 24 hours</td>
        <td>Google</td>
      </tr>
    </tbody>
  </table>
</div>
```

CSS:
```css
.cookie-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
}
.cookie-table th,
.cookie-table td {
  text-align: left;
  padding: 0.6rem 0.8rem;
  border-bottom: 1px solid rgba(180, 180, 180, 0.15);
}
.cookie-table th {
  font-weight: 500;
  opacity: 0.75;
  font-size: 0.7rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}
.cookie-table code {
  font-family: var(--font-mono, ui-monospace, monospace);
  font-size: 0.78rem;
  opacity: 0.9;
}
```

### `{{ANALYTICS_BLOCK}}` examples

Vercel Analytics:
```html
<p>
  <strong>Vercel Analytics</strong> — collects anonymous metrics such as
  page views and average session duration. Does not collect personally
  identifying data. Operated by Vercel Inc. (United States), transfer
  governed by Standard Contractual Clauses.{" "}
  <a href="https://vercel.com/legal/privacy-policy" target="_blank" rel="noopener">
    Vercel Privacy Policy
  </a>
</p>
```

Google Analytics 4:
```html
<p>
  <strong>Google Analytics 4</strong> — measures user interactions to help
  us understand site usage. Operated by Google Ireland Ltd. (EU controller)
  with infrastructure that may transfer data to Google LLC (United States).
  Transfer is governed by Standard Contractual Clauses and the
  EU-US Data Privacy Framework. IP addresses are anonymised before storage.{" "}
  <a href="https://policies.google.com/privacy" target="_blank" rel="noopener">
    Google Privacy Policy
  </a>
</p>
```

PostHog:
```html
<p>
  <strong>PostHog</strong> — product analytics. Hosted in EU or US
  region depending on project configuration. Operated by PostHog Inc.{" "}
  <a href="https://posthog.com/privacy" target="_blank" rel="noopener">
    PostHog Privacy Policy
  </a>
</p>
```

Plausible / Umami (privacy-friendly analytics):
```html
<p>
  <strong>Plausible Analytics</strong> — privacy-friendly,
  cookie-less analytics. No personal data is collected; only aggregated
  usage statistics. Hosted in the EU.{" "}
  <a href="https://plausible.io/privacy" target="_blank" rel="noopener">
    Plausible Privacy Policy
  </a>
</p>
```

---

*This template is not legal advice. Before going live, review by a qualified lawyer is recommended.*
