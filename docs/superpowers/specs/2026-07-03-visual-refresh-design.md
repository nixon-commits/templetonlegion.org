# Visual Refresh Design — Post 220 Site

## Goal

General polish/modernization of `index.html`'s "overall look and feel." The site's structure, content, and functionality (nav, lightbox gallery, rental/membership forms) already work well and are out of scope. This is a CSS-first visual design pass, staying within a formal/patriotic register appropriate for a veterans' organization.

## Non-goals

- No changes to page structure, section order, or content copy.
- No changes to the rental (`https://formspree.io/f/meorlvkr`) or membership (`https://formspree.io/f/mwprnapd`) form fields, `name` attributes, or submission behavior.
- No new build tooling — stays a single static `index.html` with inline `<style>`, matching the current setup.
- No IA rework, no hero carousel, no multi-step forms (these were considered and explicitly deferred as a larger, separate effort).

## Decisions (validated via visual mockups)

**Typography** — "Classic Americana" pairing:
- Headings: **Lora** (weight 600 for section titles, 700 for hero `h2`), serif fallback stack `Lora, Georgia, serif`
- Body: **Inter** (400 body copy, 500/600 for labels/emphasis), fallback stack `Inter, -apple-system, BlinkMacSystemFont, sans-serif`
- Loaded via Google Fonts `<link>` in `<head>` (preconnect + stylesheet, `display=swap`), replacing the current system font stack (`'Segoe UI', Tahoma, Geneva, Verdana, sans-serif`)

**Color** — refined green accent, navy unchanged:
- `--navy: #003366` (unchanged — matches logo)
- Accent color corrected from the misleadingly-named `--red: #204F2E` to a properly-named token with a slightly richer shade: `--accent: #2E6B45` (with a darker hover shade, e.g. `#234F34`, replacing the current ad-hoc `#a01729` hover value which doesn't relate to green at all)
- All current usages of `var(--red)` (buttons, hover states, dividers, footer link hovers, social icons, form focus borders, meeting-banner border) get renamed to `var(--accent)` and repainted with the new shade — no crimson red anywhere; this was deliberately moved away from in a prior commit and stays that way
- Traditional Legion crimson (`#C41E3A`) and a brass/gold third accent were both considered and rejected in favor of keeping the site's established green identity

**Spacing & type scale** — fluid via `clamp()`:
- Hero heading, hero padding, section-title size, and container padding move from fixed `rem`/`em`/breakpoint jumps to `clamp(min, preferred-vw, max)` so they scale smoothly between mobile and desktop instead of snapping at the 768px breakpoint
- Existing `@media (max-width: 768px)` block stays for structural changes (nav collapse, card grid to single column, form-row to single column) — `clamp()` replaces it only for pure sizing/spacing values

**Surfaces (cards, content-box, meeting-banner, calendar-container)**:
- Replace the current flat `box-shadow: 0 2px 8px rgba(0,0,0,0.1)` treatment with a softer combination: a thin `1px solid` neutral border (`#e8e6df`) + a subtle navy-tinted shadow (`0 1px 3px rgba(0,51,102,0.06)`), consistent across all card-like containers
- Border-radius increases slightly and becomes a shared token (`--radius: 10px`) instead of the current mix of `4px`/`8px` values

**Hero**:
- Same background image (`banner.jpeg`) and same structural approach (gradient overlay + sticky-header negative-margin trick), gradient adjusted slightly for a touch more depth (`rgba(0,51,102,0.72)` to `rgba(0,45,90,0.85)`) rather than a flat single-tone overlay
- Sizing becomes fluid per the spacing scale above

**Motion (small, optional, reduced-motion-safe)**:
- A light fade-in-on-scroll for `.section` elements using `IntersectionObserver` (opacity 0→1, translateY 10px→0)
- Guarded by `@media (prefers-reduced-motion: reduce)` — users with that preference see no animation, content is simply visible
- This is additive and isolated: if it causes any issue, it can be deleted entirely without affecting anything else in this spec

## What stays exactly as-is

- All HTML structure, element IDs, and class names used by JavaScript (`#mainNav`, `.form-container.active`, `#lightbox`, etc.)
- All existing JavaScript behavior (mobile nav toggle, lightbox open/close/keyboard nav, form show/hide toggles)
- Both Formspree form `action` URLs and all form field `name` attributes
- Google Calendar embed
- Image assets and their locations

## Implementation notes

- This is a rewrite of the `<style>` block in `index.html` plus the two `<link>` tags for Google Fonts in `<head>`. No other files change except possibly renaming `TAL-brand-secondary-1C-white.png` usage if contrast needs revisiting (not expected).
- Verify color contrast of `--accent` (`#2E6B45`) against white button text and against the navy footer background meets WCAG AA (4.5:1) before finalizing the exact hex — the mockup value is a starting point, not locked.
- Since this is a single static file with no build step, test by opening `index.html` directly in a browser at a few widths (375px, 768px, 1440px) plus a quick pass with `prefers-reduced-motion: reduce` toggled in devtools.
