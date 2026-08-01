# Frontend Design Knowledge Base

Read this file before building UI, dashboards, landing pages, or any user-facing React/web interface. All guidance is here.

Frontend is one of the biggest weaknesses of cheaper models. They often:

* make everything look similar,
* have weak visual hierarchy,
* poor spacing,
* mediocre UX,
* generic dashboards,
* inconsistent component composition.

This knowledge base is separate from backend/systems engineering. It teaches **visual judgment and interface composition**, not only component APIs. Screenshots and real product references often teach more than raw React code — a design system is taste under constraints.

**How to use this file:** prefer principles + patterns over copy-paste. Study the cited products and libraries for spacing, hierarchy, typography, and interaction — do not clone their brand.

---

# SHADCN_UI.md

Copy-paste components you own — the modern default for serious React UI.

## The Core Identity

shadcn/ui is not an npm component monolith. You **generate/copy source into your repo** (built on Radix primitives + Tailwind), so you own the code, the tokens, and the variants. That invert of "dependency you can't surgically edit" is the point.

```
  Radix primitive (a11y + behavior)
       +
  Tailwind tokens / variants
       +
  your repo's components/ui/*
```

## What to Steal

- Composition patterns: `Button` variants, `Dialog` structure, form field wiring.
- Consistent reliance on **accessible primitives** underneath styled shells.
- A single source of design tokens (CSS variables) driving light/dark.

## What to Avoid

- Dropping in every block until the app looks like the shadcn docs site.
- Treating the registry as a brand — it's a starting kit; differentiate.

## What to Carry Away

1. **Own your components** when you need design control; wrap primitives, don't only theme from outside.
2. **Behavior and appearance separate** — Radix for behavior, your tokens for look.
3. **Variants as a system** (CVA-style) beat one-off class strings.
4. **Fewer well-tuned components** beat a kitchen-sink UI kit.
5. **Docs sites are not your product** — restyle aggressively.

---

# MAGIC_UI.md

Motion-forward marketing components — use sparingly and with intent.

## The Core Identity

Magic UI (and similar registries) ship polished, animated building blocks for landings and feature sections. They teach **presence**: staggered reveals, text effects, beam/border accents, marquee social proof.

## When They Help

- Hero moments, feature showcases, launch pages where delight is the job.
- Learning timing curves and staggered children from working examples.

## When They Hurt

- App chrome (settings, tables, forms) with constant motion noise.
- Stacking three showpiece effects until hierarchy collapses.

## What to Carry Away

1. **Motion is hierarchy** — animate the one thing that matters.
2. **Marketing ≠ product UI** — borrow craft, not constant spectacle.
3. **Performance budgets** — watch CLS, main-thread cost, reduced-motion.
4. **One signature effect** beats five generic ones.
5. **Study structure under the effect** — layout still does the heavy lifting.

---

# ACETERNITY_UI.md

High-craft React UI patterns for distinctive marketing surfaces.

## The Core Identity

Aceternity UI popularized a wave of distinctive, Tailwind + Framer Motion compositions (spotlight cards, bento grids, tracing beams). The lesson is **composition originality**: backgrounds, masks, and micro-layout tricks that escape the purple-gradient AI default.

## What to Carry Away

1. **Backgrounds and masks create atmosphere** without extra cards.
2. **Bento layouts** organize features without a wall of identical cards.
3. **Restraint** — one Aceternity-grade section per page is often enough.
4. **Steal geometry, not identity** — re-token colors/type to your brand.
5. **Always check mobile** — fancy absolute layouts break first on small screens.

---

# MOTION.md

Tasteful animation with Motion (formerly Framer Motion).

## The Core Identity

Motion gives React a declarative animation model: `animate`, layout transitions, `AnimatePresence` for enter/exit, gestures, and scroll-linked stories. Models fail at animation by either skipping it or animating everything.

```
  presence  -->  AnimatePresence
  layout    -->  layout / layoutId
  feedback  -->  whileHover / whileTap
  story     -->  staggered children / scroll
```

## Principles

- **Duration short** (150–300ms UI chrome; longer only for narrative).
- **Easing with intent** — enter slightly decelerate; exits can be quicker.
- **Honor `prefers-reduced-motion`.**
- **Layout animations** for reordering; don't fake with opacity alone when position changes.

## What to Carry Away

1. **Animate state change, not decoration.**
2. **Exit animations need presence management** — unmount timing matters.
3. **Shared layoutIds** create continuity across routes/modals.
4. **Stagger sparingly** — lists of 3–7, not 40.
5. **Reduced motion is a feature**, not an afterthought.

---

# REFACTORING_UI.md

Design principles for developers (Refactoring UI concepts).

## The Core Identity

Refactoring UI teaches non-designers to make interfaces look deliberate: hierarchy, spacing scales, limited type sizes, empty space, and avoiding "gray on gray on gray." You don't need the book as a dependency — you need the **checklist**.

## High-Leverage Rules

- Start with too much white space; tighten until it feels aligned — not the reverse.
- Establish a **type scale** (e.g. 3–5 sizes) and stop inventing font sizes.
- Establish a **spacing scale** (4/8-based) and stop using magic numbers.
- One accent color for actions; neutrals do the rest.
- De-emphasize secondary text (size, weight, color) instead of shouting with red/blue everywhere.
- Borders and shadows are hierarchy tools — if everything is elevated, nothing is.

## What to Carry Away

1. **Hierarchy first** — if you squint, can you still see primary vs secondary?
2. **Scales beat vibes** — type and space on a system.
3. **Neutrals carry UI** — accent is for action and status.
4. **Empty space is structure**, not waste.
5. **Finish by removing**, not adding.

---

# RADIX_UI.md

Accessible primitives without imposing visuals.

## The Core Identity

Radix UI Primitives provide **unstyled, accessible** components: dialogs, menus, popovers, tabs, toast, etc. Focus traps, keyboard roving tabindex, aria wiring — solved once.

## What to Carry Away

1. **Don't hand-roll modal focus** — use primitives.
2. **Styling belongs to you** — primitives are behavior.
3. **Composition API** (`Root`/`Trigger`/`Content`) scales better than mega-props.
4. **Portal + positioning** are part of the primitive's job.
5. **Accessibility is the floor** for any design system.

---

# TREMOR.md

Dashboard-oriented React components on Tailwind.

## The Core Identity

Tremor focuses on **metrics UI**: KPI cards, charts, trackers, input blocks tuned for admin/analytics. It teaches density, numeric typography, and chart+legend composition for data products.

## What to Carry Away

1. **Dashboards need a data hierarchy** — KPI → trend → table, not 12 equal cards.
2. **Number formatting is UX** — units, deltas, sparklines.
3. **Chart defaults matter** — axes, grids, colorblind-safe series.
4. **Density ≠ clutter** if alignment and grouping are clean.
5. **Don't use marketing hero patterns inside app shells.**

---

# REACT_BITS.md

Small animated building blocks for React.

## The Core Identity

React Bits-style libraries showcase isolated motion/visual patterns (text animators, backgrounds, cursors). Use as a **catalog of micro-techniques**, then reimplement in your tokens.

## What to Carry Away

1. **Isolate the idea** — re-skin before shipping.
2. **Microinteractions teach timing** better than theory alone.
3. **Avoid novelty traps** in productivity UI.
4. **Compose with your design system**, don't parallel it.
5. **Delete anything that doesn't support a user goal.**

---

# TAXONOMY.md

Next.js dashboard taxonomy / reference app structure.

## The Core Identity

Taxonomy-style reference projects (shadcn-era Next.js examples) teach **app information architecture**: marketing pages + auth + dashboard + settings + content sections with consistent nav, tables, and forms.

## What to Carry Away

1. **IA before chrome** — nav groups match mental models.
2. **Settings patterns** — sections, danger zones, save affordances.
3. **Table + filter + empty state** as a triple, not an afterthought.
4. **Auth-gated shells** differ from marketing layouts — separate roots.
5. **Reference apps are curricula** — copy structure, not aesthetics blindly.

---

# REACT_ARIA.md

Behavior-first accessibility from Adobe React Aria / Aria Components.

## The Core Identity

React Aria separates **hooks/behavior** from rendering more aggressively than many kits. It encodes keyboard interactions, i18n-aware formatting, and advanced patterns (collections, date pickers, grids).

## What to Carry Away

1. **Internationalization is part of UI behavior** (focus, selection, formatting).
2. **Headless hooks** let design systems stay branded.
3. **Complex widgets are research problems** — don't invent date picker a11y.
4. **Collections virtualization** patterns matter at scale.
5. **Pair with your visuals** — Aria is not a look.

---

# DESIGN_SYSTEMS.md

How professionals build reusable interface systems.

## Sources to Study

1. shadcn/ui
2. Magic UI
3. Aceternity UI
4. Origin UI
5. HeroUI (formerly NextUI)
6. Radix UI / React Aria / MUI / Chakra / Mantine (broader ecosystem)

## System Layers

```
  tokens (color, space, type, radius, shadow)
    -->
  primitives (button, input, checkbox)
    -->
  patterns (form field, page header, data table)
    -->
  features (billing settings, onboarding step)
```

Rules: tokens never hardcode in features; patterns own spacing rhythm; features compose patterns only.

## What to Carry Away

1. **Tokens first** — without them, "consistency" is wishful.
2. **Patterns > orphan components.**
3. **Document variants and states** (hover/focus/disabled/loading/error).
4. **One radius / shadow language** across the product.
5. **Resist new one-off components** — extend the system.

---

# TYPOGRAPHY.md

Type is hierarchy.

## Rules

- Prefer expressive, purposeful fonts; avoid defaulting to Inter/Roboto/Arial/system for branded surfaces (product apps may stay neutral — be intentional).
- Limit weights (regular/medium/semibold) and sizes (a small scale).
- Line length ~45–75 characters for reading text.
- Pair display + body carefully; don't use display fonts for dense data.
- Tabular nums for dashboards.

## What to Carry Away

1. **Size + weight + color** encode hierarchy together.
2. **Fewer sizes than you think.**
3. **Body readability beats brand hero on app screens.**
4. **Match type to surface** — marketing vs app vs code.
5. **Never randomize letter-spacing** without a system.

---

# SPACING.md

Spacing is structure.

## Rules

- Use a spacing scale (4px foundation is common).
- Consistent padding inside components; consistent gaps in stacks.
- Related items closer; unrelated groups farther (proximity principle).
- Align to a mental grid; avoid 13px / 27px orphans.
- On mobile, increase tap targets before shrinking all gaps blindly.

## What to Carry Away

1. **If it looks "off," check spacing before color.**
2. **Stacks and clusters** beat ad-hoc margins.
3. **Section padding** creates page rhythm.
4. **Dense UI still needs grouping.**
5. **Don't nest cards inside cards to fake structure** — use space.

---

# ANIMATIONS.md

Tasteful motion sources: Motion, React Bits, Animate UI, Motion Primitives, GSAP examples.

## Rules

- Motion supports meaning: feedback, continuity, attention.
- Prefer transform/opacity; avoid layout thrash.
- Interruptible animations; never block input without need.
- GSAP shines for timeline-heavy marketing; Motion shines for React app state.

## What to Carry Away

1. **One job per animation.**
2. **Fast feedback, slower storytelling.**
3. **Shared element transitions** glue navigations.
4. **Test reduced motion.**
5. **Delete motion that doesn't teach or confirm.**

---

# DASHBOARDS.md

Study: Tremor, Taxonomy, Refine, Cal.com, Supabase Dashboard.

## Composition Pattern

```
  Page header (title + primary action)
  KPI row (3–5 max)
  Main chart / calendar / feed
  Secondary table or lists
```

Avoid: six identical KPI cards, charts without context, rainbow categorical colors, every widget in a heavy card chrome.

## What to Carry Away

1. **One primary question per screen.**
2. **KPIs need comparison** (vs yesterday / target).
3. **Tables need empty/loading/error states.**
4. **Filters are part of the design**, not bolted on.
5. **App shell consistency** (nav width, header height) builds trust.

---

# LANDING_PAGES.md

Study composition: Vercel, Linear, Raycast, Stripe, Resend.

## Hero Budget

First viewport usually:

* brand
* one headline
* one short supporting sentence
* one CTA group
* one dominant visual plane

Avoid: stat strips, logo salads, multiple competing CTAs, inset collage heroes, floating badge clutter on the hero image.

## What to Carry Away

1. **One composition, not a dashboard.**
2. **Brand must survive removing the nav.**
3. **Full-bleed visual plane** beats tiled cards in the hero.
4. **Sections have one job each.**
5. **Ship 2–3 intentional motions**, not noise.

---

# MOBILE_UI.md

Study: Mobbin (patterns), platform HIG/Material, responsive landings.

## Rules

- Thumb zones and 44px+ targets.
- Bottom nav vs hamburger — know the job.
- Don't shrink a desktop table; reflow to cards/lists.
- Sticky CTAs carefully — don't cover content.
- Respect safe areas.

## What to Carry Away

1. **Design mobile as its own composition.**
2. **Prioritize tasks**, not pixel parity.
3. **Gesture conflicts** (swipe vs scroll) need care.
4. **Type scales up slightly** for reading on phones.
5. **Test on a real device width**, not only DevTools.

---

# ACCESSIBILITY.md

Sources: Apple HIG, Material Design 3, Radix/React Aria, WAI-ARIA patterns.

## Floor Requirements

- Keyboard path for all actions.
- Visible focus.
- Sufficient contrast.
- Labels for inputs; don't rely on placeholder alone.
- Respect reduced motion; don't convey info by color alone.

## What to Carry Away

1. **A11y is design quality**, not compliance theater.
2. **Primitives carry you far** — use them.
3. **Announce dynamic changes** (live regions) when needed.
4. **Test with keyboard only** before shipping.
5. **Accessible ≠ ugly** — constraints sharpen design.

---

# FORMS.md

Sources: React Hook Form, TanStack Form, Zod, Conform, Vest.

## Pattern

```
  schema (Zod) --> form state --> field UI --> async submit --> server errors
```

- Labels, helper text, inline errors aligned.
- Don't block typing with premature errors; validate on blur/submit thoughtfully.
- Preserve input on failure; map server errors to fields.
- Dangerous actions need confirmation patterns.

## What to Carry Away

1. **Schema as source of truth.**
2. **Field layout is a pattern** — label/control/error alignment.
3. **Disable submit sparingly** — explain why when you do.
4. **Optimistic UI only when rollback is safe.**
5. **Multi-step forms show progress and allow back.**

---

# CHARTS.md

Sources: Tremor, Recharts, Nivo, ECharts, Observable.

## Rules

- Start with the question; pick chart type second.
- Label axes; include units; prefer direct labels over huge legends when possible.
- Avoid 3D and chartjunk.
- Use OKLCH/HSL-tuned palettes; test colorblind safety.
- Empty and loading states for charts, not blank boxes.

## What to Carry Away

1. **Clarity over decoration.**
2. **Time series vs categorical** need different defaults.
3. **Tooltip + annotation** explain spikes.
4. **Observable notebooks** teach analytical craft.
5. **Fewer series** — split charts before spaghetti.

---

# NAVIGATION.md

## Rules

- Information architecture before component choice.
- Active states obvious; groups labeled.
- Don't mix 5 nav paradigms on one shell.
- Breadcrumbs for depth; tabs for peer sections.
- Command palette (Linear/Raycast-like) for power users — optional enhancer, not the only path.

## What to Carry Away

1. **Nav is product strategy made visible.**
2. **Deep-link everything meaningful.**
3. **Mobile nav is a redesign**, not a collapse.
4. **Settings belong in a predictable place.**
5. **Escape hatches** (search/command) reduce nav bloat.

---

# DARK_MODE.md

## Rules

- Design with tokens (`background`, `foreground`, `muted`, `border`, `accent`) — not inverted hex hacks.
- Dark mode is not "pure black + neon" by default; control contrast.
- Elevation via luminance steps more than heavy shadows.
- Images/illustrations need dark-aware variants.
- Honor system preference with manual override.

## What to Carry Away

1. **Tokens make dark mode possible.**
2. **Test charts and statuses** on both themes.
3. **Borders + spacing** replace shadows carefully.
4. **Brand accent may need retuning** for dark surfaces.
5. **Avoid pure #000 unless intentional.**

---

# GLASSMORPHISM.md

## Rules

- Glass is atmosphere, not a layout strategy.
- Need real backdrop content; over blur tanks performance/readability.
- Keep text contrast legal; frosted panes fail often.
- Use on heroes/modals sparingly; avoid glass tables of data.

## What to Carry Away

1. **One glass surface per view** is usually enough.
2. **Fallback solids** for reduced performance/a11y.
3. **Borders + soft highlight** sell the material more than max blur.
4. **Don't put dense forms on glass.**
5. **If hierarchy weakens, remove glass.**

---

# GRADIENTS.md

## Rules

- Gradients for atmosphere and brand — not for body text.
- Prefer subtle meshes / two-stop soft gradients over rainbow streaks.
- Avoid the cliché purple-to-indigo AI look unless it *is* the brand.
- CSS variables for gradient stops; keep them themeable.
- Pair with photography or real product UI for landings when possible — abstract gradient alone is a weak visual anchor.

## What to Carry Away

1. **Restraint reads premium.**
2. **Gradients support, they don't replace content.**
3. **Check banding and contrast on text overlays.**
4. **Animate gradients slowly or not at all.**
5. **Match landing gradient language to product chrome.**

---

# COMPONENT_PATTERNS.md

## High-Value Patterns

- Page header: title, description, actions
- Empty state: explanation + CTA
- Data table: filters, sort, row actions, bulk actions
- Confirm destructive dialog
- Toast / inline alert for feedback
- Skeleton loaders that match layout

## Composition Rule

If removing a border/shadow/radius doesn't hurt interaction or understanding, it shouldn't be a card.

## What to Carry Away

1. **Patterns beat pages made of freeform divs.**
2. **States are the work** (empty/loading/error/success).
3. **Cards are for interaction containers**, not decoration.
4. **Keep action placement predictable.**
5. **Document pattern usage in the design system.**

---

# COLOR_THEORY.md

Sources: Tailwind Colors, Radix Colors, Open Color, Material Color, OKLCH examples.

## Rules

- Neutrals first; semantic colors for success/warn/danger/info.
- OKLCH helps even perceptual lightness across hues.
- Radix-style scales (steps 1–12) teach hover/active/solid/contrast roles.
- Don't use color alone for state.
- Limit accent hues.

## What to Carry Away

1. **Scales > loose hex picks.**
2. **Semantic tokens** (`danger`) over raw `red-500` in features.
3. **Test contrast** early.
4. **Charts need a separate categorical palette.**
5. **Brand color ≠ action color** always — define roles.

---

# VISUAL_HIERARCHY.md

## Squint Test

Blur your eyes: you should still see primary action / headline / clustering.

## Tools

- Size, weight, contrast, proximity, alignment, whitespace
- Not: more borders, more colors, more badges

## What to Carry Away

1. **One focal point per section.**
2. **Secondary content recedes.**
3. **Alignment creates calm.**
4. **Badges/chips inflate noise** — budget them.
5. **Hierarchy is design debt when everything is bold.**

---

# MICROINTERACTIONS.md

## Examples

- Button press depth / color shift
- Successful save check
- Copy-to-clipboard confirmation
- Pull-to-refresh affordance
- Subtle list reorder

## Rules

- Confirm user actions that otherwise feel uncertain.
- Keep under ~200ms for tool-like feedback.
- Don't animate decorative chrome on every hover in dense UIs.

## What to Carry Away

1. **Feedback > flair.**
2. **Same interaction, same response** across the app.
3. **Optimistic only with clear failure recovery.**
4. **Sound/haptics rare on web** — visual is enough.
5. **Micro ≠ micro-clutter.**

---

# CREATIVITY_SOURCES.md

Scrape/study (do not copy wholesale):

* Awwwards
* Godly
* Land-book
* Mobbin
* Lapa Ninja

Learn: spacing, hierarchy, typography, composition, layouts, interaction patterns.

## Screenshot-First Learning

For frontend, **screenshots may be more valuable than code**. Curate examples of dashboards, forms, pricing pages, onboarding, mobile screens, empty states — pair each with a one-paragraph "why it works." That trains judgment raw React often doesn't capture.

## What to Carry Away

1. **Build a personal reference library.**
2. **Annotate principles**, don't hoard links.
3. **Reimplement in your tokens** within 24 hours of studying.
4. **Separate marketing craft from app craft.**
5. **Taste is trained**, not downloaded.

---

# SYNTHESIS.md (Frontend)

## The Common Failure Mode of Cheap Models

Same Inter UI, purple gradient, three feature cards, generic dashboard, inconsistent spacing. This file exists to break that attractor basin.

## Frontend Context Engineering Checklist

1. **Tokens** — color/space/type/radius defined?
2. **Hierarchy** — squint test pass?
3. **Composition** — one job per section; hero budget respected on landings?
4. **States** — empty/loading/error designed?
5. **A11y** — keyboard, focus, contrast, reduced motion?
6. **Motion** — only where it clarifies?
7. **References** — studied a real product (Linear/Stripe/Supabase/Cal) before inventing?
8. **Own the components** — primitives + your brand, not a docs clone?

## Proposed Library Shape (for a larger KB)

```
Frontend/
  Design Systems/
  Typography/
  Spacing/
  Animations/
  Dashboards/
  Landing Pages/
  Mobile UI/
  Accessibility/
  Forms/
  Charts/
  Navigation/
  Dark Mode/
  Glassmorphism/
  Gradients/
  Component Patterns/
  Color Theory/
  Visual Hierarchy/
  Microinteractions/
```

## Priority Repositories

| Rank | Repo |
|---|---|
| ★★★★★ | shadcn/ui |
| ★★★★★ | Magic UI |
| ★★★★★ | Aceternity UI |
| ★★★★★ | Motion |
| ★★★★★ | Refactoring UI (concepts) |
| ★★★★☆ | Radix UI |
| ★★★★☆ | Tremor |
| ★★★★☆ | React Bits |
| ★★★★☆ | Taxonomy |
| ★★★★☆ | React Aria |

## The One-Sentence Version

Great frontend context teaches **judgment under constraints** — hierarchy, space, type, state, and motion — so models compose interfaces like products, not like generic component demos.
