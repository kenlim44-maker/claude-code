# Materials, Depth, and Multimodal Feedback

## Materials & depth: translucency conveys hierarchy

Apple uses translucent materials as a floating functional layer that adds structure without stealing focus. On the web, approximate with `backdrop-filter`.

- **Build nav/toolbars/sheets as translucent layers** (`backdrop-filter: blur()` + semi-transparent background) with content scrolling underneath, not opaque bars that consume a fixed strip.
- **Material weight encodes hierarchy:** darker/heavier materials separate structural regions (sidebars); lighter materials draw attention to interactive elements. **Never stack a light translucent surface on another**; legibility collapses.
- **Bigger surfaces read as thicker:** stronger blur and a deeper shadow than small chips. Make shadows context-aware: heavier over busy content, lighter over plain backgrounds.
- **Dim to focus, separate to keep flow.** A modal task pairs the surface with a scrim and pushes the background back. A parallel, non-blocking panel uses translucency and offset without a scrim. For stacked sheets, progressively dim and push back each parent.
- **Vibrancy keeps text legible over changing backgrounds.** Over translucent surfaces use higher-contrast, slightly heavier text with a small letter-spacing bump rather than flat gray. Put color on a solid layer, not the translucent foreground.
- **Scroll edge effects, not hard dividers.** Instead of a 1px border under a sticky header, fade a small blur/gradient mask where content meets floating chrome, only where floating UI overlaps content.
- **Materialize, don't just fade.** Glass surfaces should read as a material arriving, not a plain opacity fade.

### Animating blur safely

Animating `backdrop-filter` blur radius forces the backdrop to be re-blurred every frame, which janks on low-end devices. Prefer:

- Animate `opacity` and `transform: scale()` on the surface, with the blur at its final radius (cheapest).
- If a blur ramp is needed, keep the surface small, animate briefly (<300ms), and test on a mid-range phone.
- Never animate blur on large full-screen surfaces. Fade a pre-blurred layer instead.

```css
.toolbar {
  background: rgba(255, 255, 255, 0.6);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  backdrop-filter: blur(20px) saturate(180%);
  border-top: 1px solid rgba(255, 255, 255, 0.4); /* bright top edge = light catching the material */
}
```

Provide fallbacks for `prefers-reduced-transparency` and `prefers-contrast` (see `accessibility.md`).

## Multimodal feedback: motion + sound + haptics

From *Designing Audio-Haptic Experiences*:

1. **Causality:** it must be obvious what caused the feedback. Fire it on the actual causal event (toggle flips, item snaps home) and match its character to the action's physicality.
2. **Harmony:** visual, sound, and haptic fire on the **same frame**. Latency between them destroys the illusion.
3. **Utility:** add feedback only where it earns its place (success, error, commit, snap). Over-feedback trains users to ignore all of it.

### Web platform reality

- The **Vibration API (`navigator.vibrate`) is not supported in iOS Safari**, so it gives no haptics on iPhone. Treat it as an Android-only enhancement and never rely on it.
- On iOS Safari 17.4+, toggling an `<input type="checkbox" switch>` produces a native haptic. Some projects exploit this as a tap-haptic workaround, but it is a hack: it must happen in a user gesture and may change.
- Always feature-detect, and make the UI complete without haptics or sound.
- For audio, use the Web Audio API with preloaded buffers; starting playback from `new Audio()` on demand adds latency that breaks harmony.
