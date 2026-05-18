# Consent Checkbox Template (English / GDPR)

A required consent checkbox to place before form submission. Implements **GDPR Art. 7** (conditions for consent) — freely given, specific, informed, unambiguous, by a clear affirmative action.

**Placeholders**:
- `{{PROCESSING_SUMMARY}}` — one sentence on what is done (e.g. "generate a symbolic interpretation")
- `{{THIRD_PARTY_SHORT}}` — brief list, comma-separated: "Anthropic (USA), Supabase (USA)"
- `{{AGE_LIMIT}}` — 16 (GDPR default) or 18 (for 18+ content)
- `{{AI_RISK_LINE}}` — optional line acknowledging AI-generated content limitations

---

## React/TSX (Next.js)

```tsx
// State (place at the top of the component)
const [consentGiven, setConsentGiven] = useState(false);

// Inside the form, BEFORE the submit button
<label className="consent-box">
  <input
    type="checkbox"
    checked={consentGiven}
    onChange={(e) => setConsentGiven(e.target.checked)}
    required
  />
  <span className="consent-box__indicator" aria-hidden="true" />
  <span className="consent-box__text">
    I am at least {{AGE_LIMIT}} years old. I give my{" "}
    <strong>explicit consent</strong> (GDPR Art. 6(1)(a)) for the data
    I have entered above to be processed for the purpose of{" "}
    <strong>{{PROCESSING_SUMMARY}}</strong>, and transferred for this
    purpose to <strong>{{THIRD_PARTY_SHORT}}</strong>.{" "}
    {{AI_RISK_LINE}}{" "}
    <Link href="/privacy" target="_blank" className="consent-link">
      Privacy Policy
    </Link>
    {" · "}
    <Link href="/terms" target="_blank" className="consent-link">
      Terms of Service
    </Link>
  </span>
</label>

// In the submit handler
const submitForm = () => {
  if (!consentGiven) {
    setError("Please check the consent box to continue.");
    return;
  }
  // ... existing submit logic
};

// Disable the submit button until consent is given
<button type="submit" disabled={submitting || !consentGiven}>
  ...
</button>
```

### `{{AI_RISK_LINE}}` examples

When AI generates content:
```
"I understand this content is AI-generated and is not a substitute for medical, legal, or financial advice."
```

For wellness/spiritual services:
```
"I understand this is a contemplative practice tool and not a medical treatment."
```

If no specific AI risk: leave empty.

### CSS

```css
.consent-box {
  display: flex;
  gap: 0.85rem;
  align-items: flex-start;
  cursor: pointer;
  padding: 0.85rem 1rem;
  border: 1px solid rgba(180, 180, 180, 0.2);
  border-radius: 0.5rem;
  transition: all 0.3s ease;
}

.consent-box:hover {
  border-color: rgba(180, 180, 180, 0.35);
}

.consent-box input[type="checkbox"] {
  position: absolute;
  opacity: 0;
  pointer-events: none;
}

.consent-box__indicator {
  flex-shrink: 0;
  width: 1.1rem;
  height: 1.1rem;
  margin-top: 0.15rem;
  border: 1.5px solid currentColor;
  border-radius: 0.2rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s ease;
}

.consent-box input:checked + .consent-box__indicator::after {
  content: "";
  width: 0.55rem;
  height: 0.55rem;
  background: currentColor;
  border-radius: 0.1rem;
}

.consent-box__text {
  font-size: 0.82rem;
  line-height: 1.55;
  opacity: 0.85;
}

.consent-link {
  text-decoration: underline;
  text-underline-offset: 2px;
}
```

---

## Plain HTML

```html
<label class="consent-box">
  <input type="checkbox" name="consent" required />
  <span>
    I am at least {{AGE_LIMIT}} years old. I give my explicit consent (GDPR
    Art. 6(1)(a)) for processing of my data to {{PROCESSING_SUMMARY}}, and
    transfer to {{THIRD_PARTY_SHORT}}.
    <a href="/privacy" target="_blank">Privacy Policy</a> ·
    <a href="/terms" target="_blank">Terms</a>
  </span>
</label>
```

---

## Important — GDPR Art. 7 requirements

For consent to be valid under the GDPR:

1. **Pre-ticked boxes are invalid** (Recital 32) — checkbox must default to unchecked
2. **Bundled consent is invalid** — separate consents for separate purposes (don't combine "I consent to processing" with "I consent to marketing")
3. **Withdrawal must be as easy as giving consent** (Art. 7(3)) — provide a clear withdrawal mechanism (e.g. via `/account`)
4. **Record of consent** — log timestamp, IP, and the exact text shown (server-side, do not surface to client)
5. **Children under 16** — parental consent is required for information society services (Art. 8); add an age verification or block sign-up below the threshold

---

*This template is not legal advice. Before going live, review by a qualified lawyer is recommended.*
