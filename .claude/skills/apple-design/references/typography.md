# Typography: Optical Sizing, Tracking, Leading

Apple designs type to change shape with size (*The Details of UI Typography*, WWDC 2020). The same discipline applies on the web.

- **Tracking (letter-spacing) is size-specific; never one value for all sizes.** Large display text wants *negative* tracking; small text wants slightly *positive* tracking. A fixed `letter-spacing` is wrong somewhere. Tighten headings, leave body near `0`.
- **Leading (line-height) tracks size inversely.** Tight on large headings, looser on body. Increase it for scripts with tall ascenders/descenders; tighten for dense, information-heavy UI.
- **Build hierarchy from weight + size + leading as a set**, not size alone. Weight adds presence without taking more space.
- **Respect the user's text-size setting** (Dynamic Type). Size type and spacing in `rem`/`em`, not px, so layout scales with the text. Never size text with `vw` alone, since it ignores browser zoom; combine with `rem` in `clamp()`.
- **Default to the platform's system font** (`system-ui`) before a custom face; it ships optical sizing, tracking tables, and legibility tuning.
- **Optical sizing only works if the font supports it.** `font-optical-sizing: auto` has an effect only for fonts with an `opsz` axis (and `system-ui` on Apple platforms). With other fonts it is a no-op, so choose tracking by size manually.

```css
:root { font: 100%/1.5 system-ui, sans-serif; }

.display {
  font-size: clamp(2rem, 1rem + 4vw, 4rem);
  line-height: 1.05;        /* tight leading for large text */
  letter-spacing: -0.02em;  /* negative tracking as it grows */
  font-optical-sizing: auto;
}

.caption {
  font-size: 0.75rem;
  letter-spacing: 0.01em;   /* slight positive tracking for small text */
}
```
