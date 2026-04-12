# i18n Lib

Internationalization in `lib/i18n/`.

## Files

| File | Purpose |
|------|---------|
| config.ts | i18n config, supported locales |
| messages.ts | Translation messages (en, es, fr) |
| navigation.ts | Navigation labels |

## Usage

- `useI18n()` – hook from `components/i18n/i18n-provider.tsx`
- `t(key)` – translate key (dot path, e.g. `login.signIn`)
- Locale stored in user preferences and cookie; `<html lang>` follows active locale.

## Conventions (sweeps and new UI)

1. **Namespaces** — Group keys by feature (`login`, `requestForm`, `contactUs`, `cookieConsent`, `appMetadata`, …). Avoid dumping product copy into `common` unless it is truly shared.
2. **Locales** — Every new key must exist under **`en`**, **`es`**, and **`fr`** in the same change, with the same nested shape.
3. **Reuse** — Prefer extending an existing subtree (e.g. `login.errors`) before adding parallel keys.
4. **Leaf vs parent** — Either call `useI18n()` in the component that renders the string, or pass `t` from a parent when the child is presentational-only (see AI chat composer pattern).
5. **Forms / Zod** — Build schemas where `t` is in scope (e.g. `useMemo(() => z.object({ email: z.string().email(t("login.errors.invalidEmail")) }), [t])`) so validation messages follow the active locale.
6. **Server components** — No `useI18n`; import `messages` and the resolved locale (e.g. cookie + `normalizeLocale`) like `generateMetadata` in `app/layout.tsx`.
7. **Registry** — Drawer titles in `lib/drawers.ts` are English fallbacks; user-facing nav should use `navigation.drawerLabels.*` in `messages.ts` when the drawer appears in the sidebar/More menu.

See also [DRAWER_PR_CHECKLIST.md](../../DRAWER_PR_CHECKLIST.md) (copy / i18n checkbox).
