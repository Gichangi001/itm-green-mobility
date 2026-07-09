# ITM Green Mobility — CSS / Design System Guide

This is the reference document for polishing UI/UX across the platform. It covers
every design token, component pattern, and animation currently in use, plus where
things live and the conventions to follow when editing.

**Scope:** 10 static HTML pages, no build step, no framework. Each page has its own
`<style>` block — there is no shared CSS file. When you change a token or component,
it has to be changed in every file that uses it (see "File map" below for which
files share which patterns).

---

## 1. File map

| File | Role | Shares CSS framework with |
|---|---|---|
| `index.html` | Marketing homepage | — (has extra sections: hero, mission, reviews, impact, gallery) |
| `apps/rider/index.html` | Client App simulation | driver, station-operator, rental |
| `apps/driver/index.html` | Driver App simulation | rider, station-operator, rental |
| `apps/station-operator/index.html` | Station Operator App simulation | rider, driver, rental |
| `apps/rental/index.html` | Rental App simulation | rider, driver, station-operator |
| `portals/management/index.html` | Management/Admin dashboard | corporate, fleet, charging, payments |
| `portals/corporate/index.html` | Corporate Client dashboard | management, fleet, charging, payments |
| `portals/fleet/index.html` | Fleet Management dashboard | management, corporate, charging, payments |
| `portals/charging/index.html` | Charging Station dashboard | management, corporate, fleet, payments |
| `portals/payments/index.html` | Payment IFS dashboard | management, corporate, fleet, charging |

Two shared component families exist:
- **Apps** (4 files) use the **phone mockup** system (`.phone`, `.p-*` classes) — see §6.
- **Portals** (5 files) use the **dashboard shell** system (`.dash`, `.kpi`, `.tabs`, `.tbl` — see §7.

`assets/hero-benin.jpg` is the only image asset in the repo (used by `index.html`'s hero, both themes).

---

## 2. Fonts

Loaded once via Google Fonts in every page's `<head>`:

```html
<link href="https://fonts.googleapis.com/css2?family=Sora:wght@400;500;600;700;800&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@500;600&display=swap" rel="stylesheet" />
```

| Font | Used for | Rule |
|---|---|---|
| **Sora** | All headings (`h1,h2,h3,h4`), stat numbers, brand name, KPI values | `font-family:'Sora',sans-serif;letter-spacing:-.02em;line-height:1.05` |
| **Inter** | Body text (default) | set on `body` |
| **JetBrains Mono** | Phone status bars, phone OTP/plate values, browser URL bar, `.mono` utility | `.mono{font-family:'JetBrains Mono',monospace}` |

---

## 3. Color tokens (CSS custom properties)

Every page defines the same token set on `:root` (light, default) and overrides it on
`html[data-theme="dark"]` (dark, opt-in). **Never hardcode a hex color in a component
rule if a token exists for it** — that's what breaks theme-switching.

### Light theme (`:root`, default)
```css
--ink:#FFFFFF;            /* page background */
--ink-2:#F4F6F8;           /* raised surface (alt sections, browser chrome) */
--ink-3:#FFFFFF;           /* card surface */
--hunter:#123D2A;          /* fixed brand green — used on self-contained dark chips regardless of theme */
--volt:#34D399;            /* clean green — backgrounds/borders/small swatches, badges with dark text on top */
--volt-text:#047857;       /* deep readable green — for green TEXT/icons on light surfaces */
--emerald:#10B981;         /* signal green, paired with volt in gradients */
--amber:#B45309;           /* charging/warning accent, deepened for light-bg contrast */
--sky:#0284C7;             /* sky blue accent */
--coral:#FF7A59;           /* decorative warmth (gallery/hero glows only) */
--violet:#7C6CF0;          /* decorative dusk accent (gallery/hero glows only) */
--text:#111417;            /* near-black body text */
--muted:#4B5563;           /* secondary text */
--faint:#6B7280;           /* tertiary text / labels */
--line:rgba(15,23,31,.10); /* borders */
--glass:rgba(255,255,255,.78); /* frosted nav background */
--tint-02/03/04:rgba(15,23,31,.035/.05/.065); /* subtle surface washes, ghost button bg, hover tints */
--radius:22px;             /* card corner radius */
--shadow:0 24px 50px -20px rgba(15,23,31,.14);
--maxw:1220px;             /* content max-width (1280px in portals) */
```

### Dark theme (`html[data-theme="dark"]`)
```css
--ink:#071A12; --ink-2:#0A2418; --ink-3:#0E2E20;
--hunter:#123D2A;          /* same as light — it's a fixed brand color, not theme-dependent */
--volt:#B7F24C;            /* electric lime */
--volt-text:#B7F24C;       /* same as --volt in dark (no separate deepening needed) */
--emerald:#2FB170; --amber:#FFB020; --sky:#58C7F2;
--coral:#FF7A59; --violet:#7C6CF0;   /* unchanged from light */
--text:#EAF3EC; --muted:#9DB5A6; --faint:#5F7A6B;
--line:rgba(183,242,76,.14);
--glass:rgba(9,28,19,.55);
--tint-02/03/04:rgba(255,255,255,.02/.03/.04);
--shadow:0 30px 60px -20px rgba(0,0,0,.65);
```

> ⚠️ **Known-fixed bug (2026-07-09):** `index.html`'s dark block once had
> `--tint-02:var(--tint-02)` (self-referencing → CSS-invalid → silently fell back to
> the light value). If a future edit ever reintroduces a var that references itself,
> it will fail silently with no console error — always grep for `--tint-0X:var(--tint-0X)`
> after bulk find/replace passes on this file.

### `index.html`-only extra tokens
Used for the homepage's headline gradient text and alternating section bands:
```css
--grad-hero-h1, --grad-stat, --grad-plat-h1   /* gradient text for h1 .g span, stat numbers, platforms-page h1 */
--stats-bg, --industries-bg                    /* alternating-band section backgrounds */
```

### Color usage rules of thumb
- **`--volt`** = bright/saturated, used as a *background* (buttons, badges, small dots) where dark text (`#052012`) sits on top. Stays visually similar across themes.
- **`--volt-text`** = the *readable-as-text* variant. Use this (never `--volt`) when coloring text or an icon that sits directly on the page background.
- **`--hunter`** (`#123D2A`) is intentionally fixed in both themes — it's used for small self-contained dark chips (brand mark, phone status bar accents) that should look the same regardless of site theme.
- Per-item accent colors (`--pc` on `.pillar`, `--ac` on `.app-card`) are inline hex (`style="--pc:#58C7F2"`) and deliberately **not** tokenized — they're small decorative swatches/badges (dark text on bright bg), same reasoning as `--volt`.

---

## 4. The "permanently dark" pattern

Three sections on `index.html` are designed to always look dark/cinematic **regardless of
site theme** — `.hero`, `.impact`, `.cta-inner`. Each pins its own local token values and
re-declares `color` so unstyled children (e.g. a plain `<h1>` with no `color` rule) inherit
the pinned value instead of the page's theme color:

```css
.impact{
  background:linear-gradient(160deg,#0b2c1d,#071c12);
  --text:#EAF3EC; --muted:#9DB5A6; --faint:#5F7A6B; --volt-text:#B7F24C;
  --tint-02:rgba(255,255,255,.02); --line:rgba(183,242,76,.14);
  color:var(--text);   /* ← required. Without this, descendants inherit body's color, not this pin. */
}
```

**If you add a new "always dark" band, you must include `color:var(--text)` on the
band's own selector** — custom-property overrides alone do *not* propagate to a
plain `color` declared higher up the tree (this was a real bug, fixed 2026-07-06).

The hero's background image (`assets/hero-benin.jpg`) is shared by both themes; only the
tint overlay differs:
```css
.hero::before{background:linear-gradient(180deg,rgba(255,255,255,.12),rgba(255,255,255,0) 30%,rgba(15,23,31,.12)), url('assets/hero-benin.jpg') 82% 58%/cover no-repeat}
html[data-theme="dark"] .hero::before{background:linear-gradient(180deg,rgba(7,20,32,.68),rgba(7,26,18,.76) 40%,rgba(7,26,18,.92) 85%), url('assets/hero-benin.jpg') 82% 58%/cover no-repeat}
```

---

## 5. Shared components (all 10 pages)

### Buttons
```css
.btn{display:inline-flex;align-items:center;gap:9px;justify-content:center;font-weight:600;font-size:15px;padding:13px 22px;border-radius:999px;cursor:pointer;border:1px solid transparent;transition:transform .25s,box-shadow .25s,background .25s,border-color .25s}
.btn:active{transform:translateY(1px)}
.btn:hover{animation:btnShake .4s ease}          /* one-time wiggle, see keyframes below */
.btn-primary{background:linear-gradient(120deg,var(--volt),var(--emerald));color:#052012;box-shadow:0 12px 30px -8px rgba(183,242,76,.5)}
.btn-primary:hover{transform:translateY(-2px);box-shadow:0 18px 40px -8px rgba(183,242,76,.65),0 0 26px 6px rgba(183,242,76,.4)}  /* glow ring */
.btn-ghost{background:var(--tint-04);border-color:var(--line);color:var(--text)}
.btn-ghost:hover{border-color:var(--volt);background:rgba(183,242,76,.07);transform:translateY(-2px);box-shadow:0 0 22px 3px rgba(183,242,76,.22)}
```
`@keyframes btnShake` — small rotate wiggle (±3°) while holding the hover `translateY(-2px)`;
respects `prefers-reduced-motion` (animation disabled). This is the "make it feel alive"
micro-interaction requested 2026-07-06 — apply it to any new primary/ghost button variant.

### Nav
- `position:sticky`, transparent until `.nav.scrolled` (JS adds this class after 30px scroll) applies `background:var(--glass);backdrop-filter:blur(18px)`.
- Theme toggle button (`#themeToggle` / `#themeToggleMobile`) shows a moon icon by default (light) and a sun icon in dark mode via `html[data-theme="dark"] .theme-toggle .i-sun{display:block}` / `.i-moon{display:none}`.
- On phones ≤430px, `.nav-cta .btn-primary` (the "Book a ride" button) hides — it's already reachable from the mobile menu. Below 640px the brand subtitle also hides. This was a deliberate crowding fix; don't remove without re-checking narrow viewports.

### Reveal-on-scroll
```css
.reveal{opacity:0;transform:translateY(28px);transition:opacity .7s cubic-bezier(.2,.8,.2,1),transform .7s cubic-bezier(.2,.8,.2,1)}
.reveal.in{opacity:1;transform:none}
```
Driven by an `IntersectionObserver` (threshold `.12`) that adds `.in` and unobserves —
one-shot, not re-triggered on scroll-up. Any new section content should get `class="... reveal"`
to match the site's fade-in rhythm.

### Theme toggle mechanism (identical script on all 10 pages)
1. A pre-paint inline script in `<head>` reads `localStorage.getItem('itm-theme')` and sets
   `data-theme="dark"` on `<html>` **before first paint** if saved — avoids a flash of wrong theme.
   Default (no saved value) = light.
2. A second script near `</body>` wires the toggle button: flips the attribute, saves the
   choice, and syncs `<meta name="theme-color">` for the browser chrome color.
3. **When copying this pattern to a new page, both scripts must be present** — the
   pre-paint one prevents flash-of-wrong-theme; the click-handler one makes the button work.

---

## 6. Phone mockup system (apps only)

Realistic iPhone frame, anchored to the **left** column of a 3-column grid
(`340px 230px 1fr` — phone | step rail | value card):

```css
.phone-frame{position:relative;display:inline-block}     /* un-clipped wrapper for the side buttons */
.phone{width:300px;height:620px;border-radius:52px;background:linear-gradient(160deg,#3a3a3c,#1c1c1e);border:6px solid #0a0a0b;overflow:hidden}
.phone::before{/* dynamic island */ width:100px;height:30px;background:#000;border-radius:18px}
.phone-btn{position:absolute;background:#0a0a0b}          /* mute switch / volume rocker / power button, positioned via .btn-mute/.btn-vol-up/.btn-vol-down/.btn-power */
.phone-screen{background:linear-gradient(180deg,#fbfdf9,#eef4e8)}   /* always light — it's a phone UI, independent of site theme */
```

**The phone screen's internal UI (`.p-*` classes) always renders light**, regardless of
site theme — these are hardcoded neutral greys (`#111417` text, `rgba(15,23,31,...)` borders),
*not* theme tokens. This is intentional (a phone app UI convention), but it means if you
fix a color token bug in the site chrome, you do **not** need to touch `.p-*` rules.

Key `.p-*` building blocks (all scoped inside `.phone-screen > .p-body`):
| Class | Use |
|---|---|
| `.p-h` / `.p-sub` | Screen title / subtitle |
| `.p-field` | Read-only label+value row (e.g. "Full name: Aïcha Dossou") |
| `.p-input-group` | Real `<input>`/`<select>` — used for the one actually-fillable-looking form (Driver App's apply step) |
| `.p-chip` / `.p-chip-row` | Pill tags; `.on`/`.ok`/`.warn`/`.info`/`.neutral` color variants |
| `.p-card` / `.p-row` | List rows inside a card |
| `.p-driver` | Avatar + name + meta row (driver/rider match cards) |
| `.p-map` | Mini map mock — `.roads` (grid texture), `.route`, `.pin.a`/`.pin.b`, `.pin-label`, `.car` (moving icon, `@keyframes carmove`), `.eta-chip`/`.earn-chip`, `.caption` |
| `.p-stat-row` / `.p-stat` | Two-up stat callouts |
| `.p-vehicle` / `.p-loc-chip` | Rider App's vehicle-type and destination pickers (selectable, `.sel`/`.on` state) |
| `.p-hero` | Rider App's "Green Hero" CO₂-saved callout |
| `.p-btn` | The in-phone primary action button — **has the glow-pulse `@keyframes glow` animation** so it's always the obvious next tap |
| `.p-spin` / `.p-searching` | Loading spinner state |
| `.p-stars` | Tap-to-rate stars (rider app) |

### The auto-resolving spinner pattern (important — was a real bug)
Steps with `searching:true` or `processing:true` show a spinner-only screen with **no
button** at all. The reveal is **not** wired to a click — `render()` itself starts a
`setTimeout` that swaps in `step.phoneAfter()` after 1400ms, guarded by a stale-check
(`if(idx !== stepAtStart) return;`) in case the user hits "Restart" mid-wait. The outer
"continue" button (`#ctaBtn`) is `disabled` for the duration via `ctaBtn.disabled = !!(step.searching || step.processing)`.

**If you add a new spinner step, follow this exact pattern** — the previous version
wired the reveal to a phone-button click that didn't exist yet at spinner time, which
permanently stuck the flow (this was the "step 5 does nothing" bug, fixed 2026-07-06,
present in all 4 apps until then).

---

## 7. Dashboard shell system (portals only)

Same 3-column onboarding grid as apps (`.stage-grid`), but the phone is replaced by a
`.browser` mock (traffic-light dots + fake URL bar) for the access/onboarding steps.
Once onboarding completes, `.hero-inner`-equivalent hides and `.dash` (`display:none` →
`.active`) takes over with a full-width dashboard layout:

```css
.dash-guide{border:1px solid var(--volt-text)}   /* the mandatory 3-step "how to use this" banner — every portal must have one */
.kpi-row{grid-template-columns:repeat(4,1fr)}    /* always exactly 4 headline numbers */
.kpi:hover{transform:translateY(-3px);box-shadow:...;border-color:var(--volt-text)}
.dash-body{grid-template-columns:210px 1fr}      /* sidebar tabs | active panel */
.tab-btn.active{background:var(--tint-04);font-weight:700}
.tbl tr:hover td{background:var(--tint-02)}      /* row hover, added during the "improve effects" pass */
```

Tab content is injected via `TABS[i].render()` returning an HTML string wrapped in
`<div class="b-fade">` for a fade-in on tab switch. Trend indicators in table cells
use `style="color:var(--emerald)"` / `var(--amber)` inline — **never hardcode
`#2FB170`/`#B9740A` etc. here**, that was a real contrast bug (hardcoded hex ignored
the light/dark swap and read as low-contrast in light mode; fixed 2026-07-06).

Every portal must have exactly: onboarding (2–3 steps) → dashboard with a `.dash-guide`
3-step banner → KPI row → sidebar tabs. This is the contract other agents/humans should
follow when adding a 6th portal.

---

## 8. Animation inventory

| Animation | Where | Purpose |
|---|---|---|
| `heroDrift` | `.hero::before` | Slow 26s alternate scale(1→1.06) — subtle Ken Burns drift on the hero photo |
| `heroDrive1/2/3` | `.hero-car.hc1/hc2/hc3` | Small SVG cars driving left→right along the hero's bottom edge, inside the same frame as the photo (not a separate strip — that was retired 2026-07-06) |
| `btnShake` | `.btn:hover` | One-time wiggle, see §5 |
| `carmove` | `.p-map .car` | Car icon sliding along the mini-map route |
| `glow` | `.p-btn` (apps) | Pulsing box-shadow — signals "tap this" |
| `pfade` / `b-fade` | phone/browser screen swaps | Fade+slide-up on step/tab change |
| `spin` | `.p-spin` | Loading spinner rotation |
| `fade` | `.page.active`, mobile menu open | Generic 0.5s/0.3s fade+slide |
| reveal (`.reveal.in`) | almost everything | Scroll-triggered fade+slide, one-shot |
| count-up | `.stat b[data-count]` | Number tween 0→target on scroll into view |

All motion respects `@media(prefers-reduced-motion:reduce)` where added — check for this
block when adding new animations; not every older animation has it yet.

---

## 9. Layout scale

| Token | Value | Notes |
|---|---|---|
| `--maxw` | 1220px (`index.html`, apps) / 1280px (portals) | content wrapper max-width |
| `--radius` | 22px | card corner radius, used everywhere via `var(--radius)` |
| Section padding | `.block{padding:96px 0}`, `64px 0` under 640px | index.html only |
| Grid breakpoints | 1000px (2-col fallback), 640px (1-col + mobile nav), 430px (hide nav CTA) | consistent across index.html |
| App/portal stage grid | `340px 230px 1fr` → `1fr` under 1000px (rail becomes horizontal chips) | apps/portals |

---

## 10. When polishing, check these first

1. **Grep for hardcoded hex before adding new UI** — `grep -n '#[0-9A-Fa-f]\{3,6\}'` in the
   file you're editing, outside of the "always dark" pinned sections and the phone-screen
   `.p-*` rules (which are intentionally fixed-light), any new hardcoded color is probably
   a bug waiting to surface in the other theme.
2. **Test both themes** for anything you touch — the fastest way is toggling `#themeToggle`
   in a headless/real browser and comparing computed `color`/`background` on the element,
   not just eyeballing a screenshot (several real bugs this project shipped were invisible
   in one theme and only showed up in the other).
2. **Test the full flow**, not just the first screen, for any app/portal change — the
   spinner-step bug (§6) only showed up several steps in.
4. **Keep the 9 apps/portals in sync** — they intentionally share identical CSS blocks
   (buttons, phone frame, dashboard shell). If you fix/improve one, propagate the same
   fix to the others rather than letting them drift (this doc exists partly because they
   drifted once already — see the tint-variable bug in §3).
