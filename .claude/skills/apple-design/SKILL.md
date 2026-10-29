---
name: apple-design
description: Apple's approach to fluid, physical motion, translated for the web. Use when building or reviewing gesture-driven UI, spring animations, drag/swipe/sheet interactions, momentum and interruptible transitions, rubber-banding, or velocity handoff. Deeper references cover translucent materials and depth, audio-haptic feedback, reduced-motion and accessibility, typography (optical sizing, tracking, leading), and Apple's design principles.
---

# Apple Design

## Initial Response

If invoked with no task or question, reply only with:

> I'm ready to help you build fluid, Apple-style interfaces on the web. My knowledge comes from Apple's WWDC design talks, translated for the web.

If invoked with a task, skip the greeting and do the task.

How Apple builds interfaces that feel like an extension of you, distilled from WWDC talks (chiefly *Designing Fluid Interfaces*, 2018) and translated to the web platform (CSS, Pointer Events, `requestAnimationFrame`, spring libraries like Motion/Framer Motion).

The through-line: **an interface feels alive when motion starts from the current on-screen value, inherits the user's velocity, projects momentum forward, and can be grabbed and reversed at any instant.** Springs make this natural because they are inherently interruptible and velocity-aware.

## Reference files (load on demand)

| Topic | File |
| --- | --- |
| Translucent materials, depth, scroll edges; audio-haptic feedback | `references/materials-and-feedback.md` |
| Reduced motion / transparency / contrast, vestibular safety | `references/accessibility.md` |
| Tracking, leading, optical sizing, Dynamic Type | `references/typography.md` |
| The eight design principles, feedback kinds, wayfinding, process | `references/design-principles.md` |

## 1. Response: kill latency

The moment lag appears, the feeling of directness "falls off a cliff."

- **Respond on pointer-down, not on release.** Highlight a button the instant it is pressed.
  - Exception: inside scrollable containers, a pressed state on pointer-down flashes on every scroll start. Delay it by ~100ms or cancel it on `pointercancel`/movement past the threshold.
- **Audit every latency.** Debounces, artificial timers, transition waits, and the legacy tap delay (avoid with `touch-action: manipulation` and a proper viewport meta).
- **Feedback must be continuous during the interaction**, not just at the end. For a drag, slider, or drawer, update the UI 1:1 with the pointer throughout.

```css
.button:active {
  transform: scale(0.97);
  transition: transform 100ms ease-out;
}
```

## 2. Direct manipulation: 1:1 tracking

Touch and content move together, and the offset from where the user grabbed is respected. Snapping the element's center to the pointer breaks the illusion.

- Use Pointer Events with `setPointerCapture` so tracking continues outside the element.
- **Set `touch-action` on the draggable.** Use `touch-action: none` for free 2D drag, or `pan-y`/`pan-x` when the perpendicular axis should still scroll. Without it the browser claims the gesture on touch devices and fires `pointercancel` mid-drag.
- **Handle `pointercancel`** exactly like release without a velocity (settle to the nearest state).
- Set `user-select: none` on the drag surface to avoid accidental text selection.
- Keep a short **position + timestamp history** (roughly the last 80-100ms of `pointermove` events) and compute velocity from it at release, not from the last two samples (too noisy).

```js
el.addEventListener('pointerdown', (e) => {
  el.setPointerCapture(e.pointerId);
  const grabOffset = e.clientY - el.getBoundingClientRect().top;
  // track {y, t} history for velocity
});
el.addEventListener('pointercancel', settleWithoutVelocity);
```

## 3. Interruptibility: the most important principle

Every animation must be interruptible and redirectable at any moment. A closing modal the user grabs again should follow the finger, not finish closing first.

- **Never lock out input during a transition.**
- **Animate from the presentation (current) value, never the target.** On interrupt, read the live on-screen value and start from there. On the web:
  - With a JS spring library, read the library's current motion value.
  - With a CSS transform or WAAPI animation, read `getComputedStyle(el).transform` (or `new DOMMatrix(...)`) or `getBoundingClientRect()`, then cancel the animation and commit that value as the new start.
- **Avoid CSS transitions and `@keyframes` for gesture-driven motion.** They cannot be grabbed and reversed smoothly. Springs animate from the current value by default.
- **When a gesture reverses, blend velocity; don't hard-cut it.** Replacing one animation with another at a reversal creates a velocity discontinuity (a "brick wall"). Choose a spring library that re-targets from the current velocity.
- **Decompose 2D motion into independent X and Y springs.** One spring on a 2D distance desyncs when X and Y velocities differ.

## 4. Behavior over animation: use springs

A fixed-duration animation can't respond to new input. A spring can: new input just changes the target and motion stays continuous.

Apple replaced mass/stiffness/damping with two parameters:

- **Damping ratio:** `1.0` is critically damped (no bounce). `< 1.0` overshoots; lower is bouncier.
- **Response:** how quickly the value approaches the target, in seconds. Lower is snappier. Not a fixed duration.

**Defaults:**
- Start with **damping `1.0`**: graceful and non-distracting.
- Add bounce (**~`0.8`**) **only when the gesture itself carried momentum** (flick, throw, release). Overshoot on a menu that just appeared feels wrong; on a flicked card it feels right.

Starting values (approximate, from Apple's talk; tune by feel):

| Interaction | Damping | Response |
| --- | --- | --- |
| Move / reposition (e.g. PiP) | `1.0` | `0.4` |
| Rotation | `0.8` | `0.4` |
| Drawer / sheet | `0.8` | `0.3` |

**Web mapping (Motion):** `bounce` and `duration` are close to, but not the same as, damping and response. `bounce ≈ 1 − dampingRatio`, and `duration` is perceptual duration, which is longer than `response`. Start with `duration` ≈ `response × 1.5-2` and adjust by eye.

```js
import { animate } from 'motion';

// Critically damped default
animate(el, { y: 0 }, { type: 'spring', bounce: 0, duration: 0.5 });

// Momentum interaction: a little bounce, only because a flick preceded it
animate(el, { y: target }, { type: 'spring', bounce: 0.2, duration: 0.5, velocity });
```

## 5. Velocity handoff: the seam between drag and animation

When a gesture ends, the animation must continue at the pointer's release velocity so there is no visible seam. This is what most separates "fluid" from "fine."

Some spring APIs take **relative** velocity, normalized by remaining distance:

```
relativeVelocity = gestureVelocity / (targetValue − currentValue)
```

Example: at `y=50`, target `y=150` (100px to go), moving 50px/s gives initial velocity `50 / 100 = 0.5`. Motion/Framer Motion take absolute px/s via the `velocity` option, so pass the raw value there. Check which convention your library uses.

## 6. Momentum projection: animate to where the gesture is going

Don't snap to the nearest boundary from the release point. Project the resting position from velocity (like scroll deceleration), then pick the snap point nearest the projection.

```js
// decelerationRate ≈ 0.998 for normal scroll feel; 0.99 for snappier
function project(initialVelocity /* px/s */, decelerationRate = 0.998) {
  return (initialVelocity / 1000) * decelerationRate / (1 - decelerationRate);
}

const projected = currentPosition + project(releaseVelocity);
const target = nearestSnapPoint(projected);
animateSpringTo(target, { velocity: releaseVelocity }); // then hand off velocity (section 5)
```

This is Apple's exponential-decay form, not the textbook `v²/(2·decel)`. It is the standard behavior in good bottom sheets and carousels (Vaul, Embla).

**Reverse vs. commit:** for a dismiss gesture, use the sign and magnitude of the release velocity first (a quick flick back toward the origin cancels, a flick away commits), and fall back to the projected position when velocity is small.

## 7. Spatial consistency: symmetric paths, anchored origins

- **Enter and exit along the same path.** A panel that slides in from the right dismisses to the right.
- **Anchor interactions to their source.** Set `transform-origin` on menus, popovers, and sheets to the trigger element, not the center.
- **Mirror easing on reversible transitions** (inverse cubic-bézier control points for the return trip).

## 8. Hint in the direction of the gesture

People predict the end state from the trajectory. Make intermediate frames point at the outcome (Control Center modules grow up and out toward the finger) rather than interpolating blindly.

## 9. Rubber-banding: soft boundaries

At an edge, resist progressively instead of stopping hard. Hard stops read as frozen; growing resistance reads as "responsive, nothing more here."

```js
// The further past the bound, the less the element follows
function rubberband(overshoot, dimension, constant = 0.55) {
  return (overshoot * dimension * constant) / (dimension + constant * Math.abs(overshoot));
}
```

On release, spring back to the bound with the release velocity handed off.

## 10. Gesture design details

- **Tap:** highlight on down (instant), commit on up. Add ~10px of hit padding, and allow cancel-by-dragging-away and back.
- **Drag/swipe:** require a small threshold (~10px hysteresis) before committing to a direction, then track 1:1.
- **Run all plausible gesture recognizers in parallel** from the first move and cancel the losers once intent is clear. Avoid recognizers that only report a final state (`swipeleft`-style); they discard the continuous tracking you need.
- **Minimize disambiguation delays.** Double-tap detection delays single taps; pay that only where double-tap exists.

## 11. Frame-level smoothness

- Animate **only compositor-friendly properties** (`transform`, `opacity`) and use `will-change` sparingly, just before motion starts. Layout-affecting and paint-heavy properties (width, top, `filter`, `backdrop-filter` radius) are expensive to animate; see the materials reference for how to handle blur.
- `requestAnimationFrame` is the display-synced clock (Apple uses `CADisplayLink`). Don't drive animation from `setInterval`.
- Keep the per-frame positional change below the strobing threshold; for very fast motion a subtle stretch or blur encodes speed better than a hard streak.
- Review motion in slow motion (DevTools animation speed) and frame by frame.

## Quick Reference

| Need | Technique | Concrete value |
| --- | --- | --- |
| Default UI spring | Critically damped | damping `1.0`, response `0.3-0.4` |
| Momentum / flick spring | Under-damped | damping `~0.8`, response `0.3-0.4` |
| Gesture to spring velocity | Hand off release velocity | `v / (target − current)` if normalized |
| Flick landing point | Project momentum | `current + (v/1000)·d/(1−d)`, `d ≈ 0.998` |
| Interrupt cleanly | Start from presentation value | read live transform / motion value |
| Avoid reversal "brick wall" | Carry velocity through re-target | spring that blends velocity |
| Reversible transition | Mirror the easing curve | inverse cubic-bézier |
| Reverse vs. commit | Release velocity first, then projection | |
| 1:1 drag | Pointer Events + capture + `touch-action` | respect the grab offset |
| Feedback | On pointer-down, continuous | never only at the end |
| Boundary | Rubber-band | progressive resistance |
| Type tracking | Size-specific | see `references/typography.md` |
| Reduced motion | Cross-fade instead of slide/spring | see `references/accessibility.md` |
