# Watcher — Design Spec (DESIGN-SPEC.md)

**Status:** Source of truth for the Watcher control-page UI. Every screen is built against this document; agents
read it before writing frontend code rather than inventing styles per screen.
**Derived from:** `WhatsApp_Bot_MVP_1.2_final.pdf` §4, §6, §11 (the stated aesthetic, fonts, and four views).
**Origin note:** This is an original design system created for Watcher. Nothing is copied from the Automera
marketing website — different product, different surface, different audience (an operator triaging work, not a
visitor being marketed to).

---

## 1. Design principles

v1.2 commits to a "Stripe-meets-Linear" aesthetic. Translated into rules:

1. **The Inbox is the product.** It must feel *fast* and *one-click*. Every other view is a settings surface that
   should fade into the background. Optimize for the operator clearing a queue in under 30 seconds per item
   (v1.2 §8: "can a user, in under thirty seconds, push a message as a structured record").
2. **Calm by default, loud only for uncertainty.** A correctly-classified message should read as quiet and
   confident. Visual weight (color, motion, prompts) is spent *only* where the bot is unsure — the
   medium/low confidence bands. This is the "Gmail spam filter" mental model (v1.2 §3) made visual.
3. **Density without clutter.** Linear-style information density: show the classification, extracted fields, and the
   primary action without scrolling, but with generous whitespace and a clear type hierarchy.
4. **Mobile-first triage.** Founders triage from their phone between meetings (v1.2 §9, §11). The Inbox is
   designed for a single-column mobile layout first, then expanded to desktop.
5. **Honest about confidence.** The UI never hides uncertainty. The confidence band is always visible, and
   low-confidence items explicitly say "bot unsure, needs you" (v1.2 §3 rubric).
6. **Bilingual data, English chrome (v1).** UI labels are English; message *content* may be Arabic, English, or
   mixed and must render correctly, including RTL runs (see §9). Full RTL UI mirroring is v2.

---

## 2. Color tokens

A neutral, near-monochrome base (Linear/Stripe restraint) with a single brand accent and a dedicated,
semantically-fixed confidence-band scale. Defined as CSS custom properties / Tailwind theme extensions.
Light is the primary theme; dark is supported and uses the same token names.

### Base / neutral (slate-cool)
| Token | Light | Dark | Use |
|-------|-------|------|-----|
| `--bg` | `#FBFBFD` | `#0B0C0E` | App background |
| `--surface` | `#FFFFFF` | `#16181C` | Cards, panels, inbox rows |
| `--surface-2` | `#F4F5F7` | `#1E2127` | Subtle fills, hover, code blocks |
| `--border` | `#E6E8EC` | `#2A2E36` | Hairline dividers, card borders |
| `--text` | `#0E1116` | `#F2F4F8` | Primary text |
| `--text-muted` | `#5B616E` | `#9BA3B0` | Secondary text, metadata |
| `--text-faint` | `#8A909C` | `#6B7280` | Timestamps, placeholders |

### Brand accent (indigo)
| Token | Value | Use |
|-------|-------|-----|
| `--accent` | `#4F46E5` | Primary buttons, active nav, focus rings, links |
| `--accent-hover` | `#4338CA` | Hover state |
| `--accent-subtle` | `#EEF0FF` (light) / `#1E1B3A` (dark) | Selected row tint, badges |

### Confidence bands (semantic — never reuse for anything else)
These map 1:1 to the v1.2 §3 routing rubric and are the most important colors in the product.
| Band | Token | Value | Meaning |
|------|-------|-------|---------|
| High (`≥0.85`) | `--band-high` | `#16A34A` (green) | Auto-route / one-click confirm |
| Medium (`0.5–0.85`) | `--band-medium` | `#D97706` (amber) | Needs a quick sign-off |
| Low (`<0.5`) | `--band-low` | `#DC2626` (red) | "Bot unsure, needs you" |
| Auto-routed tag | `--tag-auto` | `--text-muted` on `--surface-2` | Audit-trail marker for rule-routed items |

### Functional
| Token | Value | Use |
|-------|-------|-----|
| `--success` | `#16A34A` | Successful route confirmation |
| `--danger` | `#DC2626` | Destructive actions, delivery failures |
| `--warning` | `#D97706` | Soft-cap warnings, dead-letter |
| `--focus-ring` | `--accent` @ 40% | 2px focus ring, always visible for a11y |

**Contrast:** all text/background pairs meet WCAG AA (≥4.5:1 body, ≥3:1 large). Band colors are paired with
an icon + label, never color alone (a11y + colorblind safety).

---

## 3. Typography

Per v1.2 §11: **Inter** (English UI + Latin data), **Cairo** (Arabic data). Both via `next/font` self-hosted (no
runtime CDN — matters for the self-hosted regulated tier where external calls are restricted).

- **Font stacks:** `--font-sans: Inter, system-ui, sans-serif;` `--font-arabic: Cairo, Inter, sans-serif;` Arabic
  runs detected at render and assigned `--font-arabic` (see §9). Monospace `--font-mono: "JetBrains Mono",
  ui-monospace` for phone numbers, IDs, JSON, and confidence values.
- **Type scale** (rem, 1rem = 16px):

| Token | Size / line-height | Weight | Use |
|-------|--------------------|--------|-----|
| `display` | 1.75 / 2.25 | 600 | Page titles |
| `h1` | 1.375 / 1.875 | 600 | View headers |
| `h2` | 1.125 / 1.625 | 600 | Section / card titles |
| `body` | 0.9375 / 1.5 | 400 | Default body, message content |
| `body-strong` | 0.9375 / 1.5 | 550 | Emphasis, sender names |
| `small` | 0.8125 / 1.25 | 400 | Metadata, field labels |
| `mono` | 0.8125 / 1.25 | 450 | Phones, IDs, confidence |

- **No font weights below 400; headings cap at 600** (Linear restraint — avoid heavy bold).

---

## 4. Spacing, radius, elevation

- **Spacing scale (4px base):** `4 · 8 · 12 · 16 · 20 · 24 · 32 · 40 · 48 · 64`. Use tokens `space-1…space-16`.
  Default card padding `space-4` (16px) mobile, `space-6` (24px) desktop.
- **Radius:** `--radius-sm: 6px` (badges, inputs), `--radius-md: 10px` (cards, buttons), `--radius-lg: 16px`
  (modals, sheets), `--radius-full` (avatars, pills).
- **Elevation (subtle, Stripe-like):**
  - `shadow-1`: `0 1px 2px rgba(16,17,22,.06)` — cards at rest.
  - `shadow-2`: `0 4px 16px rgba(16,17,22,.08)` — popovers, dropdowns.
  - `shadow-3`: `0 12px 40px rgba(16,17,22,.16)` — modals, mobile action sheets.
  - Borders do most of the separation work; shadows are a light second layer, never heavy.
- **Layout:** max content width `1200px`; Inbox uses a two-pane layout ≥1024px (list + detail), single column
  below. Sidebar nav `240px` desktop, collapses to a bottom tab bar on mobile.

---

## 5. Iconography & motion

- **Icons:** Lucide (v1.2 §11), `1.5px` stroke, sized `16`/`20`/`24`. Consistent metaphors: `inbox`, `radio`
  (Sources/watching), `plug`/`webhook` (Destinations), `git-branch` (Rules), `gauge` (Eval/Admin), `check`,
  `pencil` (edit/override), `skip-forward` (skip), `merge`, `link`, `user-plus`.
- **Motion:** fast and minimal. Transitions `120–160ms ease-out`. Row selection, button press, and toast
  entrance only. Respect `prefers-reduced-motion`. No decorative animation — this is a work tool. (The one
  exception allowed: a brief success pulse on the confidence chip when an item is routed.)

---

## 6. Core components

### Buttons
- **Primary:** `--accent` fill, white text, `--radius-md`, `space-2 space-4` padding. One primary per context.
- **Secondary:** `--surface` fill, `--border`, `--text`. **Ghost:** transparent, used for tertiary/`Skip`.
- **Destructive:** `--danger` text/border; solid fill only on confirm.
- Always show focus ring; min touch target `44px` on mobile.

### Confidence chip (signature component)
A pill showing the band: colored dot + label + overall score in mono. `HIGH · 0.92`. Color from §2 bands.
On hover/expand, reveals the per-field breakdown (intent / person / company) as a small bar set — because
v1.2 §3 decomposes confidence by field and the router uses the *lowest* field score.

### Inbox row / card
The atomic unit. Contains, in priority order:
1. **Confidence chip** (leftmost, color-coded band).
2. **`summary_one_line`** as `body-strong` — the bot's plain-language read of the message.
3. **Extracted fields** as compact labeled chips: person name + `appears_to_be`, company (if any), `phone`
   (mono), `language` flag, `suggested_record_type` (Individual / Contact+Company / Company-only — v1.2 §6).
4. **Intent** badge (`new_lead`, `support_issue`, …).
5. **Primary action** by band (§7).
6. **`auto` tag** if rule-routed (audit trail; v1.2 §4).
- Selected row tinted `--accent-subtle`. Media items (voice/image/PDF) show a modality icon + transcript
  preview with a subtle "transcribed" marker (lower-confidence styling, per build-spec addendum §6).

### Field-edit popover
One-click edit/override of any extracted field inline (v1.2 §4: "confirm, edit, or override in one click").
Editing a field that drove classification re-evaluates the suggested record type live.

### Identity-match card
Surfaces a candidate duplicate (v1.2 §7). Shows the existing record vs. the new message side by side, with
three actions, **Merge as primary** (v1.2's recommended default), then `Keep separate, link related`, then
`This is a new person`. Shows match basis (phone / name+company) and score.

### Destination & recipe components
- **Recipe picker:** card grid of recipes (Google Sheets, HubSpot, Pipedrive, Notion, Airtable, Custom) with
  logo, one-line setup effort, and a "paste your webhook URL" field (v1.2 §5 table).
- **Field-mapping table:** internal field → destination field, edited once at setup.

### Rule builder
Plain-language condition→action rows, no DSL (v1.2 §9): `[sender is new] AND [message contains "quote"] →
[route to HubSpot as new_lead, tag inbound-whatsapp]`. Dropdowns, not free text.

### Toast / status
Bottom-center (mobile), bottom-right (desktop). Route success → green with destination-record link. Webhook
failure → amber with retry + link to dead-letter (build-spec addendum §11).

### Empty states
Every view has an explicit empty state. Inbox-zero is a deliberately rewarding moment ("Queue clear ✓").

---

## 7. The three confidence bands → three interaction patterns

This is the heart of the UI and maps directly to v1.2 §3 + §6:

| Band | Inbox visual | Primary action | Control-chat behavior |
|------|--------------|----------------|------------------------|
| **High** `≥0.85` | Green chip, quiet row | **Confirm & route** (one click) — or auto-routed, shown as `auto` | One-line audit summary only, no prompt |
| **Medium** `0.5–0.85` | Amber chip, slightly raised | **Review:** Confirm / Change / Skip | Quick-reply buttons in control chat |
| **Low** `<0.5` | Red chip, "bot unsure, needs you" label | Manual classify — fields shown as editable, none pre-trusted | No ping; lands in inbox only |

The router keys off the **lowest field-level confidence**, not just overall (v1.2 §3) — the chip's expanded
view must make the weakest field obvious so the operator knows *what* to check.

---

## 8. Views (the four-view spine + admin)

Per v1.2 §4 and §11. Nav order reflects frequency of use:

1. **Inbox** — the triage queue and the product's center of gravity. Two-pane on desktop (list + detail),
   single-column mobile. Must feel instant; optimistic UI on confirm/route. Includes auto-routed items marked
   `auto` for audit.
2. **Sources** — opt-out model (v1.2 §4): a settings screen listing chats on the connected number with a
   mute/exclude toggle. First-run exclusion wizard walks the chat list once. *Not a workflow* — it should feel
   like a settings page you visit rarely.
3. **Destinations** — connected Sheets/webhooks + recipe picker + field mapping (§6).
4. **Rules** — the rule builder list (§6).
5. **Admin / Eval** (founder-only, role-gated) — eval dashboard surfacing current accuracy, per-language
   breakdown, calibration, confusion matrix, and per-tenant LLM/media spend (v1.2 §12 + build-spec
   addendum §16). Visually distinct (utilitarian, data-dense) from the operator views.

---

## 9. Arabic, RTL, and bidi (v1 scope)

- UI chrome stays **LTR English**. Message **content** is rendered with automatic direction detection: each
  rendered message/field gets `dir="auto"`, and runs detected as Arabic use `--font-arabic` (Cairo).
- Inbox cards, summaries, and extracted name/company fields must handle mixed (code-switched) text — the
  Gulf norm (v1.2 §3) — without breaking layout. Use logical CSS properties (`margin-inline`, `padding-inline`)
  so individual RTL content blocks lay out correctly inside the LTR shell.
- Numbers/phones always rendered LTR in mono regardless of surrounding script.
- A `language` indicator (`en` / `ar` / `mixed`) chip from the classifier appears on each item.

---

## 10. Accessibility & responsiveness baseline

- WCAG AA contrast everywhere; band meaning never conveyed by color alone (icon + text label always).
- Full keyboard operability of the Inbox: `j/k` to move, `Enter` to confirm, `e` to edit, `s` to skip
  (Linear-style) — triage speed is a feature.
- Visible focus rings; `aria-live` on the toast region; semantic landmarks.
- Breakpoints: mobile `<640`, tablet `640–1024`, desktop `>1024`. Inbox two-pane only `≥1024`.
- Touch targets `≥44px`; bottom tab bar nav on mobile.

---

## 11. Implementation notes for agents

- **Stack:** Next.js 15 + TypeScript + Tailwind CSS (v1.2 §11). Tokens above become the Tailwind theme
  (`tailwind.config`) + CSS variables for light/dark. Lucide for icons. `next/font` self-hosted for Inter + Cairo.
- **Tokens are law:** never hardcode a hex value in a component — reference the token. New color needs?
  Propose a token here first.
- **Component source of truth:** build the Confidence chip, Inbox row, Identity-match card, and Field-edit
  popover as primitives first; every view composes them.
- **Self-hosted constraint:** no external font/icon/CDN calls at runtime — everything bundled, because the
  regulated tier forbids data egress (v1.2 §10).
- **Do not** import anything from the Automera marketing site. This is a separate product with a separate
  visual language; shared "brand" is intentionally minimal at this stage.
