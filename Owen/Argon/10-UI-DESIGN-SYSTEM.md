# 10 — UI DESIGN SYSTEM & DESIGN TOKENS

**Document Ref:** TC-DOC-10-DESIGN-SYSTEM  
**System:** Visual Design Tokens, Typography & Component Standards  
**Architectural Authority:** Level 2 Design Specification  

---

## 1. Color Palette & Semantic Tokens

The visual character is a **Sports Media + Modern Editorial Hybrid** adhering strictly to Manchester United heritage:

```css
:root {
  /* Brand Primary Colors */
  --color-brand-primary: #DA020E;     /* Manchester Red */
  --color-brand-secondary: #0B0E14;   /* Dark Navy / Deep Onyx */
  --color-brand-accent: #FBE122;      /* Cup Gold Accent */

  /* Surface & Neutral Colors */
  --surface-background: #0B0E14;      /* Pure Dark Background */
  --surface-card: #141A23;            /* Elevated Card Surface */
  --surface-card-hover: #1C2430;      /* Card Hover State */
  --surface-border: #242D3D;          /* Subtle Dividing Borders */

  /* Text & Foreground */
  --text-primary: #FFFFFF;            /* High Contrast Headings */
  --text-secondary: #94A3B8;          /* Slate Body Text */
  --text-muted: #64748B;              /* Timestamps & Bylines */

  /* Semantic Status Colors */
  --status-live: #EF4444;             /* Pulsing Matchday Red */
  --status-confirmed: #10B981;        /* Confirmed Green */
  --status-rumour: #F59E0B;           /* Speculative Amber */
  --status-stale: #EAB308;            /* Outdated / Degraded Yellow */
}
```

---

## 2. Typography Scale

- **Display & Headings:** `Barlow Condensed` / `Oswald` (Bold, uppercase, athletic editorial sports feel).
- **Body & Takeaways:** `Inter` / `Plus Jakarta Sans` (High legibility on mobile screens).
- **Match Clock & Numeric Digits:** `JetBrains Mono` / Tabular Numerics (Zero horizontal jitter on live clock updates).

---

## 3. Core Component Specifications

### A. Live Match Scoreboard Card
- Solid Dark Navy background with a 2px top border in Manchester Red.
- Opponent crests rendered at 48x48px with high-DPI crispness.
- Central score digits rendered at 36px bold tabular font.
- Pulsing red badge: `● LIVE 68'`.

### B. Editorial Takeaway Box
- Elevated card surface with a 3px Manchester Red vertical left accent border.
- Bold heading: **RINGKASAN UTAMA (3 POIN)**.
- Unordered list with red bullet points and crisp white text.

### C. Transfer Intelligence Tag
- If `status == 'CONFIRMED'`: Emerald badge (`bg-emerald-500/20 text-emerald-400 border border-emerald-500/30`).
- If `status == 'RUMOUR'`: Amber badge (`bg-amber-500/20 text-amber-400 border border-amber-500/30`).