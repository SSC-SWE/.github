# Styling Standard

The single source of truth for how the site is styled. This document applies to **every repo** in the site; code review enforces it. When a repo needs a new color, spacing value, or component style, add it here first, then use it.

**Version 2 — Tailwind-first.** Tailwind CSS is the styling system for all new work. Repos written against v1 (plain CSS, BEM-lite) — `transcriber-web`, `landing-web` — keep their existing stylesheets until migrated; never mix the two systems inside one app.

## 1. Approach

- **Tailwind CSS v4** is the only styling system. One entry stylesheet per app, `src/theme.css`, imported once at the entry point (`main.tsx`).
- Install with `@tailwindcss/vite`. No `tailwind.config.js` — configuration lives in CSS (`@theme`).
- **Components:** `shadcn/ui` primitives are vendored into `components/ui` and owned by the repo. `Arc UI` blocks from the `@uiarc` registry may be installed alongside them; the CSS modules they ship are the only CSS modules allowed.
- **No** styled-components, emotion, styled-jsx, Sass/Less, or hand-written CSS modules.
- Every design value comes from a token (Section 2). Utilities are the public API for those tokens.

## 2. Design tokens

Two layers in `theme.css`: raw values in `:root`, then `@theme inline` exposing them as Tailwind utilities.

```css
@import 'tailwindcss';

:root {
  color-scheme: dark;

  /* Surfaces */
  --background: #101010;
  --surface: #171717;
  --surface-2: #1f1f1f;
  --surface-3: #262626;

  /* Lines */
  --border: rgba(255, 255, 255, 0.09);
  --border-strong: #393939;

  /* Text */
  --foreground: #ededed;
  --muted-foreground: #a6a6a6;
  --faint-foreground: #8a8a8a;

  /* Semantic */
  --accent: #6366f1;
  --accent-soft: rgba(99, 102, 241, 0.14);
  --ok: #22c55e;
  --warn: #f59e0b;
  --danger: #ef4444;

  /* Motion */
  --ease-out: cubic-bezier(0.23, 1, 0.32, 1);
}

@theme inline {
  --color-background: var(--background);
  --color-surface: var(--surface);
  --color-surface-2: var(--surface-2);
  --color-surface-3: var(--surface-3);
  --color-foreground: var(--foreground);
  --color-muted-foreground: var(--muted-foreground);
  --color-faint: var(--faint-foreground);
  --color-border: var(--border);
  --color-accent: var(--accent);
  --color-ok: var(--ok);
  --color-warn: var(--warn);
  --color-danger: var(--danger);

  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
}
```

Rules for extending:

- **Add a token before adding a value.** If a rule needs a value not in `:root`, add the token — never inline it in a class.
- Name by **role**, not by look: `--danger`, `--muted-foreground`, `--surface-2` — never `--red`, `--gray-2`, `--line-3`.
- Keep the **names** stable across apps; values may differ per app (an app may choose its own accent hue and surface scale — `spark-gateway` runs a darker scale with an indigo accent). shadcn's variable names (`--color-card`, `--color-primary`, …) must be **mapped onto** these tokens, never duplicated.
- **One accent color per app**, used for focus rings, selection and primary emphasis only. Status colors (ok/warn/danger) are for state, never decoration.
- Native controls must render dark: `color-scheme: dark` on `:root`.

## 3. Color

- Use semantic utilities (`bg-surface`, `text-muted-foreground`, `border-border`), never raw hex in components.
- **Contrast (WCAG AA):** body and muted text ≥ 4.5:1 against their surface; large text (≥ 18px, or bold ≥ 14px) ≥ 3:1. Muted/hint text is not exempt — check it (`#8a8a8a` = 5.5:1 on `#101010`).
- Status color never travels alone: pair it with a label or an icon.

## 4. Typography

- One font stack, defined once in `@theme` as `--font-sans`, system-first plus CJK fallbacks:

```css
--font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Inter', Roboto,
  'Helvetica Neue', 'PingFang HK', 'PingFang TC', 'Microsoft JhengHei',
  'Noto Sans CJK TC', sans-serif;
```

- Sizes via utilities: `text-xs` (12px) for hints and meta, `text-sm` (14px) for UI, `text-base` (16px) for card titles, and a page-title size of 26px with `tracking-tight` (add a `--text-page-title` token before reaching for an arbitrary value). Line-height 1.5 body, 1.1–1.2 headings.
- Monospace (`--font-mono`: `ui-monospace, 'SF Mono', Menlo, monospace`) **only** for data that is measured — timestamps, durations, tokens, IDs. Never as a stylistic choice.
- Numbers in tables and stat cards get `tabular-nums`.

## 5. Spacing & radius

- Spacing comes from the default 4px scale (`p-2`, `gap-3`, `space-y-6`). No arbitrary spacing values.
- Radius: `rounded-sm` 6px (small controls, segmented buttons), `rounded-md` 8px (buttons, inputs, selects), `rounded-lg` 12px (cards, dropzone), `rounded-xl` for overlays, `rounded-full` for pills and circular buttons.

## 6. Component composition

- **shadcn/ui primitives** live in `components/ui`. They may be edited, but only to bind tokens — never to introduce new colors or sizes.
- **Arc UI blocks** live in `components/arc` and keep their shipped CSS modules. Do not rewrite an Arc block into Tailwind classes; compose around it.
- Conditional classes go through `cn()` (`clsx` + `tailwind-merge`), never string concatenation:

```tsx
<div className={cn('rounded-md px-2 py-1', isActive && 'bg-surface-2 text-foreground')} />
```

- Variants use `cva`, not hand-assembled class strings:

```tsx
const button = cva('inline-flex items-center rounded-md', {
  variants: {
    variant: { primary: 'bg-accent text-white', ghost: 'text-muted-foreground hover:bg-surface-2' },
    size: { sm: 'h-8 px-2.5 text-xs', md: 'h-9 px-3 text-sm' },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
});
```

- Layout is plain utilities in JSX. That is the system, not a violation of it.
- Element and component names describe **role**, not color (`KeyTable`, `QuotaMeter`, not `BlueCard`).

## 7. States

Every interactive element defines all five, with variant prefixes:

1. **default**
2. `hover:` — visible but subtle
3. `focus-visible:` — a ring from the accent token (`focus-visible:ring-2 focus-visible:ring-accent focus-visible:outline-none`); never `outline-none` without a ring replacement
4. `active:` — press feedback
5. `disabled:` / `aria-disabled` — reduced opacity plus `cursor-not-allowed`

Component state carried on the element uses a data attribute, not a second class: `data-[state=open]:bg-surface-2`, `aria-invalid:border-danger`.

## 8. Motion

- Durations: 150ms for state changes, 200ms for entrance and exit. Nothing longer for UI.
- One easing token, `--ease-out`, applied via `ease-[var(--ease-out)]` or a component variant. No bare `ease`/`ease-in` on UI.
- Animate `transform` and `opacity` only; never animate layout properties.
- Respect reduced motion at the call site (`motion-safe:` / `motion-reduce:transition-none`) and keep the global override in `theme.css`:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## 9. Bans

- **No inline `style={{…}}`.** Two documented exceptions: a computed `width` for a progress meter, and a computed `transform` from a pointer gesture. Both carry a comment.
- **No arbitrary values** (`bg-[#171717]`, `text-[13px]`, `p-[7px]`). If a value is missing, add a token. Allowed exceptions: percentage widths in meters and accessibility floors such as `min-h-11`.
- **No `!important`** — including Tailwind's `!` prefix — except the reduced-motion override above.
- **No raw hex or `rgba()` outside `theme.css`.**
- **No magic z-index.** Use the documented scale: dropdown `z-20`, sticky header `z-30`, overlay `z-40`, modal `z-50`, toast `z-60`.
- **No `@apply`** outside the base layer in `theme.css`; compose in the component instead.
- **No nested cards.** Group with borders, dividers, or space.
- **No emoji or glyph icons.** Use `lucide-react` (or the shared SVG set in legacy apps).

## 10. Accessibility

- Focus-visible ring on every focusable element; keyboard users never see hover.
- Labels above inputs (`<Label htmlFor>`); never placeholder-as-label.
- Touch targets ≥ 44px (`min-h-11 min-w-11`) on touch surfaces.
- Errors: `role="alert"`, `aria-invalid` on the field, `aria-describedby` pointing at the message, message below the field.
- Icon-only buttons carry `aria-label`; decorative SVG gets `aria-hidden="true"`.
- Charts and meters expose their value as text or `aria-valuenow` — never colour alone.

## 11. Pre-merge checklist

- [ ] Tokens and utilities only — no raw hex, no arbitrary values without a documented exception.
- [ ] Conditional classes via `cn()`; variants via `cva`.
- [ ] Contrast passes (body ≥ 4.5:1, large ≥ 3:1).
- [ ] All five states defined; `focus-visible` ring present, no bare `outline-none`.
- [ ] `motion-reduce` respected for any animation.
- [ ] No inline `style` except a commented computed-value exception.
- [ ] Icons from `lucide-react`; icon-only buttons labelled.
- [ ] New values added to this document if the token set was extended.
