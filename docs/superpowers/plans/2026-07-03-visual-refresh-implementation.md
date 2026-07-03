# Post 220 Visual Refresh Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `index.html`'s inline `<style>` block (and add two small `<script>` additions) so the site's typography, color, spacing, and surface treatment read as a considered design system instead of a generic template, while every existing structural element, ID, and form integration stays byte-for-byte functionally identical.

**Architecture:** This is a single static HTML file with an inline `<style>` block and inline `<script>` block — no build step, no test framework. "Tests" in this plan are therefore: (a) deterministic `grep` checks that old values are fully gone and new tokens are used consistently, and (b) live browser verification via the Playwright MCP tools (`browser_navigate`, `browser_resize`, `browser_click`, `browser_take_screenshot`, `browser_console_messages`) since there is no automated test suite to run instead.

**Tech Stack:** Plain HTML/CSS/JS, Google Fonts (Lora + Inter, loaded via `<link>`), Formspree (unchanged), Google Calendar embed (unchanged).

## Global Constraints

- Single static `index.html` file, inline `<style>`/`<script>`, no build step — every task edits `index.html` directly.
- Never change the Formspree form `action` URLs: `https://formspree.io/f/meorlvkr` (rental) and `https://formspree.io/f/mwprnapd` (membership).
- Never change any form field `name` attributes (Formspree uses these as the submitted field labels).
- Never change element `id`s referenced by JavaScript: `#mainNav`, `#lightbox`, `#lightbox-img`, `#rentalForm`, `#membershipForm`.
- `--navy: #003366` is fixed and unchanged throughout.
- Every new color/spacing/radius/shadow value must be expressed via a CSS custom property defined in `:root` — no new one-off hardcoded hex values scattered through individual rules.
- Every `font-family` declaration must include a system fallback (e.g. `'Lora', Georgia, serif`) — never a Google-Fonts-only declaration, since fonts load over the network and must degrade gracefully.
- Any new motion must be guarded by `@media (prefers-reduced-motion: reduce)`.
- Spec reference: `docs/superpowers/specs/2026-07-03-visual-refresh-design.md`

---

### Task 1: Foundation — fonts, design tokens, accent color rename

**Files:**
- Modify: `index.html` (`<head>`, `:root`, `body`, and every usage of the old `--red` token)
- Test: N/A (no test framework) — verified via `grep` + a Playwright screenshot pass (steps below)

**Interfaces:**
- Produces: CSS custom properties available to all later tasks — `--accent` (#2E6B45), `--accent-hover` (#234F34), `--accent-light` (#6FBF8B, for accent-colored text on navy backgrounds), `--border-subtle` (#e8e6df), `--radius` (10px), `--shadow-sm`, `--shadow-md`. Google Fonts `Inter` (400/500/600) and `Lora` (600/700) loaded and available site-wide.

- [ ] **Step 1: Add Google Fonts `<link>` tags to `<head>`**

In `index.html`, find:

```html
    <link rel="apple-touch-icon" href="/favicon.svg">
    <style>
```

Replace with:

```html
    <link rel="apple-touch-icon" href="/favicon.svg">

    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Lora:wght@600;700&display=swap" rel="stylesheet">
    <style>
```

- [ ] **Step 2: Replace the `:root` token block**

Find:

```css
        :root {
            --navy: #003366;
            --red: #204F2E;
            --white: #FFFFFF;
            --gray: #F5F5F5;
            --dark-gray: #333333;
        }
```

Replace with:

```css
        :root {
            --navy: #003366;
            --accent: #2E6B45;
            --accent-hover: #234F34;
            --accent-light: #6FBF8B;
            --white: #FFFFFF;
            --gray: #F5F5F5;
            --dark-gray: #333333;
            --border-subtle: #e8e6df;
            --radius: 10px;
            --shadow-sm: 0 1px 3px rgba(0, 51, 102, 0.06);
            --shadow-md: 0 4px 16px rgba(0, 51, 102, 0.1);
        }
```

- [ ] **Step 3: Update the body font stack**

Find:

```css
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: var(--dark-gray);
        }
```

Replace with:

```css
        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
            line-height: 1.6;
            color: var(--dark-gray);
        }
```

- [ ] **Step 4: Rename every `var(--red)` usage to `var(--accent)`**

Run:

```bash
sed -i '' 's/var(--red)/var(--accent)/g' index.html
```

This rewrites every usage across the stylesheet (`.header-top`, `nav a:hover`, `.header-social a`, `.section-title`, `.btn`, `.meeting-banner`, `.meeting-item strong`, form focus border, `.form-required`, `.footer-section h4`, `.footer-section a:hover`, `.social-links a`, `.lightbox-close:hover`) **and** the one inline usage in the body markup (the "contact us" link color in the Events section).

- [ ] **Step 5: Replace hardcoded hover hex with the new hover token**

Run:

```bash
sed -i '' 's/#a01729/var(--accent-hover)/g' index.html
```

This affects `.header-social a:hover` and `.social-links a:hover` (2 occurrences).

- [ ] **Step 6: Fix footer text contrast — use the lighter accent variant on navy**

`--accent` (#2E6B45) on the navy (#003366) footer background computes to roughly 2:1 contrast, below WCAG AA's 4.5:1 minimum for text. Two footer rules sit directly on that navy background and need the lighter variant instead.

Find:

```css
        .footer-section h4 {
            margin-bottom: 1rem;
            color: var(--accent);
        }
```

Replace with:

```css
        .footer-section h4 {
            margin-bottom: 1rem;
            color: var(--accent-light);
        }
```

Find:

```css
        .footer-section a:hover {
            color: var(--accent);
        }
```

Replace with:

```css
        .footer-section a:hover {
            color: var(--accent-light);
        }
```

(`--accent-light` on navy computes to ~5.7:1 — passes AA.)

- [ ] **Step 7: Verify old tokens are fully gone**

Run:

```bash
grep -n 'var(--red)\|--red:\|#204F2E\|#a01729' index.html
```

Expected: no output (empty).

Run:

```bash
grep -c 'var(--accent-light)' index.html
```

Expected: `2`

- [ ] **Step 8: Visual smoke check**

Use the Playwright MCP tools:

```
browser_navigate to file:///Users/jnixon/templetonlegion.org/index.html
browser_take_screenshot (full page)
browser_console_messages
```

Confirm: page renders with no console errors, header/buttons/footer show the green accent (not crimson red, not the old flat `#204F2E`), body text renders in a sans-serif (Inter, or its fallback if the font hasn't loaded yet — either is acceptable at this stage since Task 2/3 haven't applied Lora to headings yet).

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
Add design tokens, load Inter/Lora fonts, rename --red to --accent

The --red variable held a forest green, not red -- a mismatch from
a prior color-scheme change. Renamed for clarity and refined the
shade slightly. Footer text uses a lighter accent variant since the
base accent fails WCAG AA contrast directly on the navy background.

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 2: Header & hero — typography and fluid sizing

**Files:**
- Modify: `index.html` (`.logo-section h1`, `.hero`, `.hero h2`, `.hero p`, and the mobile `.hero h2` override)
- Test: N/A — verified via Playwright screenshots at 3 widths (steps below)

**Interfaces:**
- Consumes: font families `Lora`/`Inter` loaded in `<head>` (from Task 1)
- Produces: no new tokens; this task only changes rules scoped to the header/hero region

- [ ] **Step 1: Apply Lora to the header wordmark**

Find:

```css
        .logo-section h1 {
            font-size: 1.8em;
            margin: 0;
        }
```

Replace with:

```css
        .logo-section h1 {
            font-family: 'Lora', Georgia, serif;
            font-weight: 600;
            font-size: 1.8em;
            margin: 0;
        }
```

- [ ] **Step 2: Make the hero fluid, warmer gradient, Lora heading**

Find:

```css
        /* Hero Section */
        .hero {
            background: linear-gradient(rgba(0, 51, 102, 0.7), rgba(0, 51, 102, 0.7)),
                        url('images/banner.jpeg');
            background-size: cover;
            background-position: center;
            color: var(--white);
            padding: 4rem 5%;
            padding-top: calc(4rem + 100px);
            margin-top: -100px;
            text-align: center;
            min-height: 400px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        .hero h2 {
            font-size: 2.5em;
            margin-bottom: 1rem;
        }

        .hero p {
            font-size: 1.2em;
            margin-bottom: 2rem;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
        }
```

Replace with:

```css
        /* Hero Section */
        .hero {
            background: linear-gradient(rgba(0, 51, 102, 0.72), rgba(0, 45, 90, 0.85)),
                        url('images/banner.jpeg');
            background-size: cover;
            background-position: center;
            color: var(--white);
            padding: clamp(2.5rem, 6vw, 4rem) 5%;
            padding-top: calc(clamp(2.5rem, 6vw, 4rem) + 100px);
            margin-top: -100px;
            text-align: center;
            min-height: 400px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        .hero h2 {
            font-family: 'Lora', Georgia, serif;
            font-weight: 700;
            font-size: clamp(1.8rem, 4vw + 1rem, 2.5em);
            margin-bottom: 1rem;
            letter-spacing: 0.2px;
        }

        .hero p {
            font-size: clamp(1rem, 1.5vw + 0.7rem, 1.2em);
            margin-bottom: 2rem;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            color: #eef2f6;
        }
```

- [ ] **Step 3: Remove the now-redundant mobile hero heading override**

The fixed mobile override would fight the fluid `clamp()` value above it. Find:

```css
            .mobile-menu-btn {
                display: block;
            }

            .hero h2 {
                font-size: 1.8em;
            }

            .cards {
```

Replace with:

```css
            .mobile-menu-btn {
                display: block;
            }

            .cards {
```

- [ ] **Step 4: Verify no leftover fixed hero override**

The `.hero h2` selector should now be defined exactly once (previously it appeared twice: once in the main rule, once in the mobile override you just deleted). Run:

```bash
grep -c '\.hero h2 {' index.html
```

Expected: `1`

- [ ] **Step 5: Visual check across breakpoints**

Use the Playwright MCP tools:

```
browser_navigate to file:///Users/jnixon/templetonlegion.org/index.html
browser_resize to 375x812 -> browser_take_screenshot
browser_resize to 768x1024 -> browser_take_screenshot
browser_resize to 1440x900 -> browser_take_screenshot
```

Confirm: hero heading and header wordmark render in a serif face (Lora), hero text scales smoothly between the three widths rather than jumping abruptly, hero background photo and gradient still show through.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
Apply Lora headings and fluid sizing to header and hero

Removes the fixed mobile hero font-size override now that clamp()
handles fluid scaling across all widths.

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 3: Section headings and content surfaces

**Files:**
- Modify: `index.html` (`.container`, `.section`, `.section-title`, `.card`/`.card:hover`/`.card h3`, `.meeting-banner`/`.meeting-banner h3`, `.calendar-container`, `.content-box`/`.content-box h3`)
- Test: N/A — verified via `grep` + Playwright screenshots

**Interfaces:**
- Consumes: `--accent`, `--border-subtle`, `--radius`, `--shadow-sm`, `--shadow-md`, `Lora`/`Inter` (from Task 1)
- Produces: no new tokens; unifies every card-like surface on the same border/shadow/radius treatment

- [ ] **Step 1: Fluid container/section spacing**

Find:

```css
        /* Main Content */
        .container {
            max-width: 1400px;
            margin: 0 auto;
            padding: 3rem 5%;
        }

        .section {
            margin-bottom: 4rem;
        }
```

Replace with:

```css
        /* Main Content */
        .container {
            max-width: 1400px;
            margin: 0 auto;
            padding: clamp(2rem, 4vw, 3rem) 5%;
        }

        .section {
            margin-bottom: clamp(2.5rem, 5vw, 4rem);
        }
```

- [ ] **Step 2: Section titles — Lora, fluid size**

Find:

```css
        .section-title {
            font-size: 2em;
            color: var(--navy);
            margin-bottom: 1.5rem;
            padding-bottom: 0.5rem;
            border-bottom: 3px solid var(--accent);
            display: inline-block;
        }
```

Replace with:

```css
        .section-title {
            font-family: 'Lora', Georgia, serif;
            font-weight: 600;
            font-size: clamp(1.5rem, 2vw + 1rem, 2em);
            color: var(--navy);
            margin-bottom: 1.5rem;
            padding-bottom: 0.5rem;
            border-bottom: 3px solid var(--accent);
            display: inline-block;
        }
```

- [ ] **Step 3: Cards — border + soft shadow + Lora heading**

Find:

```css
        .card {
            background: var(--white);
            padding: 2rem;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            transition: transform 0.3s, box-shadow 0.3s;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }

        .card h3 {
            color: var(--navy);
            margin-bottom: 1rem;
            font-size: 1.5em;
        }
```

Replace with:

```css
        .card {
            background: var(--white);
            padding: 2rem;
            border: 1px solid var(--border-subtle);
            border-radius: var(--radius);
            box-shadow: var(--shadow-sm);
            transition: transform 0.3s, box-shadow 0.3s;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: var(--shadow-md);
        }

        .card h3 {
            font-family: 'Lora', Georgia, serif;
            font-weight: 600;
            color: var(--navy);
            margin-bottom: 1rem;
            font-size: 1.5em;
        }
```

- [ ] **Step 4: Meeting banner — rounded corner + Lora heading**

Find:

```css
        .meeting-banner {
            background: var(--gray);
            padding: 2rem;
            border-left: 5px solid var(--accent);
            margin: 2rem 0;
        }

        .meeting-banner h3 {
            color: var(--navy);
            margin-bottom: 1rem;
        }
```

Replace with:

```css
        .meeting-banner {
            background: var(--gray);
            padding: 2rem;
            border-left: 5px solid var(--accent);
            border-radius: var(--radius);
            margin: 2rem 0;
        }

        .meeting-banner h3 {
            font-family: 'Lora', Georgia, serif;
            font-weight: 600;
            color: var(--navy);
            margin-bottom: 1rem;
        }
```

- [ ] **Step 5: Calendar container — border + soft shadow**

Find:

```css
        /* Calendar Styling */
        .calendar-container {
            margin: 2rem 0;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
```

Replace with:

```css
        /* Calendar Styling */
        .calendar-container {
            margin: 2rem 0;
            border: 1px solid var(--border-subtle);
            border-radius: var(--radius);
            overflow: hidden;
            box-shadow: var(--shadow-sm);
        }
```

- [ ] **Step 6: Content box — border + soft shadow + Lora heading**

Find:

```css
        /* Content Sections */
        .content-box {
            background: var(--white);
            padding: 2rem;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            margin-bottom: 2rem;
        }

        .content-box h3 {
            color: var(--navy);
            margin-bottom: 1rem;
        }
```

Replace with:

```css
        /* Content Sections */
        .content-box {
            background: var(--white);
            padding: 2rem;
            border: 1px solid var(--border-subtle);
            border-radius: var(--radius);
            box-shadow: var(--shadow-sm);
            margin-bottom: 2rem;
        }

        .content-box h3 {
            font-family: 'Lora', Georgia, serif;
            font-weight: 600;
            color: var(--navy);
            margin-bottom: 1rem;
        }
```

- [ ] **Step 7: Verify no stale flat shadows remain on these surfaces**

Run:

```bash
grep -n '0 2px 8px rgba(0,0,0,0.1)' index.html
```

Expected: no output.

- [ ] **Step 8: Visual check**

Use the Playwright MCP tools:

```
browser_navigate to file:///Users/jnixon/templetonlegion.org/index.html
browser_take_screenshot (full page)
```

Confirm: About/Events/What We Do/Hall/Membership/Contact sections all show the same bordered, softly-shadowed surface treatment; section titles render in Lora; card hover (via `browser_hover` on a `.card` element, then screenshot) still lifts and deepens its shadow.

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
Unify card/content-box/meeting-banner/calendar surfaces

Replaces the repeated flat box-shadow with a shared border + soft
shadow token pair, and applies Lora to section-level headings.

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 4: Scroll-reveal motion (optional polish, fully isolated)

**Files:**
- Modify: `index.html` (new CSS block before `/* Responsive Design */`, new JS function in the existing `<script>` block)
- Test: N/A — verified via `grep` + Playwright interaction check

**Interfaces:**
- Consumes: `.section` elements (already present in the DOM)
- Produces: `.is-visible` CSS class, `initializeScrollReveal()` JS function. Nothing later depends on this — it can be deleted entirely by removing this task's two additions without touching anything else.

- [ ] **Step 1: Add the reveal CSS, guarded for reduced motion**

Find:

```css
        /* Responsive Design */
        @media (max-width: 768px) {
```

Replace with:

```css
        /* Scroll reveal (motion) */
        .section {
            opacity: 0;
            transform: translateY(12px);
            transition: opacity 0.6s ease, transform 0.6s ease;
        }

        .section.is-visible {
            opacity: 1;
            transform: translateY(0);
        }

        @media (prefers-reduced-motion: reduce) {
            .section {
                opacity: 1;
                transform: none;
                transition: none;
            }
        }

        /* Responsive Design */
        @media (max-width: 768px) {
```

- [ ] **Step 2: Add the reveal JS**

Find:

```javascript
        // Initialize gallery on page load
        window.addEventListener('DOMContentLoaded', initializeGallery);
    </script>
```

Replace with:

```javascript
        // Initialize gallery on page load
        window.addEventListener('DOMContentLoaded', initializeGallery);

        // Reveal sections on scroll (skipped entirely for reduced-motion users)
        function initializeScrollReveal() {
            const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
            const sections = document.querySelectorAll('.section');

            if (prefersReducedMotion) {
                sections.forEach(section => section.classList.add('is-visible'));
                return;
            }

            const observer = new IntersectionObserver((entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        entry.target.classList.add('is-visible');
                        observer.unobserve(entry.target);
                    }
                });
            }, { threshold: 0.15 });

            sections.forEach(section => observer.observe(section));
        }

        window.addEventListener('DOMContentLoaded', initializeScrollReveal);
    </script>
```

- [ ] **Step 3: Verify both additions landed**

Run:

```bash
grep -c 'is-visible' index.html
```

Expected: `3` — one line in the CSS (`.section.is-visible {`) and two lines in the JS (the reduced-motion branch's `classList.add('is-visible')` and the observer callback's `classList.add('is-visible')`).

Run:

```bash
grep -n 'prefers-reduced-motion' index.html
```

Expected: 2 matches — one in the CSS `@media` guard, one in the JS `matchMedia` call.

- [ ] **Step 4: Interaction check**

Use the Playwright MCP tools:

```
browser_navigate to file:///Users/jnixon/templetonlegion.org/index.html
browser_console_messages
```

Confirm no console errors (a broken selector or typo here would throw on page load). Then scroll down (e.g. `browser_evaluate` with `window.scrollTo(0, 1500)` or use `browser_snapshot` and click a lower nav link like "Hall Rental") and take a screenshot — confirm the section below the fold is visible (not stuck at `opacity: 0`).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
Add reduced-motion-safe fade-in for sections on scroll

Isolated addition: one CSS block and one JS function, both easily
removable without touching anything else if it causes any issue.

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 5: Full regression pass — every existing feature still works

**Files:**
- None modified — this task only verifies Tasks 1–4 didn't break anything that already worked.
- Test: N/A — Playwright interaction pass + `grep` diff-safety checks

**Interfaces:**
- Consumes: the finished page from Tasks 1–4
- Produces: a pass/fail report; no code changes unless a regression is found, in which case fix it and re-run this task's checks before continuing

- [ ] **Step 1: Confirm nothing outside `index.html` changed unexpectedly**

Run:

```bash
git diff --stat 3ce6975..HEAD
```

Expected: only `index.html`, `.gitignore`, and the two `docs/superpowers/**` files from this plan/spec appear — no changes to `images/`, `favicon.svg`, or anything else.

- [ ] **Step 2: Confirm the two Formspree integrations are untouched**

Run:

```bash
grep -c 'formspree.io/f/meorlvkr' index.html
grep -c 'formspree.io/f/mwprnapd' index.html
grep -c 'name="' index.html
```

Expected: `1`, `1`, and `30` (9 meta-tag `name=` attributes in `<head>` + 11 rental-form fields + 10 membership-form fields, unchanged from the pre-refresh baseline).

- [ ] **Step 3: Mobile nav toggle**

Use the Playwright MCP tools:

```
browser_navigate to file:///Users/jnixon/templetonlegion.org/index.html
browser_resize to 375x812
browser_snapshot
browser_click on the "☰" mobile menu button
browser_take_screenshot
```

Confirm: nav links become visible in a vertical stack after the click.

- [ ] **Step 4: Lightbox gallery**

```
browser_resize to 1440x900
browser_click on the "Hall Rental" nav link
browser_click on the first hall photo thumbnail
browser_take_screenshot
browser_press_key Escape
browser_take_screenshot
```

Confirm: clicking a thumbnail opens the full-size lightbox overlay; Escape closes it.

- [ ] **Step 5: Form toggles**

```
browser_click on "Show Rental Form ▼"
browser_take_screenshot
browser_click on "Show Membership Form ▼" (scroll to Membership section first if needed)
browser_take_screenshot
```

Confirm: both forms expand when their toggle button is clicked, and all labeled fields (Full Name, Email, Phone, Preferred Date, Event Type, etc. for rental; First Name, Last Name, Email, Phone, Period of Service, Branch of Service, etc. for membership) are visible and legible with the new styling.

- [ ] **Step 6: Final full-page screenshots at three widths**

```
browser_resize to 375x812 -> browser_take_screenshot
browser_resize to 768x1024 -> browser_take_screenshot
browser_resize to 1440x900 -> browser_take_screenshot
```

Confirm against the approved mockup direction: Lora/Inter typography, refined green accent (no crimson, no flat old green), bordered soft-shadow cards throughout, fluid hero/section sizing with no abrupt jumps between the three widths, footer heading/link-hover text legible against the navy background.

- [ ] **Step 7: Report**

No commit for this task (verification only). If every check above passed, the visual refresh is complete. If anything failed, fix it as part of the task whose region it belongs to (Task 1 for tokens/fonts, Task 2 for header/hero, Task 3 for section surfaces, Task 4 for motion), re-commit that task, and re-run this Task 5 checklist from Step 1.
