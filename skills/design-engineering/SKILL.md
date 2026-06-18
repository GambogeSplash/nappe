---
name: design-engineering
description: Use when building, reviewing, or refining any UI, frontend, component, animation, layout, or visual design. Provides craft standards for motion, micro-interactions, interaction states, spacing, typography, color and contrast, accessibility, and performance, plus a process for avoiding generic AI-default design and verifying work by rendering and comparing.
---

# Design Engineering Standard

Hold front-end and design work to the level of a senior design engineer. This teaches judgment, not a single house style: a house style copied everywhere is just a new kind of generic. Use the principles to make decisions and the numbers as real targets.

When producing copy, avoid em dashes and dash-joined clauses in sentences (recast with commas, colons, or parentheses), and avoid decorative sparkle icons. Both read as generic AI output.

---

## How to operate

These decide whether the work is good before any pixel is placed.

**Lead with the frame, not the surface.** Before designing, state the mental model in one line: who is this for, what are they trying to do, what is the one thing this screen is about. A clear frame makes every later decision obvious. Present rationale as the frame ("we treat the pending state as a promise, not a fact"), not as a description of what it looks like.

**Kill the generic default.** Left alone, models converge on the same look: a SaaS sans like Inter or Roboto, a purple-to-blue gradient on white, evenly rounded cards, a centered hero, a sparkle icon on the AI button. This is the tell of unconsidered work. Treat the first idea that arrives as the average idea and push past it. Pick type, color, and layout that mean something for this specific product.

**Never take the lazy path.** When cloning or matching a reference, observe the real behavior before building: scroll it, hover it, click it, watch the transitions. Pull real assets instead of synthesizing substitutes silently. Measure real values (durations, colors, spacing) instead of eyeballing them. Interactions are part of the design, not polish added later.

**Verify with your own eyes.** This is the single most important habit. After building, render the result, take a screenshot, and compare it against the intent or the reference. List what is wrong, then fix it. Repeat until it matches. "Looks done in the code" is not a stop signal; "I looked at it rendered and it is right" is. Check the real interaction state (hovered, focused, loading, error, empty), not just the idle one.

**Design for the first five minutes.** Always check what the screen looks like for a brand-new user with one item and zero history. If half the affordances are dropdowns of things they do not have yet, the design is broken. Empty states are a primary design surface, not an afterthought.

**No visible-but-dead controls.** A button that opens nothing, a dropdown of fake choices: worse than absent. Either wire it to something real or hide it until it works.

**Restraint when it is noise.** When a surface starts to feel like information overload, cut and consolidate rather than expand. Five things done well beat twenty-five done shallowly. A visual and its hover state make an inline legend redundant; remove the legend.

**Finish the sweep.** If you name six follow-on edits, ship all six or say explicitly which you are deferring and why. Do not leave the work in a half-state and call it done.

**Be honest about quality.** When asked how good something is, rate it against a senior bar and name the failure modes plainly. Soft-pedaling hides the problems that matter. Flag your own redundancy and broken data before anyone else has to.

---

## Avoiding generic design

The goal is a considered, specific look reached by reasoning, not by reaching for a default.

**Typography carries most of the signal.** High contrast between two type choices reads as intentional: a display serif with a clean grotesque, a geometric sans with a monospace for labels. Set body text at 16px or larger (long-form 18 to 20px), line-height around 1.5, and line length between 45 and 75 characters. Do not default to Inter, Roboto, Open Sans, or the system stack out of habit; choose a typeface that fits the product's character, and do not over-pick the same trendy alternative every time either. The skill is matching type to product, not memorizing a font.

**Commit to a real palette.** Build 8 to 10 steps per color so you always have the exact value you need. Choose colors that say something about the product rather than the default blue. Use color with intent: one accent doing real work beats six competing for attention.

**Let visual weight map to truth.** Things that are real, confirmed, or primary get strong, saturated, solid treatment. Things that are pending, uncertain, or secondary get a quieter treatment (lower contrast, a hatch or tint, less weight). Do not weaken the confident state to make it elegant; weaken the uncertain one. The interface should never imply a number or state the underlying data contradicts.

**Decide on depth deliberately.** Flat and layered are both valid; an accidental mix is not. If flat, get hierarchy from spacing, weight, and border contrast. If using elevation, do it well (see "Shadows"). Avoid the unconsidered middle: a single weak grey drop shadow on everything, gloss, and fake 3D bevels read as cheap.

**Cohesion over genre cliché.** Keep one coherent visual language across the app. Do not bolt a dark "terminal zone" onto an otherwise light product because the genre usually looks that way. If a section needs to feel different, get there with density and accent, not a competing theme.

**No decorative accents.** Skip colored left-border stripes to mark "mine," borders on cards purely for decoration, and sparkle icons as garnish. Borders that carry real meaning (a destructive action, a focus ring, a selected state) stay.

---

## Motion and animation

Motion should feel alive and purposeful, never like a loading spinner standing in for thought. Six tests for any animation: natural, fast, purposeful, runs at 60fps, interruptible, and accessible.

**Duration.** Most UI motion lives between 150ms and 400ms. Treat 300ms as a ceiling for common transitions, not a target. Anything past 500ms feels like a drag.

| Interaction | Duration |
|---|---|
| Instant feedback (toggle, checkbox, hover color) | ~100ms or none |
| Buttons, small controls, state changes | 150 to 200ms |
| Dropdowns, popovers, tooltips | 150 to 250ms |
| Modals, drawers, sheets, page-level | 200 to 300ms |
| Large or complex movement | up to 400ms (hard ceiling) |

**Easing direction matters more than the exact curve.** Things arriving decelerate (ease-out); things leaving accelerate (ease-in). Exits should be a touch faster than entrances because the user is already moving on (a common pairing is 300ms in, 200ms out). Reach for custom curves; the built-in CSS keywords are usually too weak. Useful starting values:

- Standard both directions: `cubic-bezier(0.4, 0, 0.2, 1)`
- Enter / decelerate: `cubic-bezier(0, 0, 0.2, 1)`
- Exit / accelerate: `cubic-bezier(0.4, 0, 1, 1)`
- Linear only for spinners and continuous rotation; it looks unnatural for anything else.

**Springs for anything interruptible.** For gesture-driven, draggable, or mouse-following motion, use spring physics (stiffness, damping, mass) rather than fixed-duration easing. Springs keep velocity when interrupted; easing animations restart from zero and feel broken under rapid input. Arrivals that settle with a spring feel physical; gratuitous bounce does not.

**Performance is non-negotiable.** A frame budget at 60fps is 16.7ms, so do your own work in about 10ms. Animate only `transform` and `opacity` (and where needed `filter` / `clip-path`); these run on the compositor and skip layout and paint. Animating `width`, `height`, `top`, `left`, `margin`, or `padding` causes layout thrash and dropped frames. Use `will-change` only right before an animation, not as a blanket.

**When not to animate.** Skip motion on keyboard-repeatable actions (they can fire hundreds of times a day), on functional data where motion delays reading, and anywhere animation is the only channel carrying important information.

**Respect reduced motion.** Honor `@media (prefers-reduced-motion: reduce)`. It does not mean "remove all motion"; it means replace large vestibular triggers (big scale, pan, spin, parallax) with a gentle opacity fade. Author the full motion, then soften it inside the query.

---

## Micro-interactions and feedback

The model for any micro-interaction: a trigger starts it, rules decide what happens, feedback communicates the result, and loops handle repetition.

**Human response thresholds (stable for decades).** Under 100ms feels instant and directly caused by the user, with no extra feedback needed beyond showing the result. Up to 1 second, the user stays in their flow but notices the wait. Keep system response under about 400ms to keep someone "in flow." Past 10 seconds you lose attention, so show progress.

**Feedback on every action.** A click, a save, a submit: each gets an immediate visible response. If a result will take longer than 1 second, show a loader; if it takes under 1 second, show none, because a sub-second flash is more annoying than nothing.

**Choose the right waiting pattern.** Spinners for short, indeterminate operations (save, auth). Skeleton screens for content loading, because they communicate structure and feel faster. Progress bars when completion is actually measurable.

**Optimistic UI where it fits.** For chat, social, and collaborative actions, update the UI immediately and reconcile on server response, with explicit rollback on failure. The network latency disappears perceptually.

**State styles, done right.** Use `:focus-visible` for focus rings so keyboard users get a clear indicator and mouse users do not get one on click; never strip focus outlines without a replacement. Order link pseudo-classes link, visited, hover, active. Suppress hover styling on touch with `@media (hover: hover)`.

**Meaningful, not decorative.** Animate to signal a real state change (a modal opening, a tab switching, a value updating) or to explain a relationship. If a motion appears for no reason and signals nothing, cut it. Reward the moment that genuinely matters (a real success), and base that reward on a real event so it does not fire on every render.

---

## Interaction details

The small decisions that separate competent from excellent.

**Hit targets.** Minimum 24x24 CSS px to meet WCAG 2.2 AA; aim for 44x44 px (Apple) or 48x48 dp (Material) for primary touch targets, with at least 8px between them. The visible element can be smaller than its padded hitbox.

**Perceived speed.** Treat 100ms as the budget for "feels instant" feedback and keep interaction animations at or under 200ms so they feel immediate. Real-time input (typing, cursor) needs sub-50ms response.

**Concrete rules worth following.**
- Build focus rings with `box-shadow`, not `outline`, so they respect border-radius.
- Set form inputs to `font-size: 16px` minimum to stop iOS zooming on focus.
- Open menus on `mousedown` rather than `click` for a snappier feel.
- Remove the tap flash with `-webkit-tap-highlight-color: transparent`.
- In lists, leave no dead zones between rows; grow padding, not gaps. Support arrow-key navigation.
- Disable a submit button after submission to prevent duplicate requests.

**Disabled states.** A native `disabled` button cannot be focused, is skipped by screen readers, often fails contrast, and hides why it is blocked. Prefer keeping the control active and showing a specific reason on click, or use `aria-disabled="true"` (which stays focusable and announced) with a tooltip or live message explaining the blocker.

**Modals and focus.** Trap Tab inside the dialog and loop it, make the background `inert`, close on Escape, and return focus to the element that opened it. Use `role="dialog"`, `aria-modal="true"`, and an `aria-labelledby` title.

**Forms.** Top-aligned labels complete fastest; labels live outside the field and placeholders are not labels. Validate inline on blur, not on every keystroke. Put each error directly beside its field. Set `type`, `inputmode`, and `autocomplete` correctly (for example `inputmode="numeric"` plus `autocomplete="one-time-code"` for OTP fields).

**Scroll.** Use `scroll-padding-top` so anchored content does not hide under a sticky header, and `overscroll-behavior: contain` to stop scroll chaining and pull-to-refresh bounce.

---

## Visual craft

**Spacing on a system.** Use a 4px base on an 8pt grid (4, 8, 12, 16, 24, 32, 48). Keep adjacent steps at least ~25% apart, which is why a non-linear scale beats fixed increments. An element's internal padding should be less than or equal to the space around it, so groups stay distinct.

**Type scale and hierarchy.** Line-height is inversely proportional to size: more for body, less for large headings (body around 1.5, headlines tighter). Establish hierarchy with size, weight, and color, not size alone. Hold body text at 16px or more.

**Color and contrast (WCAG, the numbers to hit).**

| What | Level | Ratio |
|---|---|---|
| Normal text | AA | 4.5:1 |
| Large text (≥24px, or ≥18.66px bold) | AA | 3:1 |
| Normal text | AAA | 7:1 |
| UI components, icons, focus indicators | AA | 3:1 |

Target AA in production. Logos, disabled controls, and pure decoration are exempt.

**Shadows.** A single flat grey shadow looks cheap. Layer two or more shadows, favor a vertical offset over heavy blur (light comes from above), and tint the shadow toward the background's hue at lower lightness so it does not read as muddy grey. Keep elevation consistent: define a small set of levels and reuse them.

**Border radius.** Keep it consistent, and nest it correctly: inner radius equals outer radius minus the padding between them, so concentric corners stay parallel (`calc(var(--outer) - var(--padding))`). Reusing the same radius on parent and child makes the gap look uneven.

**Optical alignment.** Trust your eye over the bounding box. Center non-rectangular shapes by their visual mass, not their rectangle (the play triangle in a circle needs a nudge right). Adjust optically whenever the math looks off.

---

## Accessibility as craft

Treat this as part of doing the job well, not a compliance pass at the end.

- **Semantic HTML first.** Use real `<button>`, `<a>`, `<nav>`, `<main>`, `<header>`. A styled `div` carries no meaning for assistive tech.
- **ARIA is the fallback, not the default.** If a native element gives you the semantics and behavior, use it instead of re-implementing with roles. Do not add a role a native element already has.
- **Visible focus** everywhere, via `:focus-visible`.
- **Contrast** meets the numbers above.
- **Reduced motion** honored.
- **Live updates** announced: `aria-live="polite"` for non-urgent changes, `assertive` only for time-critical alerts. Meaningful icons need 3:1 contrast and a text alternative.

---

## Performance is part of the feel

Jank reads as low quality even when nothing is broken. Design with these targets in mind.

| Core Web Vital | Measures | Good |
|---|---|---|
| LCP (Largest Contentful Paint) | load speed | ≤ 2.5s |
| CLS (Cumulative Layout Shift) | visual stability | ≤ 0.1 |
| INP (Interaction to Next Paint) | responsiveness | ≤ 200ms |

- **Stop layout shift.** Set explicit `width`/`height` or `aspect-ratio` on images and media so space is reserved before they load. Reserve space for anything that arrives late (ads, embeds, banners) rather than letting it push content down.
- **Keep the main thread free.** Any task over 50ms blocks interaction; break up long tasks so INP stays low.
- **Lean on perceived performance.** Optimistic updates, skeletons, and the sub-1s no-loader rule make a product feel fast without changing server time.

---

## The pre-ship checklist

Before showing a UI to anyone, answer yes to all of these.

- [ ] Does it lead with a clear frame, not just a look?
- [ ] Did I avoid the generic defaults (font, gradient, sparkle, centered-everything)?
- [ ] Is depth a deliberate choice, applied consistently?
- [ ] Does visual weight match what is real versus pending?
- [ ] Is motion purposeful, under ~300ms, on transform/opacity, and reduced-motion safe?
- [ ] Does every action give immediate feedback, with the right loading pattern?
- [ ] Do focus, hover, active, disabled, loading, error, and empty states all exist and look right?
- [ ] Do contrast, hit targets, and semantics meet the numbers above?
- [ ] Any layout shift or jank? Any task over 50ms on interaction?
- [ ] Did I render it, screenshot it, compare to intent, and fix the differences?
- [ ] No visible-but-dead controls?
- [ ] Did I finish the whole sweep, not part of it?
