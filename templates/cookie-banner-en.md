# Cookie Banner — English Variant

For English deployments. The **architecture is identical** to the Turkish banner (`cookie-banner.md`) — same `cookieConsent.ts`, `useCookieConsent.ts`, `ConsentGate.tsx`. Only the **UI strings** and **paths** change.

**Placeholders**:
- `{{PROJECT_NAME}}`
- `{{POLICY_URL}}` — `/cookie-policy`
- `{{SETTINGS_URL}}` — `/cookie-settings` (optional)

---

## What stays the same from the TR template

Copy these files **unchanged** from `cookie-banner.md`:
- `lib/cookieConsent.ts` — types, helpers (`loadConsent`, `saveConsent`, `acceptAll`, `rejectAll`)
- `lib/useCookieConsent.ts` — hook
- `components/ConsentGate.tsx` — wrapper (`requires="analytics"` etc.)

The localStorage key, event name, and `ConsentState` shape are language-neutral.

---

## What changes — UI components

### `components/CookieBanner.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";
import Link from "next/link";
import {
  loadConsent,
  saveConsent,
  acceptAll,
  rejectAll,
  type ConsentState,
  DEFAULT_CONSENT,
} from "@/lib/cookieConsent";
import CookieSettings from "./CookieSettings";

export default function CookieBanner() {
  const [visible, setVisible] = useState(false);
  const [expanded, setExpanded] = useState(false);
  const [state, setState] = useState<ConsentState>(DEFAULT_CONSENT);

  useEffect(() => {
    const existing = loadConsent();
    if (!existing) {
      setVisible(true);
    } else {
      setState(existing);
    }
  }, []);

  if (!visible) return null;

  const onAcceptAll = () => {
    acceptAll();
    setVisible(false);
  };

  const onRejectAll = () => {
    rejectAll();
    setVisible(false);
  };

  const onSaveCustom = () => {
    saveConsent(state);
    setVisible(false);
  };

  return (
    <div className="cookie-banner" role="dialog" aria-label="Cookie preferences">
      {!expanded ? (
        <div className="cookie-banner__compact">
          <p>
            We use strictly necessary cookies to operate the service. With your
            consent, we also use analytics cookies to improve it.{" "}
            <Link href="{{POLICY_URL}}">Cookie Policy</Link>.
          </p>
          <div className="cookie-banner__actions">
            <button onClick={onRejectAll} className="cookie-banner__btn cookie-banner__btn--secondary">
              Necessary only
            </button>
            <button onClick={() => setExpanded(true)} className="cookie-banner__btn cookie-banner__btn--secondary">
              Customise
            </button>
            <button onClick={onAcceptAll} className="cookie-banner__btn cookie-banner__btn--primary">
              Accept all
            </button>
          </div>
        </div>
      ) : (
        <div className="cookie-banner__expanded">
          <h2>Cookie preferences</h2>
          <CookieSettings state={state} onChange={setState} />
          <div className="cookie-banner__actions">
            <button onClick={() => setExpanded(false)} className="cookie-banner__btn cookie-banner__btn--secondary">
              Back
            </button>
            <button onClick={onSaveCustom} className="cookie-banner__btn cookie-banner__btn--primary">
              Save preferences
            </button>
          </div>
        </div>
      )}
    </div>
  );
}
```

### `components/CookieSettings.tsx`

```tsx
"use client";

import type { CookieCategory, ConsentState } from "@/lib/cookieConsent";

interface CategoryDefinition {
  key: CookieCategory;
  label: string;
  description: string;
  required: boolean;
}

// Only include categories the project actually uses
const CATEGORIES: CategoryDefinition[] = [
  {
    key: "necessary",
    label: "Strictly necessary",
    description:
      "Required for the basic operation of the service (authentication, language, your cookie preference). Cannot be disabled.",
    required: true,
  },
  {
    key: "functional",
    label: "Functional",
    description:
      "Remembers preferences such as theme or layout to improve your experience.",
    required: false,
  },
  {
    key: "analytics",
    label: "Analytics",
    description:
      "Helps us understand how the service is used through aggregated, often anonymised statistics.",
    required: false,
  },
  {
    key: "marketing",
    label: "Marketing",
    description:
      "Used to deliver targeted advertising. Not currently in use on this site.",
    required: false,
  },
];

interface Props {
  state: ConsentState;
  onChange: (newState: ConsentState) => void;
}

export default function CookieSettings({ state, onChange }: Props) {
  const toggle = (key: CookieCategory) => {
    if (key === "necessary") return;
    onChange({ ...state, [key]: !state[key] });
  };

  return (
    <div className="cookie-settings">
      {CATEGORIES.map((cat) => (
        <div key={cat.key} className="cookie-settings__row">
          <div className="cookie-settings__info">
            <h3>{cat.label}</h3>
            <p>{cat.description}</p>
          </div>
          <label className="cookie-settings__toggle">
            <input
              type="checkbox"
              checked={state[cat.key]}
              disabled={cat.required}
              onChange={() => toggle(cat.key)}
              aria-label={`${cat.label} cookies`}
            />
            <span className="cookie-settings__slider" aria-hidden="true" />
          </label>
        </div>
      ))}
    </div>
  );
}
```

### `app/cookie-settings/page.tsx` — Re-edit preferences page

```tsx
"use client";

import { useEffect, useState } from "react";
import Link from "next/link";
import {
  loadConsent,
  saveConsent,
  type ConsentState,
  DEFAULT_CONSENT,
} from "@/lib/cookieConsent";
import CookieSettings from "@/components/CookieSettings";

export default function CookieSettingsPage() {
  const [state, setState] = useState<ConsentState>(DEFAULT_CONSENT);
  const [saved, setSaved] = useState(false);

  useEffect(() => {
    const existing = loadConsent();
    if (existing) setState(existing);
  }, []);

  const onSave = () => {
    saveConsent(state);
    setSaved(true);
    setTimeout(() => setSaved(false), 2500);
  };

  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-2xl mx-auto">
        <header className="mb-10">
          <h1 className="text-2xl md:text-3xl font-light tracking-wide">Cookie settings</h1>
          <p className="text-sm opacity-70 mt-2">
            Manage which cookies {{PROJECT_NAME}} may use. See the{" "}
            <Link href="{{POLICY_URL}}">Cookie Policy</Link> for details about
            each category.
          </p>
        </header>

        <CookieSettings state={state} onChange={setState} />

        <div className="mt-8 flex items-center gap-4">
          <button onClick={onSave} className="cookie-banner__btn cookie-banner__btn--primary">
            Save preferences
          </button>
          {saved && <span className="text-sm opacity-75">Saved ✓</span>}
        </div>

        <p className="mt-10 text-sm opacity-60">
          Your preference is stored on your device under the key{" "}
          <code>cookie-consent</code>. Clearing your browser data will reset it.
        </p>
      </div>
    </main>
  );
}
```

---

## Strings reference (for translating quickly)

| TR | EN |
|---|---|
| Çerez Politikası | Cookie Policy |
| Çerez ayarları | Cookie settings |
| Tümünü kabul et | Accept all |
| Sadece zorunlu | Necessary only |
| Özelleştir | Customise |
| Tercihleri kaydet | Save preferences |
| Geri | Back |
| Kesinlikle gerekli | Strictly necessary |
| İşlevsel | Functional |
| Analitik | Analytics |
| Pazarlama | Marketing |
| Kaydedildi | Saved |

---

## Notes

- Use the same CSS classes (`.cookie-banner`, `.cookie-settings`, etc.) — see the TR template's CSS block. No changes needed.
- The Footer's English variant should link to `/cookie-policy` and `/cookie-settings` instead of the TR equivalents. See `footer-en.md`.
- For bilingual sites (TR + EN with locale switching), use one banner component that pulls strings from a translation file. The architecture supports this without changes to the consent layer.

---

*This template is not legal advice. Before going live, review by a qualified lawyer is recommended.*
