# Privacy Policy Template (English / GDPR)

Used as `app/privacy/page.tsx` in Next.js App Router. Adapt to HTML/Markdown for other frameworks.

This template implements **GDPR Article 13 / 14** information notice (data subjects' right to be informed) and follows the EDPB Guidelines on Transparency.

**Placeholders** (Claude fills these in):
- `{{PROJECT_NAME}}` — site/service name (e.g. "Bloom Within")
- `{{DATA_CONTROLLER_NAME}}` — natural person name or legal entity
- `{{DATA_CONTROLLER_LOCATION}}` — city/country (e.g. "Istanbul, Turkey")
- `{{CONTACT_EMAIL}}` — privacy contact email
- `{{COLLECTED_DATA_LIST}}` — markdown list (e.g. "- Name\n- Email")
- `{{PROCESSING_PURPOSE_LIST}}` — markdown list (project-specific purposes)
- `{{LAWFUL_BASIS}}` — Article 6 basis: "consent" / "contract" / "legitimate interests" with reasoning
- `{{THIRD_PARTY_BLOCK}}` — one paragraph per recipient (Anthropic, Supabase, Vercel, etc.)
- `{{RETENTION_STATEMENT}}` — "not stored" or retention periods per data category
- `{{TRANSFER_BLOCK}}` — international transfers (Art 44-49) with safeguards
- `{{COOKIE_STATEMENT}}` — "no cookies" or link to cookie policy
- `{{SUPERVISORY_AUTHORITY_BLOCK}}` — DPA information for relevant jurisdiction
- `{{DATE}}` — last updated date (ISO format)

---

## `app/privacy/page.tsx`

```tsx
import type { Metadata } from "next";
import Link from "next/link";

export const metadata: Metadata = {
  title: "Privacy Policy — {{PROJECT_NAME}}",
  description: "Information about how {{PROJECT_NAME}} processes personal data under the GDPR.",
};

export default function PrivacyPolicyPage() {
  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-3xl mx-auto">
        <header className="text-center mb-12">
          <h1 className="text-3xl md:text-4xl font-light tracking-wide">Privacy Policy</h1>
          <p className="text-sm opacity-70 mt-2">How we process your personal data</p>
        </header>

        <article className="prose prose-invert max-w-none space-y-6">
          <p className="border-l-2 pl-4 italic opacity-90">
            This notice is provided under <strong>Articles 13 and 14 of the
            General Data Protection Regulation (EU 2016/679, &ldquo;GDPR&rdquo;)</strong>
            to inform you about how we process your personal data when you use {{PROJECT_NAME}}.
          </p>

          <section>
            <h2>1. Data Controller</h2>
            <p>
              {{PROJECT_NAME}} is operated by{" "}
              <strong>{{DATA_CONTROLLER_NAME}}</strong> ({{DATA_CONTROLLER_LOCATION}}),
              acting as the data controller within the meaning of GDPR Art. 4(7).
            </p>
            <p>
              <strong>Contact for privacy matters:</strong>{" "}
              <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>
            </p>
            <p className="text-sm opacity-70">
              No Data Protection Officer (DPO) has been appointed because the
              criteria of GDPR Art. 37(1) are not met. You may contact the email
              above for any data protection matter.
            </p>
          </section>

          <section>
            <h2>2. Personal Data We Process</h2>
            <p>We process the following categories of personal data:</p>
            <ul>
{{COLLECTED_DATA_LIST}}
            </ul>
            <p>
              All data is provided by you voluntarily through forms, registration
              or interaction with the service. We do not acquire your data from
              other sources or enrich it with third-party data.
            </p>
          </section>

          <section>
            <h2>3. Purposes of Processing</h2>
            <p>Your data is processed only for the following purposes:</p>
            <ul>
{{PROCESSING_PURPOSE_LIST}}
            </ul>
            <p>
              We do <strong>not</strong> use your data for marketing, profiling,
              advertising, resale to third parties, statistical analysis beyond
              service operation, or any incompatible purpose.
            </p>
          </section>

          <section>
            <h2>4. Lawful Basis (GDPR Art. 6)</h2>
{{LAWFUL_BASIS}}
            <p className="text-sm opacity-70">
              Where processing is based on your consent (Art. 6(1)(a)), you may
              withdraw it at any time without affecting the lawfulness of
              processing carried out before withdrawal.
            </p>
          </section>

          <section>
            <h2>5. Retention Period (GDPR Art. 5(1)(e))</h2>
{{RETENTION_STATEMENT}}
          </section>

          <section>
            <h2>6. Recipients of Personal Data</h2>
{{THIRD_PARTY_BLOCK}}
            <p className="text-sm opacity-70">
              Each of these recipients acts as a processor or independent
              controller as defined by GDPR Art. 4(8)/(7). We share with them only
              the minimum data necessary for the stated purpose.
            </p>
          </section>

          <section>
            <h2>7. International Data Transfers (GDPR Art. 44-49)</h2>
{{TRANSFER_BLOCK}}
          </section>

          <section>
            <h2>8. Your Rights as a Data Subject (GDPR Art. 15-22)</h2>
            <p>Under the GDPR, you have the following rights:</p>
            <ul>
              <li><strong>Right of access</strong> (Art. 15) — to obtain confirmation of whether your data is being processed and a copy of it</li>
              <li><strong>Right to rectification</strong> (Art. 16) — to have inaccurate data corrected</li>
              <li><strong>Right to erasure / right to be forgotten</strong> (Art. 17) — to have your data deleted</li>
              <li><strong>Right to restriction of processing</strong> (Art. 18) — to limit how your data is processed</li>
              <li><strong>Right to data portability</strong> (Art. 20) — to receive your data in a structured, machine-readable format</li>
              <li><strong>Right to object</strong> (Art. 21) — to processing based on legitimate interests or for direct marketing</li>
              <li><strong>Rights related to automated decision-making</strong> (Art. 22) — including profiling that has legal or similarly significant effects</li>
              <li><strong>Right to withdraw consent</strong> (Art. 7(3)) — where processing is based on consent</li>
            </ul>
            <p>
              To exercise any of these rights, contact us at{" "}
              <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>. We will
              respond within one month (extendable by two further months for
              complex requests, GDPR Art. 12(3)).
            </p>
          </section>

          <section>
            <h2>9. Right to Lodge a Complaint</h2>
{{SUPERVISORY_AUTHORITY_BLOCK}}
          </section>

          <section>
            <h2>10. Cookies and Tracking</h2>
{{COOKIE_STATEMENT}}
          </section>

          <section>
            <h2>11. Automated Decision-Making</h2>
            <p>
              We do not make decisions based solely on automated processing that
              produce legal effects or similarly significantly affect you (GDPR
              Art. 22). Where AI is used to generate content for you, you remain
              free to accept, ignore or contest the output.
            </p>
          </section>

          <section>
            <h2>12. Security (GDPR Art. 32)</h2>
            <p>
              We apply appropriate technical and organisational measures to
              protect your data, including TLS encryption in transit, access
              controls, and the principle of least privilege for server-side
              keys. No system is perfectly secure; in the event of a personal
              data breach affecting your rights and freedoms we will notify the
              competent supervisory authority within 72 hours (Art. 33) and you
              without undue delay where required (Art. 34).
            </p>
          </section>

          <section>
            <h2>13. Children</h2>
            <p>
              This service is not directed at children under 16 years of age
              (GDPR Art. 8). If you are under 16, please do not provide personal
              data without your parent or guardian's consent.
            </p>
          </section>

          <section>
            <h2>14. Changes to this Policy</h2>
            <p>
              The current version of this policy is always available on this
              page. Material changes will be highlighted with an updated date
              and, where appropriate, a separate notice.
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

### `{{COLLECTED_DATA_LIST}}` example

Based on detected PII fields:
```html
              <li>Name and surname</li>
              <li>Email address</li>
              <li>Date of birth</li>
              <li>User-generated content (posts, comments, uploads)</li>
              <li>IP address (server logs only)</li>
```

### `{{PROCESSING_PURPOSE_LIST}}` example

Project-specific:
```html
              <li>Providing the service you requested</li>
              <li>Maintaining your user account</li>
              <li>Responding to your enquiries</li>
              <li>Ensuring service security and preventing abuse</li>
```

### `{{LAWFUL_BASIS}}` examples

For consent-based processing (forms, registration, optional data):
```html
            <p>
              We process your personal data on the basis of:
            </p>
            <ul>
              <li>
                <strong>Your consent</strong> (Art. 6(1)(a)) — for data you
                voluntarily provide through forms and registration. The consent
                checkbox before submission documents your explicit consent.
              </li>
              <li>
                <strong>Performance of a contract</strong> (Art. 6(1)(b)) —
                where processing is necessary to provide the service you have
                signed up for.
              </li>
              <li>
                <strong>Legitimate interests</strong> (Art. 6(1)(f)) — for
                essential security logging (IP addresses in server access logs)
                where this is necessary to protect the service from abuse.
              </li>
            </ul>
```

### `{{RETENTION_STATEMENT}}` examples

If no data is stored:
```html
            <p className="border-l-2 pl-4">
              We do <strong>not store</strong> your personal data in any
              database, file, or persistent memory. Data is held only in server
              memory during request processing and is automatically discarded
              when the request completes.
            </p>
```

If data is stored:
```html
            <p>
              Your account profile and user-generated content are retained for
              as long as your account is active. Upon account deletion, data is
              removed within 30 days, except where retention is required by law
              (e.g. server access logs are retained for 6 months for security
              purposes under legitimate interests).
            </p>
            <p>
              You may request deletion at any time via your{" "}
              <Link href="/account">account settings</Link> or by contacting{" "}
              <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>.
            </p>
```

### `{{THIRD_PARTY_BLOCK}}` example

One paragraph per detected third party:
```html
            <p className="border-l-2 pl-4">
              <strong>Anthropic, PBC</strong> (United States) — provides the
              Claude large language model used to generate AI responses. The
              data you submit for AI processing is transmitted to Anthropic's
              servers. According to Anthropic's policy, inputs may be retained
              for up to 30 days for safety auditing.{" "}
              <a href="https://www.anthropic.com/legal/privacy" target="_blank" rel="noopener">
                anthropic.com/legal/privacy
              </a>
            </p>

            <p>
              <strong>Supabase Inc.</strong> (United States, with EU hosting
              region available) — provides authentication and database
              infrastructure. Your account data and content are stored on
              Supabase-managed PostgreSQL.{" "}
              <a href="https://supabase.com/privacy" target="_blank" rel="noopener">
                supabase.com/privacy
              </a>
            </p>

            <p>
              <strong>Vercel Inc.</strong> (United States) — provides hosting
              and edge network. Server access logs (including IP addresses)
              are retained for operational purposes.{" "}
              <a href="https://vercel.com/legal/privacy-policy" target="_blank" rel="noopener">
                vercel.com/legal/privacy-policy
              </a>
            </p>
```

### `{{TRANSFER_BLOCK}}` example

When transfers to third countries occur:
```html
            <p>
              Some of our service providers are located outside the European
              Economic Area (EEA), in particular in the United States
              (Anthropic, Vercel, Supabase). These transfers are governed by:
            </p>
            <ul>
              <li>
                <strong>Standard Contractual Clauses (SCCs)</strong> as adopted
                by the European Commission (Implementing Decision 2021/914),
                where applicable
              </li>
              <li>
                The <strong>EU-US Data Privacy Framework</strong> for certified
                US providers (where applicable)
              </li>
              <li>
                Your <strong>explicit consent</strong> under Art. 49(1)(a) where
                no other safeguard applies
              </li>
            </ul>
            <p>
              You can request a copy of the safeguards in place by contacting
              us at <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>.
            </p>
```

When no third-country transfers:
```html
            <p>
              All processing takes place within the European Economic Area
              (EEA). No transfers to third countries occur.
            </p>
```

### `{{COOKIE_STATEMENT}}` examples

If no cookies:
```html
            <p>
              This service does not use tracking, analytics or marketing
              cookies. Only strictly necessary technical storage (such as your
              authentication session) is used, which does not require consent
              under ePrivacy Directive Art. 5(3).
            </p>
```

If cookies are used:
```html
            <p>
              We use cookies and similar technologies to operate the service
              and, with your consent, for analytics. Full details are in our{" "}
              <Link href="/cookie-policy">Cookie Policy</Link>.
            </p>
```

### `{{SUPERVISORY_AUTHORITY_BLOCK}}` examples

For EU/EEA data subjects:
```html
            <p>
              If you believe our processing of your personal data infringes the
              GDPR, you have the right to lodge a complaint with a supervisory
              authority (GDPR Art. 77). You may complain to the authority in
              your Member State of residence, place of work, or place of the
              alleged infringement.
            </p>
            <p>
              A list of EU supervisory authorities is available at:{" "}
              <a href="https://edpb.europa.eu/about-edpb/about-edpb/members_en" target="_blank" rel="noopener">
                edpb.europa.eu/about-edpb/about-edpb/members
              </a>
            </p>
```

If the data controller is also subject to KVKK (Turkey-based operator):
```html
            <p>
              For EU residents: you may also lodge a complaint with your
              national data protection authority. List of EU supervisory
              authorities:{" "}
              <a href="https://edpb.europa.eu/about-edpb/about-edpb/members_en" target="_blank" rel="noopener">
                edpb.europa.eu/about-edpb/about-edpb/members
              </a>
            </p>
            <p>
              For Turkish residents: complaints may also be lodged with the
              Personal Data Protection Authority (KVKK) at{" "}
              <a href="https://www.kvkk.gov.tr" target="_blank" rel="noopener">
                kvkk.gov.tr
              </a>. See our Turkish{" "}
              <Link href="/aydinlatma">Aydınlatma Metni</Link> for the
              KVKK-specific notice.
            </p>
```

---

*This template is not legal advice. Before going live — particularly for commercial activity, paid services, children's data, or sensitive data — review by a qualified GDPR/data protection lawyer is recommended.*
