---
template: terms-of-service
language: en
legal_basis:
  - gdpr_general
  - eu_consumer_rights_directive
jurisdiction: EU
last_updated: 2026-05-21
last_reviewed: 2026-05-21
review_cadence: 12_months
version: "0.3"
---

# Terms of Service Template (English)

Used as `app/terms/page.tsx` in Next.js App Router. Adapt for other frameworks.

**Placeholders**:
- `{{PROJECT_NAME}}`
- `{{DATA_CONTROLLER_NAME}}`
- `{{DATA_CONTROLLER_LOCATION}}` — city/country, used in governing law section
- `{{CONTACT_EMAIL}}`
- `{{SERVICE_DESCRIPTION}}` — short one-liner (e.g. "an AI-powered symbolic interpretation tool", "a social platform for food experimenters")
- `{{AI_BLOCK}}` — AI-specific disclaimer if generative content is part of the service
- `{{UGC_BLOCK}}` — user-generated content terms if applicable
- `{{ECOMMERCE_BLOCK}}` — purchase/refund terms if payments are accepted
- `{{AGE_LIMIT}}` — minimum age (default: 16 for EU service, 18 if 18+ topic)
- `{{DISCLAIMER_LIST}}` — health/finance/etc. specific disclaimers
- `{{GOVERNING_LAW_BLOCK}}` — jurisdiction-specific governing law clause
- `{{DATE}}`

---

## `app/terms/page.tsx`

```tsx
import type { Metadata } from "next";
import Link from "next/link";

export const metadata: Metadata = {
  title: "Terms of Service — {{PROJECT_NAME}}",
  description: "{{PROJECT_NAME}} terms of service and disclaimer.",
};

export default function TermsOfServicePage() {
  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-3xl mx-auto">
        <header className="text-center mb-12">
          <h1 className="text-3xl md:text-4xl font-light tracking-wide">Terms of Service</h1>
          <p className="text-sm opacity-70 mt-2">and limitation of liability</p>
        </header>

        <article className="prose prose-invert max-w-none space-y-6">
          <p className="border-l-2 pl-4 italic">
            By using this service, you agree to the terms below. If you do not
            agree, please do not use the service.
          </p>

          <section>
            <h2>1. The Service</h2>
            <p>
              {{PROJECT_NAME}} is {{SERVICE_DESCRIPTION}}. The content provided
              <strong> is not</strong>:
            </p>
            <ul>
{{DISCLAIMER_LIST}}
            </ul>
          </section>

{{AI_BLOCK}}

          <section>
            <h2>3. Age Requirement</h2>
            <p>
              This service is intended only for users who are at least{" "}
              <strong>{{AGE_LIMIT}} years old</strong>. Users under
              {{AGE_LIMIT}} require explicit consent from a parent or legal
              guardian; because we do not operate a verification mechanism for
              such consent, minors should not use the service. Where the GDPR
              applies, the age threshold for consent to information society
              services is 16 (Art. 8), or lower as set by the Member State of
              residence.
            </p>
          </section>

          <section>
            <h2>4. Limitation of Liability</h2>
            <p>
              To the maximum extent permitted by applicable law,{" "}
              {{DATA_CONTROLLER_NAME}} shall not be liable for any direct,
              indirect, incidental, consequential, special, or punitive
              damages arising from your use of, or inability to use, the
              service. This includes — but is not limited to — decisions made
              or not made on the basis of content provided, financial loss,
              health-related outcomes, relationship consequences, or any
              third-party action.
            </p>
            <p>
              Service interruptions, delays, or data loss arising from
              servers, networks, or third-party service providers fall outside
              our liability.
            </p>
            <p className="text-sm opacity-70">
              Nothing in this clause excludes liability that cannot be
              excluded under applicable consumer protection law.
            </p>
          </section>

          <section>
            <h2>5. Intellectual Property</h2>
            <p>
              The visual identity, written compositions, architecture, code,
              and templates of the service are the property of{" "}
              {{DATA_CONTROLLER_NAME}} (or its licensors).
            </p>
            <p>
              Content generated for you may be freely read, stored, or shared
              with attribution for your personal use. Commercial republication,
              bulk automated access, or scraping for training datasets is
              prohibited without prior written permission.
            </p>
          </section>

{{UGC_BLOCK}}

          <section>
            <h2>6. Prohibited Use</h2>
            <p>The following actions are prohibited:</p>
            <ul>
              <li>Submitting another person's personal data without their consent</li>
              <li>Automated access (bots, scrapers, mass requests) that imposes load on the service</li>
              <li>Reverse-engineering or attempting to replicate the service</li>
              <li>Using the service to manipulate, harm, harass, or defraud others</li>
              <li>Uploading content that infringes intellectual property, violates law, or contains malware</li>
            </ul>
            <p>
              We may suspend or terminate access for users who violate these
              terms, without prior notice where the breach is material.
            </p>
          </section>

{{ECOMMERCE_BLOCK}}

          <section>
            <h2>7. Service Availability</h2>
            <p>
              The service is provided on an &ldquo;as is&rdquo; and &ldquo;as
              available&rdquo; basis. We do not guarantee that the service
              will be uninterrupted, error-free, or available at any specific
              time. We may modify, suspend, or discontinue any part of the
              service at our discretion.
            </p>
          </section>

          <section>
            <h2>8. Privacy and Data Protection</h2>
            <p>
              Our processing of personal data is described in our{" "}
              <Link href="/privacy">Privacy Policy</Link>. Your rights under
              the GDPR (access, rectification, erasure, portability, objection,
              withdraw consent) are described there and exercisable via
              <a href="mailto:{{CONTACT_EMAIL}}"> {{CONTACT_EMAIL}}</a>.
            </p>
          </section>

          <section>
            <h2>9. Feedback and Contact</h2>
            <p>
              For technical issues or feedback:{" "}
              <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>
            </p>
          </section>

          <section>
            <h2>10. Governing Law and Jurisdiction</h2>
{{GOVERNING_LAW_BLOCK}}
          </section>

          <section>
            <h2>11. Changes</h2>
            <p>
              These terms may be updated as needed; the current version is
              always available on this page. Material changes will be
              communicated through the service or by email where applicable.
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

### `{{AI_BLOCK}}` — when AI generates content

```html
          <section>
            <h2>2. AI-Generated Content</h2>
            <p>
              Responses, interpretations, or other outputs produced by the
              service are generated in real time by third-party large language
              models. Such models are probabilistic; the same input may
              produce different outputs at different times. Generated text may
              contain factual errors, logical inconsistencies, or unexpected
              statements.
            </p>
            <p>
              The service makes <strong>no warranty</strong> as to the
              accuracy, consistency, or fitness for any specific purpose of
              AI-generated content. You should not rely on it as a substitute
              for professional advice (medical, legal, financial, or otherwise).
            </p>
          </section>
```

### `{{UGC_BLOCK}}` — when users post/upload content

```html
          <section>
            <h2>5a. User-Generated Content</h2>
            <p>
              Content uploaded by users (text, images, recipes, comments,
              uploads) is the responsibility of the uploading user.
              {{PROJECT_NAME}} does not actively pre-screen content but
              reserves the right to:
            </p>
            <ul>
              <li>Remove content that is harassing, discriminatory, illegal, or violates these terms</li>
              <li>Receive notice-and-takedown requests for copyright infringement at <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a></li>
              <li>Cooperate with law enforcement regarding content that may constitute a criminal offence</li>
              <li>Terminate accounts of repeat infringers without notice</li>
            </ul>
            <p>
              By uploading content, you represent that you own the rights or
              have obtained necessary permissions. You grant {{PROJECT_NAME}}
              a non-exclusive, worldwide, royalty-free licence to host,
              display, and distribute your content to the extent necessary to
              provide the service.
            </p>
            <p>
              For EU users: under the Digital Services Act (Regulation
              2022/2065), illegal content can be reported via the contact
              email above, and we will act expeditiously upon obtaining actual
              knowledge.
            </p>
          </section>
```

### `{{ECOMMERCE_BLOCK}}` — when payments are accepted

```html
          <section>
            <h2>6a. Purchases and Right of Withdrawal</h2>
            <p>
              For purchases by EU/EEA consumers, the right of withdrawal under
              Directive 2011/83/EU applies. You may withdraw from a contract
              within <strong>14 days</strong> of conclusion without giving any
              reason, by sending a clear statement to{" "}
              <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>.
            </p>
            <p>
              The right of withdrawal does not apply where digital content is
              supplied immediately and you have provided express prior consent
              and acknowledgement that this causes the withdrawal right to be
              lost (Art. 16(m) Directive 2011/83/EU).
            </p>
            <p>
              <strong>Refunds:</strong> processed within 14 days of receipt of
              your withdrawal statement, using the same payment method.
            </p>
            <p>
              <strong>Dispute resolution:</strong> EU consumers may use the
              European Commission's Online Dispute Resolution (ODR) platform
              at{" "}
              <a href="https://ec.europa.eu/consumers/odr" target="_blank" rel="noopener">
                ec.europa.eu/consumers/odr
              </a>.
            </p>
          </section>
```

### `{{DISCLAIMER_LIST}}` — based on detected risk areas

If health claims are detected:
```html
              <li>A substitute for medical, psychological, or psychiatric diagnosis or treatment</li>
              <li>A basis for any health-related decision</li>
```

If divination/spiritual claims detected:
```html
              <li>A claim of prophecy, fortune-telling, or predicting the future</li>
              <li>A religious ruling, fatwa, or doctrinal teaching</li>
```

If financial claims detected:
```html
              <li>Investment, legal, or financial advice</li>
              <li>Regulated financial advisory service within the meaning of MiFID II or any other financial regulation</li>
```

If educational/certification claims:
```html
              <li>An accredited educational or certification programme</li>
              <li>A formal academic qualification</li>
```

If wellness/spiritual (meditation, chakra, breathwork type):
```html
              <li>A medical treatment for anxiety, depression, or any condition</li>
              <li>A substitute for therapy or professional mental health care</li>
```

### `{{GOVERNING_LAW_BLOCK}}` examples

For Turkey-based operator serving EU:
```html
            <p>
              These terms are governed by the laws of the Republic of Turkey.
              Any dispute arising out of or in connection with these terms
              shall be submitted to the exclusive jurisdiction of the courts
              of {{DATA_CONTROLLER_LOCATION}}.
            </p>
            <p>
              For consumers resident in the European Union, this clause does
              not deprive you of the protection afforded by mandatory
              consumer protection rules of your country of habitual residence
              under Art. 6 of the Rome I Regulation (593/2008).
            </p>
```

For EU-based operator:
```html
            <p>
              These terms are governed by the laws of {{DATA_CONTROLLER_LOCATION}}.
              Any dispute arising out of or in connection with these terms
              shall be submitted to the competent courts of the same
              jurisdiction, without prejudice to mandatory consumer
              protections under Art. 6 Rome I.
            </p>
```

---

*This template is not legal advice. Before going live — particularly for paid services, e-commerce, or sensitive content — review by a qualified lawyer is recommended.*
