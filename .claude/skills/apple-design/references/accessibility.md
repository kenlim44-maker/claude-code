# Reduced Motion & Accessibility

Reduced motion doesn't mean *no* feedback; it means a gentler, non-vestibular equivalent. Respond to three independent signals and build them into your components:

- **`prefers-reduced-motion: reduce`:** replace slides/springs/parallax with short opacity **cross-fades** or static transitions. Drop elastic overshoot. Keep opacity/color changes that aid comprehension.
- **`prefers-reduced-transparency: reduce`:** make translucent surfaces frostier or solid: raise background opacity, drop the blur.
- **`prefers-contrast: more`:** near-solid backgrounds with a defined, contrasting border.

Also:

- Avoid full-viewport moving backgrounds, slow looping oscillations (around 0.2 Hz, one cycle per ~5s), and abrupt brightness jumps (ease dark/light theme changes).
- Make large moving objects semi-transparent while they travel, and fade big surfaces out during a large reposition and back in once settled.
- Gesture-driven motion that tracks the finger 1:1 (drags) is user-controlled and generally fine under reduced motion; it is the *automatic* spring travel, parallax, and zoom that should be tamed.
- In JS-driven springs, read the preference with `matchMedia('(prefers-reduced-motion: reduce)')` and switch to a short fade or an instant state change.

```css
@media (prefers-reduced-motion: reduce) {
  .sheet { transition: opacity 200ms ease; transform: none !important; }
}
@media (prefers-reduced-transparency: reduce) {
  .toolbar { background: white; -webkit-backdrop-filter: none; backdrop-filter: none; }
}
@media (prefers-contrast: more) {
  .toolbar { background: Canvas; border: 1px solid CanvasText; }
}
```

```js
const reduce = matchMedia('(prefers-reduced-motion: reduce)');
const spring = reduce.matches
  ? { duration: 0.15, ease: 'linear' }
  : { type: 'spring', bounce: 0, duration: 0.5 };
```
