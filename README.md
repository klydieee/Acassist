# Acassist Design System — "Midnight Glass"

**v1.0** · Build Over Nights · Education Crisis track · Oct 3–4, 2026
**Visual reference:** the glassmorphism onboarding concept — frosted panels, glossy 3D objects, violet→blue gradient CTA on deep navy.
**Scope:** the UI lane of the Team Build Sheet. Core scoring flow first, decoration last.

---

## 0. Read this first (four decisions baked in)

1. **Vanilla, not React.** The Build Sheet locks the UI to raw HTML/CSS/vanilla JS in the Tauri webview. `shadcn/ui` and `lucide-react` are React-only, so they are **not** in the MUST path. Instead:
   - Tokens use **shadcn's exact variable names** (`--background`, `--primary`, `--ring`…), so shadcn/Basecoat themes drop in later.
   - Icons come from vanilla **`lucide`**, bundled locally.
   - **Basecoat** (shadcn-style components for plain HTML) is optional. See §9.
   - If the team *does* switch to React, Appendix A has the shadcn + lucide-react mapping. That is a team call, since it changes the architecture.
2. **Offline applies to the UI too.** No CDN `<script>`, no Google Fonts `<link>`, no remote images. Fonts, icons and images ship inside the app. Otherwise the app looks broken in exactly the classrooms it is built for.
3. **Glass is GPU-expensive, and the target classrooms have low-end hardware.** The system has a built-in `lite` mode (§3.3) that swaps blur for solid translucent surfaces. It turns on automatically for reduced-transparency users and low-core machines.
4. **Muted text was lifted from the reference.** The reference's grey-lavender subcopy measures about 3.2:1 where it sits over the purple glow, which fails WCAG AA. We use `#A9AED6` instead (§11).

---

## 1. Design principles

| # | Principle | In practice |
|---|-----------|-------------|
| 1 | **Spend the boldness in one place** | The glossy 3D objects and frosted glass are the memorable thing. They appear on hero, empty and result moments only. Working screens (quiz, lesson, forms) are quiet glass cards with **one** gradient button. |
| 2 | **Offline is normal, never an error** | The connection chip is informational, never amber or red. Offline = violet "everything works". Online = azure "AI ready". |
| 3 | **One primary action per screen** | One gradient button per view. Everything else is secondary or ghost. |
| 4 | **Honest UI** | Say "Scored on this device". Never "secure", "encrypted", "cheat-proof" or "unbreakable". This matches the pitch's tamper-resistant-not-unbreakable story. |
| 5 | **Readable before pretty** | Lexend, 17px body, 44px hit targets, AA contrast verified. Glass never sits behind long text without enough fill. |
| 6 | **Cheap to render** | Pre-rendered 3D (PNG/WebP), no WebGL, no ambient animation, at most 2 stacked blur layers per screen. |

**Alignment:** centered for onboarding, empty states and results (like the reference). Left-aligned for questions and lessons (reading). Primary button right-aligned in card footers on desktop, full-width in centered hero cards.

**Copy rules:** sentence case, plain verbs, no ALL CAPS labels, no eyebrow labels above headings, no jargon ("seal", "hash", "salt" never appear in the UI).

---

## 2. File layout

```
src/
├─ index.html
├─ styles/
│  ├─ tokens.css          ← §3
│  ├─ base.css            ← §4
│  └─ components.css      ← §5
├─ assets/
│  ├─ fonts/lexend-latin-wght-normal.woff2
│  ├─ vendor/lucide.min.js
│  ├─ decor/              ← 3D objects, WebP  (§8)
│  └─ fallback/           ← pre-generated module JSON (Build Sheet MUST #5)
└─ app.js
```

`index.html` head:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Acassist</title>
  <link rel="stylesheet" href="styles/tokens.css">
  <link rel="stylesheet" href="styles/base.css">
  <link rel="stylesheet" href="styles/components.css">
  <script src="assets/vendor/lucide.min.js" defer></script>
</head>
```

`app.js` bootstrap (performance mode + icons):

```js
const root = document.documentElement;

// Performance mode: user choice wins; otherwise guess from CPU cores.
const savedFx = localStorage.getItem('fx');            // 'lite' | 'full' | null
if (savedFx) root.dataset.fx = savedFx;
else if ((navigator.hardwareConcurrency || 8) <= 4) root.dataset.fx = 'lite';

// Icons: call again after injecting new markup (e.g. each question render).
window.renderIcons = () =>
  lucide.createIcons({ attrs: { class: 'icon', 'aria-hidden': 'true' } });
document.addEventListener('DOMContentLoaded', renderIcons);

// Elementary mode: presentation only, never touches scoring.
// root.dataset.grade = module.module.grade_level;   // SHOULD, see §5.9
```

---

## 3. `styles/tokens.css`

```css
/* Acassist "Midnight Glass" — design tokens
   Semantic names follow shadcn/ui so shadcn & Basecoat themes
   are drop-in compatible. Colors are plain CSS colors (no hsl(var())). */

:root {
  color-scheme: dark;

  /* Primitives */
  /* ink: the deep navy canvas */
  --ink-950:#040720; --ink-900:#070B2E; --ink-850:#0A1040;
  --ink-800:#0E1752; --ink-700:#17226E;
  /* violet: primary brand, left end of the CTA */
  --violet-300:#C9A6FF; --violet-400:#A66BFF; --violet-500:#8B2EFF;
  --violet-600:#7C22F0; --violet-700:#5E14C0;
  /* indigo: right end of the CTA */
  --indigo-500:#3A4BFF;
  /* azure: electric blue (spheres, focus, "online") */
  --azure-300:#7CCBFF; --azure-400:#3DB1FF; --azure-500:#1B8CFF;
  /* orchid: magenta-pink (spheres, card edge glow) */
  --orchid-300:#F0A5F5; --orchid-400:#E070EA; --orchid-500:#C84BDD;
  /* state hues */
  --mint-400:#3DDC97; --amber-400:#FFC23D;
  --rose-400:#FF5C7A; --rose-600:#D6244A;
  /* neutrals */
  --white:#FFFFFF; --periwinkle-300:#A9AED6;   /* muted text, AA-safe on glass */

  /* Semantic (shadcn/ui-compatible) */
  --background:#070B2E;                         /* = --ink-900 */
  --foreground:#F4F5FF;
  --card:var(--glass-fill);
  --card-foreground:var(--foreground);
  --popover:#121A5A;                            /* popovers are opaque on purpose */
  --popover-foreground:var(--foreground);
  --primary:var(--violet-500);
  --primary-foreground:var(--white);
  --secondary:rgba(255,255,255,.08);
  --secondary-foreground:var(--foreground);
  --muted:rgba(255,255,255,.06);
  --muted-foreground:var(--periwinkle-300);
  --accent:rgba(61,177,255,.16);
  --accent-foreground:var(--white);
  --destructive:var(--rose-600);                /* FILLED danger (white text = 4.99:1) */
  --destructive-foreground:var(--white);
  --border:rgba(255,255,255,.16);               /* decorative edges only */
  --input:rgba(255,255,255,.42);                /* CONTROL borders, ≥3:1 (WCAG 1.4.11) */
  --ring:var(--azure-300);
  --radius:1.25rem;

  --chart-1:var(--violet-400); --chart-2:var(--azure-400); --chart-3:var(--orchid-400);
  --chart-4:var(--mint-400);   --chart-5:var(--amber-400);

  --sidebar:rgba(10,16,64,.72);          --sidebar-foreground:var(--foreground);
  --sidebar-primary:var(--violet-500);   --sidebar-primary-foreground:var(--white);
  --sidebar-accent:var(--accent);        --sidebar-accent-foreground:var(--white);
  --sidebar-border:var(--border);        --sidebar-ring:var(--ring);

  /* Brand additions: state colors for TEXT/ICONS on dark */
  --success:var(--mint-400);
  --warning:var(--amber-400);
  --danger:var(--rose-400);
  --info:var(--azure-300);

  /* Glass */
  --glass-fill:rgba(255,255,255,.08);
  --glass-fill-strong:rgba(255,255,255,.12);
  --glass-solid:#1B2260;                 /* opaque stand-ins used in lite mode */
  --glass-solid-strong:#232B6E;
  --glass-filter:blur(16px) saturate(140%);
  --selected-fill:rgba(61,177,255,.18);

  /* Gradients */
  --grad-primary:linear-gradient(100deg,var(--violet-500) 0%,var(--indigo-500) 100%);
  --grad-edge:linear-gradient(155deg,var(--orchid-500) 0%,rgba(200,75,221,0) 38%,
              rgba(27,140,255,0) 62%,var(--azure-500) 100%);   /* magenta → cyan rim */
  --bg-mesh:
    radial-gradient(60% 50% at 16% 20%,rgba(124,34,240,.45),transparent 70%),
    radial-gradient(50% 45% at 88% 92%,rgba(27,140,255,.28),transparent 70%),
    radial-gradient(40% 35% at 72% 8%,rgba(200,75,221,.16),transparent 70%),
    var(--ink-900);

  /* Elevation */
  --shadow-1:0 8px 24px rgba(2,4,24,.45);
  --shadow-2:0 24px 60px rgba(2,4,24,.55);
  --glow-primary:0 8px 28px rgba(139,46,255,.45);

  /* Type (Lexend, self-hosted) */
  --font-sans:"Lexend",ui-sans-serif,system-ui,"Segoe UI",Roboto,"Helvetica Neue",sans-serif;
  --font-mono:ui-monospace,"SF Mono",Menlo,Consolas,monospace;
  --text-xs:.8125rem;  --text-sm:.9375rem; --text-base:1.0625rem; --text-lg:1.25rem;
  --text-xl:1.5rem;    --text-2xl:1.875rem; --text-3xl:2.375rem;  --text-4xl:3rem;
  --leading-tight:1.15; --leading-snug:1.3; --leading-body:1.55; --leading-lesson:1.7;
  --tracking-heading:-.015em;

  /* Space, radius, sizes */
  --space-1:.25rem; --space-2:.5rem;  --space-3:.75rem; --space-4:1rem;
  --space-5:1.25rem; --space-6:1.5rem; --space-8:2rem;   --space-10:2.5rem;
  --space-12:3rem;  --space-16:4rem;
  --radius-sm:.625rem; --radius-md:.875rem; --radius-lg:var(--radius);
  --radius-xl:1.75rem; --radius-2xl:2.25rem; --radius-full:999px;
  --hit-min:44px;            /* minimum touch/click target */
  --content-max:44rem;       /* quiz / lesson column */
  --shell-max:72rem;         /* app chrome */
  --measure:62ch;            /* max line length for lessons */

  /* Motion & layers */
  --ease-out:cubic-bezier(.22,1,.36,1);
  --dur-1:120ms; --dur-2:200ms; --dur-3:360ms;
  --z-base:0; --z-sticky:10; --z-popover:20; --z-modal:30; --z-toast:40;
}

/* 3.1 Elementary mode (SHOULD): bigger type, bigger targets
   Set via  document.documentElement.dataset.grade = 'elementary'.
   Presentation only; never affects scoring. */
:root[data-grade="elementary"] {
  --hit-min:56px;
  --text-sm:1.0625rem; --text-base:1.25rem; --text-lg:1.5rem;
  --text-xl:1.875rem;  --text-2xl:2.25rem;
  --radius:1.5rem;
}

/* 3.2 Light bulb moments for tokens that other modes override */

/* 3.3 Lite mode: no backdrop blur, opaque glass
   Auto: user prefers reduced transparency, or no backdrop-filter support.
   Manual: data-fx="lite" (Settings toggle + low-core auto-detect). */
:root[data-fx="lite"] {
  --glass-filter:none;
  --glass-fill:var(--glass-solid);
  --glass-fill-strong:var(--glass-solid-strong);
}
@media (prefers-reduced-transparency: reduce) {
  :root {
    --glass-filter:none;
    --glass-fill:var(--glass-solid);
    --glass-fill-strong:var(--glass-solid-strong);
  }
}
@supports not ((backdrop-filter:blur(1px)) or (-webkit-backdrop-filter:blur(1px))) {
  :root {
    --glass-filter:none;
    --glass-fill:var(--glass-solid);
    --glass-fill-strong:var(--glass-solid-strong);
  }
}
```

### 3.4 Token cheat-sheet

| Group | Tokens | Rule of thumb |
|-------|--------|---------------|
| Canvas | `--background`, `--bg-mesh` | Mesh sits behind everything (fixed pseudo-element, §4). |
| Surfaces | `--card` / `--glass-fill`, `--glass-fill-strong`, `--popover` | Cards = glass. Popovers/menus = opaque. |
| Text | `--foreground`, `--muted-foreground` | Headings and body in foreground. Hints and subcopy in muted. |
| Action | `--primary`, `--grad-primary`, `--glow-primary` | Gradient + glow on the one primary button only. |
| Borders | `--border` (decor), `--input` (controls) | Anything clickable uses `--input`. |
| Focus | `--ring` | 3px outline, 3px offset, always visible. |
| State text | `--success`, `--danger`, `--warning`, `--info` | Colors for text and icons on dark. For filled danger use `--destructive`. |

---

## 4. `styles/base.css`

```css
/* Acassist — base: font, reset, canvas, typography, layout */

/* Self-hosted variable font. Copy the `latin` wght file from
   node_modules/@fontsource-variable/lexend/files/ (check exact filename). */

@font-face {
  font-family:"Lexend";
  font-style:normal;
  font-weight:100 900;
  font-display:swap;
  src:url("../assets/fonts/lexend-latin-wght-normal.woff2") format("woff2");
}

*,*::before,*::after { box-sizing:border-box; }
html { -webkit-text-size-adjust:100%; }

body {
  margin:0;
  min-height:100vh; min-height:100dvh;
  font:400 var(--text-base)/var(--leading-body) var(--font-sans);
  color:var(--foreground);
  background:var(--background);
  -webkit-font-smoothing:antialiased;
}
/* Fixed mesh canvas as a pseudo-element: cheaper than background-attachment:fixed */
body::before {
  content:""; position:fixed; inset:0; z-index:-1;
  background:var(--bg-mesh);
  pointer-events:none;
}

::selection { background:rgba(139,46,255,.55); color:var(--white); }
a { color:var(--azure-300); text-underline-offset:.2em; }
img { max-width:100%; display:block; }

/* Typography */
h1,h2,h3 {
  margin:0; font-weight:700;
  line-height:var(--leading-tight);
  letter-spacing:var(--tracking-heading);
  text-wrap:balance;
}
h1 { font-size:var(--text-2xl); }
h2 { font-size:var(--text-xl); }
h3 { font-size:var(--text-lg); line-height:var(--leading-snug); }
p  { margin:0; }

.display { font-size:var(--text-3xl); font-weight:700; }      /* hero headline */
.lede {                                                       /* muted subcopy under a headline */
  color:var(--muted-foreground);
  font-size:var(--text-base);
  text-wrap:balance;
}
.lede strong { color:var(--foreground); font-weight:500; }    /* the reference's white emphasis */
.muted { color:var(--muted-foreground); }
.small { font-size:var(--text-sm); }

/* Lesson reading column (renders Build Sheet blocks: heading / paragraph) */
.lesson { max-width:var(--measure); }
.lesson h2 { font-size:var(--text-xl); margin-top:var(--space-8); }
.lesson h2:first-child { margin-top:0; }
.lesson p  { margin-top:var(--space-4); line-height:var(--leading-lesson); }

/* Focus: always visible */
:focus-visible { outline:3px solid var(--ring); outline-offset:3px; }

/* Layout */
.shell  { min-height:100vh; min-height:100dvh; display:grid; grid-template-rows:auto 1fr; }
.topbar {
  display:flex; align-items:center; justify-content:space-between; gap:var(--space-4);
  width:100%; max-width:var(--shell-max); margin:0 auto;
  padding:var(--space-4) var(--space-6);
}
.stage  { display:grid; place-items:center; padding:var(--space-6); position:relative; }
.column { width:100%; max-width:var(--content-max); }
.stack > * + * { margin-top:var(--stack-gap,var(--space-4)); }
.center { text-align:center; }
.row    { display:flex; align-items:center; gap:var(--space-3); }
.row--between { justify-content:space-between; }
.row--end { justify-content:flex-end; }

/* Decorative 3D objects: purely visual, never interactive */
.decor { position:absolute; pointer-events:none; user-select:none; z-index:var(--z-base); }

.sr-only {
  position:absolute; width:1px; height:1px; margin:-1px; padding:0;
  overflow:hidden; clip:rect(0 0 0 0); white-space:nowrap; border:0;
}

/* Icons (lucide, inline SVG) */
.icon {
  width:1.25rem; height:1.25rem; flex:none;
  stroke-width:1.75;                      /* lighter than lucide's default 2 */
  stroke-linecap:round; stroke-linejoin:round;
}
.icon--sm { width:1rem; height:1rem; }
.icon--lg { width:1.75rem; height:1.75rem; }

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  *,*::before,*::after {
    animation-duration:.01ms !important; animation-iteration-count:1 !important;
    transition-duration:.01ms !important; scroll-behavior:auto !important;
  }
}
```

---

## 5. Components

### 5.1 Inventory (maps to the Build Sheet's MUST / SHOULD / ROADMAP)

| Component | Where | Priority | shadcn/ui equivalent | Our class |
|-----------|-------|----------|----------------------|-----------|
| Button (primary / secondary / ghost / danger) | everywhere | **MUST** | `button` | `.btn` |
| Glass card | every screen | **MUST** | `card` | `.card`, `.glass` |
| Text input / textarea | identification questions, teacher form | **MUST** | `input`, `textarea`, `label` | `.input` |
| Option (radio card) | multiple choice, true/false | **MUST** | `radio-group` | `.option-list`, `.option` |
| Progress bar | quiz | **MUST** | `progress` | `.progress` |
| Connection chip | top bar | **MUST** | `badge` | `.chip[data-conn]` |
| Alert (inline) | errors, AI-fallback notice | **MUST** | `alert` | `.alert` |
| Score number + ring | result | MUST (number) / SHOULD (ring) | none | `.score-ring` |
| Template picker | teacher scaffolds | SHOULD | `select` / `toggle-group` | native `<select class="input">` |
| Performance toggle | settings | SHOULD | `switch` | native `<input type="checkbox" role="switch">` |
| Flashcard (flip) | study view | SHOULD (first thing cut) | `card` | `.flashcard` (spec only) |
| Dialog, toast, tabs, table | n/a | ROADMAP | n/a | use `.alert` inline until needed |

### 5.2 `styles/components.css`

```css
/* Acassist — components
   Flat, single-class selectors on purpose (no specificity fights). */

/* Glass surface */
.glass {
  position:relative; isolation:isolate;
  border-radius:var(--radius-xl);
  background:var(--glass-fill);
  -webkit-backdrop-filter:var(--glass-filter);
          backdrop-filter:var(--glass-filter);
  box-shadow:var(--shadow-1);
}
/* Gradient rim (magenta top-left → azure bottom-right) via masked border */
.glass::before {
  content:""; position:absolute; inset:0; padding:1px;
  border-radius:inherit; background:var(--grad-edge);
  -webkit-mask:linear-gradient(#000 0 0) content-box,linear-gradient(#000 0 0);
  -webkit-mask-composite:xor;
          mask-composite:exclude;
  pointer-events:none;
}
.card { composes:glass; }   /* (plain CSS has no `composes`; see next rule) */
.card {
  position:relative; isolation:isolate;
  padding:var(--space-8);
  border-radius:var(--radius-xl);
  background:var(--glass-fill);
  -webkit-backdrop-filter:var(--glass-filter);
          backdrop-filter:var(--glass-filter);
  box-shadow:var(--shadow-1);
}
.card::before {
  content:""; position:absolute; inset:0; padding:1px;
  border-radius:inherit; background:var(--grad-edge);
  -webkit-mask:linear-gradient(#000 0 0) content-box,linear-gradient(#000 0 0);
  -webkit-mask-composite:xor; mask-composite:exclude;
  pointer-events:none;
}

/* Button─ */
.btn {
  --h:var(--hit-min);
  display:inline-flex; align-items:center; justify-content:center; gap:var(--space-2);
  min-height:var(--h); padding:0 var(--space-6);
  border:0; border-radius:var(--radius-lg);
  font:600 var(--text-base)/1 var(--font-sans);
  color:var(--primary-foreground);
  background:var(--grad-primary);
  box-shadow:var(--glow-primary), inset 0 1px 0 rgba(255,255,255,.25);
  cursor:pointer;
  transition:transform var(--dur-1) var(--ease-out), filter var(--dur-1) var(--ease-out);
}
.btn:hover    { filter:brightness(1.1); }
.btn:active   { transform:translateY(1px) scale(.99); }
.btn:disabled { opacity:.45; box-shadow:none; cursor:not-allowed; filter:none; transform:none; }
.btn--lg      { --h:56px; font-size:var(--text-lg); }
.btn--block   { width:100%; }
.btn--secondary {
  background:var(--secondary); color:var(--secondary-foreground);
  box-shadow:inset 0 0 0 1px var(--input);
}
.btn--ghost {                                  /* "Already have an account?" style */
  background:transparent; color:var(--muted-foreground); box-shadow:none;
}
.btn--ghost:hover { color:var(--foreground); background:var(--muted); filter:none; }
.btn--danger { background:var(--destructive); color:var(--destructive-foreground); box-shadow:none; }

/* Input */
.input {
  width:100%; min-height:var(--hit-min); padding:var(--space-2) var(--space-4);
  border-radius:var(--radius-md);
  border:1px solid var(--input);
  background:rgba(4,7,32,.45);
  color:var(--foreground);
  font:400 var(--text-base)/var(--leading-snug) var(--font-sans);
}
textarea.input { min-height:7rem; resize:vertical; padding-top:var(--space-3); }
.input::placeholder { color:var(--muted-foreground); opacity:1; }
.input:focus-visible { outline:3px solid var(--ring); outline-offset:2px; }
.input[aria-invalid="true"] { border-color:var(--danger); }
.label { display:block; margin-bottom:var(--space-2); font-weight:500; font-size:var(--text-sm); }
.hint  { margin-top:var(--space-2); color:var(--muted-foreground); font-size:var(--text-sm); }

/* Option (radio card) — multiple choice & true/false
   Sibling structure (input + label) so no :has() is needed on older WebKitGTK.
   NO backdrop-filter on options: fill only (perf). */
.option-list        { display:grid; gap:var(--space-3); }
.option-list--pair  { grid-template-columns:1fr 1fr; }      /* true / false */
.option-input       { position:absolute; opacity:0; pointer-events:none; }
.option {
  display:flex; align-items:center; gap:var(--space-4);
  min-height:var(--hit-min); padding:var(--space-3) var(--space-5);
  border-radius:var(--radius-lg);
  border:1px solid var(--input);
  background:var(--glass-fill);
  cursor:pointer;
  transition:background var(--dur-1) var(--ease-out), border-color var(--dur-1) var(--ease-out);
}
.option:hover { background:var(--glass-fill-strong); }
.option__mark {
  width:1.5rem; height:1.5rem; flex:none; border-radius:50%;
  border:2px solid var(--input);
}
.option-input:checked + .option {
  background:var(--selected-fill);
  border-color:var(--azure-300);
  box-shadow:inset 0 0 0 1px var(--azure-300);
}
.option-input:checked + .option .option__mark {
  border-color:var(--azure-300);
  background:var(--azure-300);
  box-shadow:inset 0 0 0 4px #243D6F;               /* ring-dot look, no extra markup */
}
.option-input:focus-visible + .option { outline:3px solid var(--ring); outline-offset:3px; }

/* Review state after submit. Never color alone: icon + word as well. */
.option__state { display:none; margin-left:auto; align-items:center; gap:var(--space-2); font-size:var(--text-sm); font-weight:500; }
.option[data-state] .option__state { display:inline-flex; }
.option[data-state="correct"]   { border-color:var(--success); box-shadow:inset 0 0 0 1px var(--success); }
.option[data-state="correct"]   .option__state { color:var(--success); }
.option[data-state="incorrect"] { border-color:var(--danger);  box-shadow:inset 0 0 0 1px var(--danger); }
.option[data-state="incorrect"] .option__state { color:var(--danger); }

/* Progress─ */
.progress { height:.625rem; border-radius:var(--radius-full); background:rgba(255,255,255,.14); overflow:hidden; }
.progress > span {
  display:block; height:100%; border-radius:inherit; background:var(--grad-primary);
  transition:width var(--dur-3) var(--ease-out);
}

/* Chip (connection status etc.) */
.chip {
  display:inline-flex; align-items:center; gap:var(--space-2);
  padding:.375rem .875rem; border-radius:var(--radius-full);
  font:500 var(--text-sm)/1 var(--font-sans);
  color:var(--chip-fg,var(--foreground)); background:var(--chip-bg,var(--muted));
  box-shadow:inset 0 0 0 1px var(--chip-ring,var(--border));
}
.chip[data-conn="offline"] { --chip-bg:rgba(166,107,255,.16); --chip-fg:var(--violet-300); --chip-ring:rgba(166,107,255,.45); }
.chip[data-conn="online"]  { --chip-bg:rgba(61,177,255,.14);  --chip-fg:var(--azure-300);  --chip-ring:rgba(61,177,255,.45); }
.chip--success { --chip-bg:rgba(61,220,151,.14); --chip-fg:var(--success); --chip-ring:rgba(61,220,151,.45); }

/* Alert (inline; replaces toast/dialog for v1)─ */
.alert {
  display:flex; gap:var(--space-3); align-items:flex-start;
  padding:var(--space-4) var(--space-5); border-radius:var(--radius-lg);
  background:var(--glass-fill-strong);
  box-shadow:inset 0 0 0 1px var(--alert-ring,var(--border));
}
.alert .icon { color:var(--alert-icon,var(--info)); margin-top:.15rem; }
.alert[data-tone="error"]   { --alert-ring:var(--danger);  --alert-icon:var(--danger); }
.alert[data-tone="warning"] { --alert-ring:var(--warning); --alert-icon:var(--warning); }
.alert[data-tone="success"] { --alert-ring:var(--success); --alert-icon:var(--success); }
.alert[data-tone="info"]    { --alert-ring:var(--info);    --alert-icon:var(--info); }

/* Score ring (SHOULD). Number is the MUST; ring is garnish.
   JS sets --pct (0–100); a count-up tween is the one "orchestrated moment". */
.score-ring {
  --pct:0; position:relative; display:grid; place-items:center;
  width:11rem; aspect-ratio:1; border-radius:50%;
  background:conic-gradient(var(--azure-400) calc(var(--pct) * 1%), rgba(255,255,255,.14) 0);
}
.score-ring::before { content:""; position:absolute; inset:.75rem; border-radius:50%; background:var(--ink-850); }
.score-ring > * { position:relative; text-align:center; }
.score-ring__num { font-size:var(--text-3xl); font-weight:700; line-height:1; }
```

> Delete the stray `.card { composes:glass; }` line when pasting. It is a reminder that `.card` is `.glass` plus padding, and plain CSS can't compose. If you prefer less duplication, use `<div class="glass card-pad">` and add `.card-pad{padding:var(--space-8)}`.

### 5.3 Reference markup: multiple-choice question card

```html
<section class="card column stack" aria-labelledby="q-prompt">
  <div class="row row--between">
    <span class="muted small">Question 3 of 10</span>
    <div class="progress" style="width:40%" role="progressbar"
         aria-label="Question 3 of 10" aria-valuemin="0" aria-valuemax="10" aria-valuenow="3">
      <span style="width:30%"></span>
    </div>
  </div>

  <h2 id="q-prompt">Which gas do plants take in?</h2>

  <div class="option-list" role="radiogroup" aria-labelledby="q-prompt">
    <input class="option-input" type="radio" name="q3" id="q3a" value="Oxygen">
    <label class="option" for="q3a"><span class="option__mark"></span><span>Oxygen</span>
      <span class="option__state"><i data-lucide="circle-check"></i>Correct</span></label>

    <input class="option-input" type="radio" name="q3" id="q3b" value="Carbon Dioxide">
    <label class="option" for="q3b"><span class="option__mark"></span><span>Carbon Dioxide</span>
      <span class="option__state"><i data-lucide="circle-check"></i>Correct</span></label>
    <!-- … -->
  </div>

  <div class="row row--end">
    <button class="btn" type="button" disabled>Next question</button>
  </div>
</section>
```

Per question `kind` (Build Sheet contract):

| `kind` | Renders as |
|--------|-----------|
| `multiple_choice` | `.option-list` with 2–4 `.option` rows |
| `true_false` | `.option-list.option-list--pair` with two large options |
| `identification` | `<label class="label">` + `<input class="input" type="text" autocomplete="off" spellcheck="false">` |

Keep the **Next / Submit button disabled until an answer exists**. For identification questions, show the exact normalization rule as a hint, e.g. "Capitalization and punctuation don't matter".

### 5.4 Review states and a design consequence of the offline hash

The shipped module has **no plaintext answers**. So:

- Review can mark a question ✓ or ✗ (the core returns correct/incorrect).
- For multiple choice, you *could* show which option was right by checking each option's hash. Decide with the Rust lane first.
- For identification, there is **no "the correct answer was ___" slot**. Don't design one, unless the teacher supplied an `explanation`.
- The result screen shows the score and "Scored on this device". It does not show an answer key.

### 5.5 Connection chip (top bar)

```html
<span class="chip" data-conn="offline"><i data-lucide="wifi-off"></i>Offline · everything works</span>
<span class="chip" data-conn="online"><i data-lucide="sparkles"></i>Online · AI ready</span>
```

Rule: **never use warning or danger colors for connection state.** Offline is the product's core promise. Teachers see the online chip, students most often the offline one.

### 5.6 Alert copy (the failure moments in scope)

| Moment | Tone | Copy |
|--------|------|------|
| Groq call fails, fallback module loads | info | "AI isn't reachable right now. Showing the saved example module instead." |
| Opened a non-module file | error | "That file isn't an Acassist module. Choose a module file ending in .json." |
| Module version unknown | warning | "This module was made with a newer version of Acassist and may not open correctly." |
| Quiz finished offline | success | "Scored on this device. No connection needed." |

Pattern: say what happened, then what to do. No apologies, no vague "something went wrong".

### 5.7 Lesson view

Render `lesson.blocks[]` with `document.createElement` and **`textContent`** (never `innerHTML`). Module files come from other people, and the Tauri webview can reach native commands. `kind:"heading"` → `<h2>`, `kind:"paragraph"` → `<p>`, all inside `.lesson`. Cap line length at `--measure`, left-aligned, 1.7 line-height.

### 5.8 Flashcard spec (SHOULD, cut first)

Same JSON, another view. Front = `prompt`, back = the answer option or teacher explanation.
- One `.card` per face. Flip with `transform: rotateY(180deg)` and `backface-visibility:hidden`, 360ms, `--ease-out`.
- Under `prefers-reduced-motion`, swap to an instant crossfade (the global reduced-motion rule already shortens the transition).
- Controls: "Flip", "Previous", "Next" as `.btn--secondary`, with arrow keys and Space for keyboard.
- No spaced-repetition logic (roadmap).

### 5.9 Elementary mode (SHOULD)

If the renderer lane agrees, set `document.documentElement.dataset.grade = module.module.grade_level`. This only swaps tokens (56px targets, +1 type step, rounder corners; see §3.1). Note the schema marks `grade_level` as "metadata only": this changes **presentation, never scoring**. Confirm with the renderer lane before wiring it.

---

## 6. Screens and wireframes

| # | Screen | Build Sheet scope | Alignment |
|---|--------|------------------|-----------|
| 1 | Home / open module | MUST (#2 loader) | centered |
| 2 | Lesson | MUST (#4) | left |
| 3 | Quiz-take | MUST (#3) | left |
| 4 | Result | MUST (#3) | centered |
| 5 | Teacher: generate module | MUST (#5, online) | left |
| 6 | Flashcards | SHOULD | centered |
| 7 | Settings (performance mode) | SHOULD | left |

**1 · Home (hero moment: allowed decor)**

```
┌──────────────────────────────────────────────────────────────┐
│ ◆ Acassist                          (⌁ Offline · everything works)
│                                                              │
│   ◯ sphere                                      ◇ cube       │
│                  ┌────────────────────────┐                  │
│                  │  Lessons that work     │                  │
│                  │  without Wi-Fi         │   (display, centered)
│                  │  Open a module once,   │                  │
│                  │  use it anywhere.      │   (lede)         │
│                  │                        │                  │
│                  │  [   Open a module   ] │   (btn--lg block)│
│                  │   Recent modules       │   (ghost list)   │
│                  └────────────────────────┘                  │
│        ◌ torus                                               │
└──────────────────────────────────────────────────────────────┘
```

**3 · Quiz-take (working screen: no decor)**

```
┌──────────────────────────────────────────────────────────────┐
│ ‹ Photosynthesis Basics                (⌁ Offline · everything works)
│                                                              │
│            ┌──────────── .card .column ───────────┐          │
│            │ Question 3 of 10        ▓▓▓░░░░░░░   │          │
│            │                                      │          │
│            │ Which gas do plants take in?         │          │
│            │                                      │          │
│            │ ┌ ○  Oxygen ─────────────────────┐   │          │
│            │ ├ ◉  Carbon Dioxide ─────────────┤   │ selected │
│            │ ├ ○  Nitrogen ───────────────────┤   │          │
│            │ └ ○  Hydrogen ───────────────────┘   │          │
│            │                    [ Next question ] │          │
│            └──────────────────────────────────────┘          │
└──────────────────────────────────────────────────────────────┘
```

**4 · Result (one orchestrated moment: count-up + ring)**

```
            ┌────────────────────────────────┐
            │             ◜ 80 ◝             │   .score-ring
            │          8 of 10 correct       │   h1
            │  Scored on this device. No     │   lede
            │  connection needed.            │
            │  [ Review answers ] [ Try again ]│  primary / secondary
            └────────────────────────────────┘
```

---

## 7. Icons: Lucide (vanilla)

**Setup (no build step):**

```bash
npm i lucide
cp node_modules/lucide/dist/umd/lucide.min.js src/assets/vendor/
```

Use `<i data-lucide="wifi-off"></i>`, then call `renderIcons()` (§2) after injecting any new markup. Local file, so it works offline. Stroke 1.75 (set in `.icon`), sizes 16/20/24/28. The set matches the thin-outline weather icons in the reference.

Icons are `currentColor`, so they inherit text color. Pair them with text wherever meaning matters (the state icons above all have a word beside them).

**Starter set** (names per current Lucide; if one doesn't resolve, search lucide.dev/icons, since some were renamed):

| Use | Icon |
|-----|------|
| Brand / modules | `graduation-cap`, `book-open`, `notebook-pen` |
| Open module | `folder-open`, `file-json` |
| Lesson / quiz / flashcards | `book-open-check`, `list-checks`, `layers` |
| Connection | `wifi-off`, `wifi`, `cloud-off`, `hard-drive` |
| AI (teacher side, online) | `sparkles`, `wand-sparkles` |
| Scoring | `circle-check`, `circle-x`, `trophy`, `badge-check` |
| Feedback | `info`, `triangle-alert` |
| Navigation | `chevron-left`, `chevron-right`, `arrow-right`, `rotate-ccw` (retake) |
| Settings / performance | `settings`, `gauge`, `zap` |
| Save / export | `save`, `download`, `file-plus` |

> Avoid `lock`, `shield-check` or `shield` on scoring UI. They imply security guarantees the pitch deliberately does not make.

---

## 8. Assets

### 8.1 The 3D "decor" objects (the one memorable thing)

The reference's look comes from five primitives: glossy **sphere** (azure), **sphere** (orchid), rounded **cube** (azure and orchid), **torus** (orchid). Education-flavored extras for empty and result states: **book, pencil, light bulb, trophy, check**.

| Option | Cost | Fit | Notes |
|--------|------|-----|-------|
| **Blender**, render your own | ~1–2 hrs | exact match | Recipe below. Best fidelity. |
| **3dicons** (3dicons.co, CC0, PNG + Blender files) | minutes | good; recolor via Blender source | Browse for book, pencil, trophy, bulb, and check availability. |
| **Fluent Emoji 3D** (microsoft/fluentui-emoji) | minutes | softer "clay" style, not glass | Fast kid-friendly objects. Style differs from the glossy primitives. |
| **Spline** (web 3D editor) | ~1 hr | good | Export transparent PNG. Check free-tier export terms. |

**Blender recipe for the glossy look:** Principled BSDF, base color from tokens (`--azure-500`, `--orchid-500`, `--violet-500`), Roughness ≈ 0.15–0.2, Coat weight 1 (Coat roughness ≈ 0.05), soft area light top-left, a magenta rim light behind, Eevee, transparent film, 1600×1600 PNG.

**Frosted glass slabs** (the card behind the shapes) are CSS, not assets. That's `.glass`.

**Asset spec:**

| Rule | Value |
|------|-------|
| Format | PNG master, shipped as **WebP with alpha** (convert with Squoosh or `cwebp`) |
| Size | ≤ 1200px longest side, ≤ 80 KB each, ≤ 500 KB total |
| Count | 6–8 objects, reused (flip/scale/rotate) across screens |
| Naming | `sphere-azure.webp`, `sphere-orchid.webp`, `cube-azure.webp`, `cube-orchid.webp`, `torus-orchid.webp`, `book.webp`, `trophy.webp` |
| Markup | `<img class="decor" src="assets/decor/cube-azure.webp" alt="" width="220" height="220" style="top:12%;left:8%">` |
| Behavior | **Static.** No float or parallax animation. Hidden on the quiz screen. |
| Lite mode | May hide all `.decor` (`:root[data-fx="lite"] .decor{display:none}`) if the machine is struggling |

### 8.2 Logo / app icon

Mark = the azure glass **cube with a check** ("answer verified"). Wordmark = "Acassist", Lexend 700. Make a 1024×1024 PNG, then run `npm run tauri icon path/to/icon.png` to generate every platform icon Tauri needs.

### 8.3 Background

100% CSS (`--bg-mesh` + `body::before`). No image, no canvas, no animated gradient. Keep the brightest purple glow behind decor, not behind long text, so muted text stays AA (§11).

### 8.4 Font

**Lexend** (OFL-1.1): a geometric sans designed for reading fluency, a close match to the reference's rounded geometric headings. The reference's exact typeface can't be identified from the image. Self-host one variable file (weights 100–900). The `latin` subset covers English and Filipino (ñ included). Fallback stack is in `--font-sans`.

---

## 9. Libraries and repositories

### 9.1 UI layer (all vendored locally, zero CDN)

| Need | Pick | Repo / package | When |
|------|------|----------------|------|
| Icons | **Lucide** (vanilla `lucide` package, ISC) | github.com/lucide-icons/lucide · npm `lucide` | MUST |
| Font | **Lexend** (OFL-1.1) | github.com/googlefonts/lexend · npm `@fontsource-variable/lexend` (Fontsource: github.com/fontsource/fontsource) | MUST |
| shadcn-style components, no React | **Basecoat** (MIT, Tailwind v4, shadcn-theme compatible, small vanilla JS) | github.com/hunvreus/basecoat · npm `basecoat-css` · basecoatui.com | **Optional.** Adds Tailwind and a build step. §5's CSS covers every MUST component, so adopt only if you want dialog/select/tabs for free. |
| shadcn reference (tokens, patterns) | shadcn/ui | github.com/shadcn-ui/ui · ui.shadcn.com | Reference only (React track, Appendix A) |
| PNG → WebP | Squoosh | github.com/GoogleChromeLabs/squoosh | MUST for decor |
| Score celebration | canvas-confetti | github.com/catdad/canvas-confetti | Optional, SHOULD tier. Skip if short on time. |
| Lesson markdown (only if paragraphs contain markdown) | marked + DOMPurify | github.com/markedjs/marked · github.com/cure53/DOMPurify | Optional. Prefer plain `textContent` blocks (§5.7). If you do use marked, **always** sanitize with DOMPurify. |

### 9.2 App shell and core (non-UI, for the other lanes)

| Need | Pick | Repo / docs | Notes |
|------|------|-------------|-------|
| Desktop shell | **Tauri v2** | github.com/tauri-apps/tauri · `npm create tauri-app@latest` → *Vanilla* | Matches "raw HTML/CSS/JS in the webview". |
| Open local module file | Tauri **dialog** + **fs** plugins | github.com/tauri-apps/plugins-workspace · `npm run tauri add dialog` / `fs` | Simulates "pushed" modules; no distribution to build. |
| SHA-256 | `sha2` + `hex` crates | github.com/RustCrypto/hashes | Seal and check, in Rust only. |
| JSON | `serde`, `serde_json` | crates.io | Module schema. |
| Seal-time salts | `rand` (or `getrandom`) | crates.io | Per-question random salt, teacher side. |
| Unicode parity (optional) | `unicode-normalization` | github.com/unicode-rs/unicode-normalization | Helps keep Rust normalization identical to JS `String.prototype.normalize`. |
| Groq call | `reqwest` (Rust) → Groq's OpenAI-compatible chat endpoint | console.groq.com/docs | Call from **Rust**, not the webview, so the API key never sits in front-end code. Pick the model from Groq's current list. |

**Gotcha worth 10 minutes:** the normalization contract (`lowercase|trim|strip_punctuation`) must produce byte-identical output in Rust and JS. "Punctuation" differs between ASCII-only checks and Unicode categories. Pick one definition and write a shared `normalization-vectors.json` of inputs/expected outputs that both sides test against.

### 9.3 Offline CSP (put in `tauri.conf.json` → `app.security.csp`)

Starting point, to confirm against the Tauri v2 CSP docs (v2.tauri.app/security/csp):

```json
"csp": "default-src 'self'; img-src 'self' data: blob:; font-src 'self'; style-src 'self' 'unsafe-inline'; script-src 'self'; connect-src ipc: http://ipc.localhost"
```

This also *enforces* the "no CDN" rule: a stray remote font or script fails loudly in dev instead of silently in a classroom.

### 9.4 Deliberately skipped

Tailwind (unless Basecoat), three.js/WebGL, Framer Motion/GSAP, Lottie, icon fonts, Google Fonts, any UI kit that needs a runtime framework. All conflict with offline-first, low-end hardware, or "resist your own cleverness".

---

## 10. Voice and microcopy

| Do | Don't |
|----|-------|
| Name the action: "Open a module", "Start quiz", "Next question", "Submit answers", "Try again", "Generate module", "Save module" | "Submit", "OK", "Continue", "Go" |
| Same word through a flow: button "Save module" → message "Module saved." | Switching verbs mid-flow |
| "Scored on this device. No connection needed." | "Secure", "encrypted", "cheat-proof", "unbreakable", "protected" |
| "Offline · everything works" | "No connection", "You are offline" |
| Teacher: "Pick a template, then add your topic." | A blank prompt box, or "Enter your prompt" |
| Explain, then direct: "That file isn't an Acassist module. Choose a .json module file." | "Error 400", "Something went wrong", "Sorry!" |

Note: the reference image contains a typo ("Get starded"). Don't copy it.

---

## 11. Accessibility and verified contrast

Ratios calculated with WCAG 2.x relative luminance. "Glass" = 8% white over `--ink-850`. "Glow" = glass over the brightest purple haze (`#4A1A80`).

| Pair | Ratio | Result |
|------|------:|:------:|
| White on `--ink-900` canvas | 19.2 | ✅ AAA |
| White on CTA violet end `#8B2EFF` | 5.34 | ✅ AA |
| White on CTA blue end `#3A4BFF` | 5.78 | ✅ AA |
| Muted `#A9AED6` on `--ink-850` | 8.37 | ✅ AAA |
| Muted `#A9AED6` on glass | 6.90 | ✅ AA |
| Muted `#A9AED6` on glow | 4.53 | ✅ AA (just) |
| Reference's muted `#8C90B8` on glow | 3.17 | ❌ replaced |
| Control border (white @42%) on glass | ≈3.3 | ✅ ≥3:1 non-text |
| Decorative border (white @16%) on bg | 1.57 | decor only; never on controls |
| Focus ring `--azure-300` on canvas | 10.2 | ✅ |
| Success `--mint-400` on glass | 8.44 | ✅ |
| Danger text `--rose-400` on glass | 5.02 | ✅ |
| White on filled `--rose-600` | 4.99 | ✅ |
| Info `--azure-300` on glass | 8.41 | ✅ |
| Muted on `--glass-solid` (lite mode) | 6.72 | ✅ |

**Checklist (ship gate):**
- [ ] Every control reachable and operable by keyboard; focus ring always visible.
- [ ] Radio options are real `<input type="radio">` in a `radiogroup` with the question as its label.
- [ ] Right/wrong is never color-only (icon + word).
- [ ] 44px minimum targets (56px in elementary mode).
- [ ] `prefers-reduced-motion` and `prefers-reduced-transparency` respected (both are in the CSS).
- [ ] Decor images have `alt=""`; icons have `aria-hidden="true"`.
- [ ] Re-check contrast if a bright decor object ever sits behind text.

---

## 12. Suggested order for the UI lane (12-hour block)

| Window | Deliver | Why |
|--------|---------|-----|
| Hours 0–1 | `tokens.css`, `base.css`, font, local Lucide, app shell with top bar and chip | Unblocks the other lanes immediately; they only need class names. |
| Hours 1–3 | `.btn`, `.card`, `.input`, `.option`, `.progress`, `.alert` → **quiz-take screen** | This is the sacred flow's UI. |
| Hours 3–5 | Result screen, lesson typography, teacher form (template picker, "Generate module") | Completes MUST #3, #4, #5. |
| **■ Core checkpoint holds** | Then and only then: | Per the Build Sheet, the core wins every time. |
| Hours 8+ | Decor objects + Home hero, score ring and count-up, lite-mode toggle, elementary mode, flashcards | All SHOULD / polish. Flashcards are cut first. |

**Roadmap (design side, not built at the event):** light/daylight theme for bright classrooms, per-school theming, TTS/read-aloud controls, native window vibrancy, richer illustration set, localization-ready type scale.

---

## Appendix A: If the team chooses the React track (shadcn/ui + lucide-react)

This contradicts the Build Sheet's "raw HTML/CSS/vanilla JS" decision and adds Vite and a build step, so it needs a team decision, not a unilateral switch. If it happens:

1. Scaffold: `npm create tauri-app@latest` → React + TypeScript, then `npx shadcn@latest init`.
2. Paste the `:root { … }` semantic block from §3 into `globals.css`. Our variable names already match shadcn's, so the theme applies with no renaming. Keep the glass, gradient and lite-mode tokens as extras.
3. Add components:
   ```bash
   npx shadcn@latest add button card input textarea label radio-group progress badge alert select switch skeleton
   ```
4. Icons: `npm i lucide-react` and import only what you use (tree-shaken), e.g. `import { WifiOff, Sparkles } from "lucide-react"`.
5. Map classes: `.btn` → `Button` (restyle `default` variant with `--grad-primary`), `.card` → `Card` + `.glass`, `.option` → `RadioGroupItem` in a card-styled label, `.chip` → `Badge`, `.alert` → `Alert`.
6. Keep every rule in §0–§1: self-hosted font, no CDN, lite mode, no security claims in copy.
