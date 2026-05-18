# GDPR Data Subject Rights — Code Templates

For any service that stores personal data, **working endpoints** are required for the data subject rights under GDPR Articles 15-22. A passive "email us" approach is insufficient under Art. 12(2) (controller shall facilitate the exercise of rights).

This template provides:
1. **`/account` panel** — user dashboard for viewing and managing their data
2. **Data export** (JSON) — Art. 15 (access) + Art. 20 (portability)
3. **Account deletion** — Art. 17 (right to erasure / right to be forgotten)
4. **Rectification** — Art. 16, via inline profile edit

**Placeholders**:
- `{{PROJECT_NAME}}`
- `{{CONTACT_EMAIL}}`
- `{{USER_TABLES}}` — tables containing user data (Claude detects from schema)

---

## 1. `/account` page — Generic React + Supabase

`app/account/page.tsx`:

```tsx
"use client";

import { useEffect, useState } from "react";
import Link from "next/link";
import { createClient } from "@/lib/supabase/client";

interface UserProfile {
  id: string;
  email: string;
  username?: string;
  display_name?: string;
  created_at: string;
}

export default function AccountPage() {
  const [profile, setProfile] = useState<UserProfile | null>(null);
  const [loading, setLoading] = useState(true);
  const [deleteConfirm, setDeleteConfirm] = useState(false);
  const [deleteText, setDeleteText] = useState("");
  const [deleting, setDeleting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const supabase = createClient();

  useEffect(() => {
    const load = async () => {
      const { data: { user } } = await supabase.auth.getUser();
      if (!user) {
        window.location.href = "/login";
        return;
      }
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("id", user.id)
        .single();
      if (error) {
        setError("Could not load profile");
      } else {
        setProfile({ ...data, email: user.email ?? "" });
      }
      setLoading(false);
    };
    load();
  }, [supabase]);

  const exportData = async () => {
    setError(null);
    try {
      const res = await fetch("/api/me/export");
      if (!res.ok) throw new Error("Export failed");
      const blob = await res.blob();
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = `{{PROJECT_NAME}}-my-data-${new Date().toISOString().split("T")[0]}.json`;
      a.click();
      URL.revokeObjectURL(url);
    } catch (err) {
      setError(err instanceof Error ? err.message : "An error occurred");
    }
  };

  const deleteAccount = async () => {
    if (deleteText !== "DELETE") {
      setError("Confirmation text does not match");
      return;
    }
    setDeleting(true);
    setError(null);
    try {
      const res = await fetch("/api/me", { method: "DELETE" });
      if (!res.ok) {
        const body = await res.json().catch(() => ({}));
        throw new Error(body.error ?? "Deletion failed");
      }
      await supabase.auth.signOut();
      window.location.href = "/?deleted=1";
    } catch (err) {
      setError(err instanceof Error ? err.message : "An error occurred");
      setDeleting(false);
    }
  };

  if (loading) {
    return <main className="min-h-screen flex items-center justify-center">Loading…</main>;
  }

  if (!profile) {
    return <main className="min-h-screen flex items-center justify-center">Profile not found</main>;
  }

  return (
    <main className="min-h-screen px-6 md:px-8 py-16">
      <div className="max-w-2xl mx-auto space-y-12">
        <header>
          <h1 className="text-3xl font-light tracking-wide">My account</h1>
          <p className="text-sm opacity-70 mt-2">
            Exercise your rights under GDPR Articles 15-22 here.
          </p>
        </header>

        {/* Information display — Art. 15 (right of access) */}
        <section className="space-y-3">
          <h2 className="text-lg font-medium">Account information</h2>
          <div className="info-row">
            <span className="info-label">Email</span>
            <span>{profile.email}</span>
          </div>
          {profile.username && (
            <div className="info-row">
              <span className="info-label">Username</span>
              <span>{profile.username}</span>
            </div>
          )}
          <div className="info-row">
            <span className="info-label">Registered</span>
            <span>{new Date(profile.created_at).toLocaleDateString("en-GB")}</span>
          </div>
        </section>

        {/* Data export — Art. 15 + Art. 20 */}
        <section className="border-t pt-8">
          <h2 className="text-lg font-medium mb-2">Download my data</h2>
          <p className="text-sm opacity-75 mb-4">
            Download all data associated with your account in JSON format. This
            file can be used to migrate to another platform (Art. 20 right to
            data portability) or kept as a personal archive.
          </p>
          <button
            type="button"
            onClick={exportData}
            className="rights-btn"
          >
            Download as JSON
          </button>
        </section>

        {/* Rectification — Art. 16 */}
        <section className="border-t pt-8">
          <h2 className="text-lg font-medium mb-2">Correct my information</h2>
          <p className="text-sm opacity-75 mb-4">
            If anything is missing or incorrect, update it from the{" "}
            <Link href="/account/edit" className="underline-link">
              profile edit page
            </Link>. To change your email, please contact{" "}
            <a href="mailto:{{CONTACT_EMAIL}}" className="underline-link">
              {{CONTACT_EMAIL}}
            </a>.
          </p>
        </section>

        {/* Account deletion — Art. 17 — DANGER ZONE */}
        <section className="border-t pt-8 danger-zone">
          <h2 className="text-lg font-medium mb-2 text-rose-300">Delete my account</h2>
          <p className="text-sm opacity-75 mb-4">
            Permanently delete your account and all associated data
            (Art. 17 right to erasure). This action is{" "}
            <strong>irreversible</strong>. After deletion:
          </p>
          <ul className="text-sm opacity-75 mb-4 space-y-1 list-disc list-inside">
            <li>You will no longer be able to log in</li>
            <li>Your posts, votes and comments will be permanently removed</li>
            <li>Messages you have sent will be removed from other users' inboxes</li>
            <li>You may register again with the same email after 30 days</li>
          </ul>

          {!deleteConfirm ? (
            <button
              type="button"
              onClick={() => setDeleteConfirm(true)}
              className="rights-btn rights-btn-danger"
            >
              I want to delete my account
            </button>
          ) : (
            <div className="delete-confirm-box space-y-3">
              <p className="text-sm">
                To confirm, type{" "}
                <strong className="text-rose-300">DELETE</strong> below:
              </p>
              <input
                type="text"
                value={deleteText}
                onChange={(e) => setDeleteText(e.target.value)}
                placeholder="DELETE"
                disabled={deleting}
                className="delete-input"
              />
              <div className="flex gap-3">
                <button
                  type="button"
                  onClick={() => {
                    setDeleteConfirm(false);
                    setDeleteText("");
                    setError(null);
                  }}
                  disabled={deleting}
                  className="rights-btn rights-btn-muted"
                >
                  Cancel
                </button>
                <button
                  type="button"
                  onClick={deleteAccount}
                  disabled={deleting || deleteText !== "DELETE"}
                  className="rights-btn rights-btn-danger"
                >
                  {deleting ? "Deleting…" : "Permanently delete my account"}
                </button>
              </div>
            </div>
          )}

          {error && (
            <p className="text-rose-300 text-sm mt-3 italic">{error}</p>
          )}
        </section>

        {/* Other GDPR rights */}
        <section className="border-t pt-8">
          <h2 className="text-lg font-medium mb-2">Other requests</h2>
          <p className="text-sm opacity-75">
            For GDPR rights not covered by the buttons above (Art. 18
            restriction of processing, Art. 21 right to object, Art. 22
            automated decision-making, withdrawal of consent), please contact{" "}
            <a href="mailto:{{CONTACT_EMAIL}}" className="underline-link">
              {{CONTACT_EMAIL}}
            </a>. We will respond within one month (Art. 12(3)).
          </p>
        </section>
      </div>
    </main>
  );
}
```

### CSS helpers

```css
.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.6rem 0;
  border-bottom: 1px solid rgba(180, 180, 180, 0.08);
}

.info-label {
  font-size: 0.7rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  opacity: 0.6;
}

.rights-btn {
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

.rights-btn:hover {
  background: rgba(255, 255, 255, 0.05);
}

.rights-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.rights-btn-danger {
  color: rgb(252, 165, 165);
  border-color: rgb(220, 38, 38);
}

.rights-btn-danger:hover {
  background: rgba(220, 38, 38, 0.08);
}

.rights-btn-muted {
  opacity: 0.6;
}

.danger-zone {
  border-color: rgba(220, 38, 38, 0.25);
}

.delete-confirm-box {
  padding: 1rem;
  border: 1px solid rgba(220, 38, 38, 0.3);
  border-radius: 0.5rem;
  background: rgba(220, 38, 38, 0.05);
}

.delete-input {
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

.delete-input:focus {
  outline: none;
  border-color: rgb(220, 38, 38);
}

.underline-link {
  text-decoration: underline;
  text-underline-offset: 2px;
}
```

---

## 2. Data export API — Supabase

`app/api/me/export/route.ts`:

```ts
import { NextRequest } from "next/server";
import { createClient } from "@/lib/supabase/server";

export const runtime = "nodejs";
export const dynamic = "force-dynamic";

export async function GET(_req: NextRequest) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    return new Response(
      JSON.stringify({ error: "Not authenticated" }),
      { status: 401, headers: { "Content-Type": "application/json" } }
    );
  }

  // Collect all user-associated data — replace table list with auto-detected {{USER_TABLES}}
  const [
    profileRes,
    contentRes,
    votesRes,
    savesRes,
    followsRes,
    messagesRes,
  ] = await Promise.all([
    supabase.from("profiles").select("*").eq("id", user.id).single(),
    supabase.from("posts").select("*").eq("user_id", user.id),
    supabase.from("votes").select("*").eq("user_id", user.id),
    supabase.from("saves").select("*").eq("user_id", user.id),
    supabase.from("follows").select("*").or(`follower_id.eq.${user.id},following_id.eq.${user.id}`),
    supabase.from("messages").select("*").or(`sender_id.eq.${user.id},receiver_id.eq.${user.id}`),
  ]);

  const exportData = {
    meta: {
      project: "{{PROJECT_NAME}}",
      export_date: new Date().toISOString(),
      user_id: user.id,
      email: user.email,
      note: "This file is your personal data export under GDPR Art. 15 (right of access) and Art. 20 (right to data portability).",
      format_version: "1.0",
    },
    profile: profileRes.data,
    posts: contentRes.data ?? [],
    votes: votesRes.data ?? [],
    saves: savesRes.data ?? [],
    follows: followsRes.data ?? [],
    messages: messagesRes.data ?? [],
  };

  const filename = `{{PROJECT_NAME}}-export-${user.id.slice(0, 8)}-${new Date().toISOString().split("T")[0]}.json`;

  return new Response(JSON.stringify(exportData, null, 2), {
    status: 200,
    headers: {
      "Content-Type": "application/json",
      "Content-Disposition": `attachment; filename="${filename}"`,
    },
  });
}
```

---

## 3. Account deletion API — Supabase

`app/api/me/route.ts`:

```ts
import { NextRequest } from "next/server";
import { createClient as createServerClient } from "@/lib/supabase/server";
import { createClient as createAdminClient } from "@supabase/supabase-js";

export const runtime = "nodejs";
export const dynamic = "force-dynamic";

export async function DELETE(_req: NextRequest) {
  const supabase = await createServerClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    return new Response(
      JSON.stringify({ error: "Not authenticated" }),
      { status: 401, headers: { "Content-Type": "application/json" } }
    );
  }

  const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL;
  const serviceRoleKey = process.env.SUPABASE_SERVICE_ROLE_KEY;

  if (!supabaseUrl || !serviceRoleKey) {
    return new Response(
      JSON.stringify({
        error:
          "Account deletion service is being configured. Please contact the site operator via in-app messaging for manual deletion under GDPR Art. 17.",
        configured: false,
      }),
      { status: 503, headers: { "Content-Type": "application/json" } }
    );
  }

  const admin = createAdminClient(supabaseUrl, serviceRoleKey, {
    auth: { autoRefreshToken: false, persistSession: false },
  });

  const userId = user.id;

  try {
    // Full deletion strategy: all UGC + PII removed
    // FK ON DELETE CASCADE may handle some automatically when profile is deleted,
    // but we delete explicitly to be safe.

    await Promise.all([
      admin.from("messages").delete().or(`sender_id.eq.${userId},receiver_id.eq.${userId}`),
      admin.from("follows").delete().or(`follower_id.eq.${userId},following_id.eq.${userId}`),
      admin.from("saves").delete().eq("user_id", userId),
      admin.from("votes").delete().eq("user_id", userId),
      admin.from("posts").delete().eq("user_id", userId),
      // Add additional tables from {{USER_TABLES}} here
    ]);

    // Storage: delete uploaded files (e.g. avatars)
    try {
      const { data: files } = await admin.storage.from("avatars").list(userId);
      if (files && files.length > 0) {
        await admin.storage
          .from("avatars")
          .remove(files.map((f) => `${userId}/${f.name}`));
      }
    } catch {
      // Storage may not exist; ignore
    }

    // Delete profile (FK trigger may cascade)
    await admin.from("profiles").delete().eq("id", userId);

    // Delete auth user
    const { error: authError } = await admin.auth.admin.deleteUser(userId);
    if (authError) {
      return new Response(
        JSON.stringify({ error: `Auth deletion failed: ${authError.message}` }),
        { status: 500, headers: { "Content-Type": "application/json" } }
      );
    }

    return new Response(
      JSON.stringify({ success: true, message: "Account deleted under GDPR Art. 17" }),
      { status: 200, headers: { "Content-Type": "application/json" } }
    );
  } catch (err) {
    return new Response(
      JSON.stringify({
        error: err instanceof Error ? err.message : "Deletion failed",
      }),
      { status: 500, headers: { "Content-Type": "application/json" } }
    );
  }
}
```

---

## 4. UGC deletion strategy decision

Before generating the deletion endpoint, ask the user:

**Option A — Full deletion** (everything goes, simple but destructive)
**Option B — Anonymisation** — UGC stays, attributed to "Deleted user" (preserves community value)
**Option C — Soft delete** — 30-day reversible window, then hard delete (most user-friendly, requires cron)

For GDPR specifically:
- Art. 17(3) allows controllers to keep data where necessary for "exercise or defence of legal claims" or for "freedom of expression and information" — but this is a high bar
- Anonymisation must be **irreversible** to fall outside the GDPR (Recital 26); pseudonymisation is not enough
- Soft delete must inform the user clearly in the deletion confirmation

Adapt the API code above based on the chosen strategy.

---

## 5. Testing checklist (must show user after generation)

- [ ] Export endpoint returns 200 and a valid JSON file
- [ ] Delete flow disables the "Permanently delete" button until "DELETE" is typed
- [ ] Service role key is NOT exposed to client: `grep -r "SUPABASE_SERVICE_ROLE" app/components/ src/components/` must return nothing
- [ ] After account deletion, attempting to log in with the same email returns the expected error
- [ ] UGC handling matches the chosen strategy (full/anonymise/soft)
- [ ] `/account` page is linked from the Privacy Policy section 8 (Your Rights)
- [ ] Footer includes a link to `/account` (when logged in) or to the contact email

---

## 6. Update the Privacy Policy

After running this template, update `app/privacy/page.tsx`:

In **section 8** (Your Rights), replace the generic email-only contact with:

```html
            <p>
              You can exercise most of these rights directly from your{" "}
              <Link href="/account">account page</Link>: download your data
              (Art. 15 + 20), correct your profile (Art. 16), or delete your
              account (Art. 17).
            </p>
            <p>
              For rights not available through self-service (Art. 18, 21, 22,
              or withdrawal of consent), contact{" "}
              <a href="mailto:{{CONTACT_EMAIL}}">{{CONTACT_EMAIL}}</a>.
            </p>
```

---

*This template is not legal advice. Service role key handling requires careful review. Before going live, review by a qualified GDPR/data protection lawyer is recommended.*
