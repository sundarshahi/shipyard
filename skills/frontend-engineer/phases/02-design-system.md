# Phase 2: Functional Design Foundation

## Objective

Establish the **minimum viable design system** so components can be built and wired. This is NOT the final design — this is defaults that work. The real design research and polish happens in Phase 5 after everything is functional.

> **If a UX design-system spec exists at `docs/design/`** (produced by the ux-designer agent in DEFINE), it is the **source of truth** — implement its design tokens, type scale, WCAG-AA color, component specs, states, and motion directly instead of inventing defaults. Phase 5 then becomes spec conformance + polish rather than from-scratch design research. If no spec exists (frontend-only build, or UX was skipped), generate the sensible defaults below.

Do NOT spend time on color theory, trend research, or visual polish here. Use sensible defaults (or the UX spec if present). Get to working components fast.

## 2.1 Design Tokens (Defaults)

Create `frontend/app/styles/tokens/`:

```
tokens/
├── colors.ts          # Neutral palette + one primary color (blue default)
├── typography.ts      # System font stack, modular scale
├── spacing.ts         # 4px base unit scale
├── breakpoints.ts     # Standard responsive breakpoints
├── shadows.ts         # 3-level elevation (sm, md, lg)
├── radii.ts           # 3-level border radius (sm, md, lg)
├── z-index.ts         # Z-index scale
├── motion.ts          # Fast/normal/slow durations
└── index.ts           # Unified export
```

Token standards (functional defaults — will be refined in Phase 5):
- **Colors** — Neutral gray scale (50-950). One primary color (blue). Semantic: `success` (green), `warning` (amber), `danger` (red). WCAG AA contrast ratios.
- **Typography** — System font stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI', ...`). Modular scale (1.25). Heading levels h1-h6. Line height: 1.5 body, 1.2 headings.
- **Spacing** — 4px base: `0, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24, 32, 40, 48, 64`.
- **Breakpoints** — `sm: 640px`, `md: 768px`, `lg: 1024px`, `xl: 1280px`, `2xl: 1536px`.
- **Motion** — `fast: 150ms`, `normal: 300ms`, `slow: 500ms`. Respect `prefers-reduced-motion`.

## 2.2 Theme Configuration (Minimal)

Create `frontend/app/styles/theme/`:

```
theme/
├── theme-provider.tsx     # React context for theme switching
├── light-theme.ts         # Light mode (default neutral palette)
├── dark-theme.ts          # Dark mode (inverted neutral palette)
├── theme.css              # CSS custom properties from tokens
└── global.css             # Reset, base styles, font loading
```

Requirements:
- Light and dark mode with system preference detection
- Theme toggle with localStorage persistence
- CSS custom properties bridge tokens to components
- No FOUC on theme load

## 2.3 Tailwind Configuration (if Tailwind selected)

Create `frontend/tailwind.config.ts`:
- Extend with default design tokens
- Standard color palette
- Typography plugin
- Animation utilities
- Container queries

**Keep it simple. These tokens will be upgraded in Phase 5 (Design & Polish) with researched colors, typography, and visual identity.**

## 2.4 Security Foundation (EMIT — secure-by-default per `security-defaults.md`)

Security defaults are written into the first draft, not bolted on after the audit. Establish them now, at the foundation, so every component and page inherits them.

**CSP + security headers.** EMIT framework-level security headers via `next.config.*` `headers()` (or middleware / framework equivalent):
- `Content-Security-Policy` that blocks inline script — **no `unsafe-inline`, no `unsafe-eval`** (use nonces/hashes for any required inline). Defense-in-depth against XSS.
- **Trusted Types** — add CSP `require-trusted-types-for 'script'` plus a named `trusted-types` policy where the framework supports it, to neutralize DOM-XSS sinks at the source (cross-ref `security-defaults.md` → "Context-aware output encoding & safe templating").
- `Strict-Transport-Security` set to **`max-age=15552000; includeSubDomains; preload`** (≥180 days), `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` (or CSP `frame-ancestors`), `Referrer-Policy: strict-origin-when-cross-origin`, a restrictive `Permissions-Policy`.
- **Cross-origin isolation** — `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Resource-Policy: same-origin` on all responses; add `Cross-Origin-Embedder-Policy: require-corp` where cross-origin isolation is actually needed (e.g. `SharedArrayBuffer`).
- **Cache-Control: `no-store`** on every response carrying sensitive data or an authenticated page (add `Pragma: no-cache` for legacy proxies) so credentials/PII are never cached by the browser or intermediaries.

**Subresource Integrity (SRI).** Prefer **self-hosting third-party scripts**. Every remaining external/CDN `<script>` and `<link rel="stylesheet">` MUST carry an `integrity` hash **and** a `crossorigin` attribute so a tampered CDN asset fails to load rather than executing.

**Cookie prefixes.** Name auth/session cookies with the **`__Host-`** prefix (or `__Secure-` when a parent-domain scope is required) IN ADDITION to `HttpOnly` + `Secure` + `SameSite` — the prefix is enforced by the browser and proves Secure/host-only scoping (cross-ref `security-defaults.md` → "Session & self-contained-token lifecycle").

**Sanitization wrapper.** EMIT `frontend/app/lib/sanitize.ts` — a DOMPurify wrapper with a strict allowlist. **No raw `dangerouslySetInnerHTML`** anywhere: any HTML injection must pass through this sanitizer first. Add an ESLint rule (`react/no-danger`) so unsanitized usage fails CI; render untrusted strings as text by default.

**Other secure defaults (carry forward):**
- No secrets in the client bundle — only `NEXT_PUBLIC_*` config values are exposed; credentials/tokens stay server-side (cross-ref `security-defaults.md`).
- Auth/session cookies are `HttpOnly` + `Secure` + `SameSite=Lax|Strict` and carry a `__Host-`/`__Secure-` prefix; no auth tokens or PII in `localStorage` (enforced in Phase 4 auth).

## 2.5 Localization (i18n) Foundation (EMIT — scaled to the Phase 1 i18n decision)

Establish i18n at the foundation so no component is ever born with a hardcoded string. Scale to the Phase 1 `i18n-decision.md`: **multi-locale →** full provider + locale routing (Phase 4); **single-locale →** still externalize strings + `Intl` formatting so a second locale is a config change, not a rewrite.

- **Library by framework:** `next-intl` (or `react-i18next`) for Next.js; `react-i18next` for Vite/Remix SPAs; Astro i18n / Nuxt i18n / Paraglide for the others. Wire the provider at the root layout.
- **Message catalogs:** `frontend/app/messages/<locale>.json` — the **default-locale catalog is the canonical key set**. EMIT a `t()`/`useTranslations` accessor. **No hardcoded user-facing strings in components** — every display string, `aria-label`, placeholder, and error message resolves from a key. (Pseudo-locale tested in Phase 6.)
- **Locale formatting:** numbers, dates, currency, and relative times go through `Intl.NumberFormat`/`DateTimeFormat`/`RelativeTimeFormat` keyed to the active locale — never hand-format.
- **Direction (RTL):** set `<html dir>` from the active locale (`rtl` for `ar`/`he`/`fa`). Author layouts with **CSS logical properties** (`margin-inline`, `padding-inline-start`, `inset-inline`, `text-align: start`) so RTL works without per-component overrides.

## 2.6 Font Loading & Web-Performance Foundation (EMIT)

Performance is built in, not tuned at the end — these defaults feed the Phase 6 Core Web Vitals budget.

- **Optimized font loading:** use the framework optimizer (`next/font` self-hosts, sets `font-display: swap`, preloads, and `size-adjust` to cut CLS) or self-hosted `@font-face` with `font-display: swap` + `<link rel="preload">` for the one critical font. Subset fonts; limit families/weights. Never block first paint on a webfont.
- **Pairs with:** the `Image` primitive (Phase 3) and route-level code-splitting + waterfall-free data loading (Phase 4) to hit the budget gated in Phase 6.

## Validation Loop

Before moving to Phase 3:
- All tokens defined and exported
- Light/dark themes render
- Theme toggle works
- Tailwind config extends with tokens
- i18n provider wired; message catalog present; **no hardcoded user-facing strings**; `Intl` used for number/date/currency; `dir`/logical properties handle RTL (scaled to the Phase 1 i18n decision)
- Fonts loaded via the optimizer with `font-display: swap` + preload + subsetting
- CSP + security headers emitted (no `unsafe-inline`/`unsafe-eval`; Trusted Types `require-trusted-types-for 'script'` where supported); COOP/CORP `same-origin` set; HSTS `max-age>=15552000; includeSubDomains; preload`; `Cache-Control: no-store` on authenticated/sensitive responses
- SRI `integrity` + `crossorigin` on every external/CDN `<script>`/`<link>` (or third-party scripts self-hosted); auth/session cookies carry a `__Host-`/`__Secure-` prefix
- `lib/sanitize.ts` (DOMPurify) present; `react/no-danger` ESLint rule on

**Do NOT present design system for approval here — it's defaults. Move to components.**

## Quality Bar

- Every color meets WCAG 2.1 AA contrast
- Typography scale is consistent
- Spacing scale covers layout needs
- No hardcoded visual values
- **No hardcoded user-facing strings** — all display text resolves from the i18n message catalog; numbers/dates/currency via `Intl`; RTL handled with logical properties
- Fonts loaded via the optimizer (`font-display: swap`, preload, subset) — no render-blocking webfont
- This is a FUNCTIONAL foundation, not the final design
- **`security-defaults checklist passes`** — CSP/security headers served (no `unsafe-inline`/`unsafe-eval`; Trusted Types directive where supported), COOP/CORP `same-origin` + HSTS `max-age>=15552000; includeSubDomains; preload` + `Cache-Control: no-store` on sensitive responses, SRI `integrity`+`crossorigin` on external scripts (or self-hosted), `__Host-`/`__Secure-` prefixed secure cookies, DOMPurify sanitizer in place with no unsanitized `dangerouslySetInnerHTML`, no secrets in the client bundle
