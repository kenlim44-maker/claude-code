# Design Foundations: The Eight Principles

Attributed to Apple's *Principles of Great Design* WWDC session. The session name and year could not be verified when this skill was written, so confirm the source before quoting it. The principles themselves are consistent with Apple's Human Interface Guidelines. Apple frames design as serving four human needs: safety/predictability, understanding, achievement, and joy.

1. **Purpose.** Make with intention; decide what *not* to build. Every feature asks for the user's time, attention, and trust.
2. **Agency.** Keep people in control: offer choices, don't force a single path. Back it with forgiveness: easy undo for slips, and a confirmation dialog only for genuinely destructive, irreversible actions (overuse trains people to click through).
3. **Responsibility.** Act in the user's interest. Privacy: ask at the right moment, only for what's needed, transparently. Safety: anticipate misuse and harm, especially with AI. Add previews, confirmations, disclaimers; cut a feature whose risk outweighs its value.
4. **Familiarity.** Build on what people know. Use metaphors that are neither too literal nor too abstract, and honor their physics. Be consistent: things that look the same behave the same and live in the same place. Break a familiar pattern only if you can show it's better, then test it.
5. **Flexibility.** Design for different contexts, devices, and abilities. Adapt to the platform and situation. Design inclusively (age, language, expertise, accessibility). When no single layout fits everyone, let people personalize.
6. **Simplicity, not minimalism.** Strip the unnecessary so the core purpose shines; burying everything in one place looks minimal but isn't simple. Be concise and clear, using hierarchy (order, spacing, contrast). Sometimes *adding* context simplifies. Show the common path first, advanced options one level deeper.
7. **Craft.** Uncompromising attention to detail builds trust. Typography, adaptive color, clear iconography, responsive animation with immediate natural feedback. Every spacing, timing, and alignment value is a deliberate choice you can defend. Jittery scroll and misaligned icons read as carelessness.
8. **Delight.** The result of getting the other seven right, not confetti on top. Decide the emotion you want people to feel and reinforce it in every decision.

## Tactical rules

- **Four kinds of feedback:** status, completion, warning, error. Confirm meaningful actions, expose ongoing status, warn before problems, validate inline rather than on submit.
- **Wayfinding.** Every screen answers: Where am I? Where can I go? What's there? How do I get out? Never trap the user.
- **Grouping & mapping.** Proximity implies relationship; place a control near what it affects and arrange controls to mirror what they change. If a control needs a label to explain it, the mapping is weak.
- **Direct, specific labels** beat generic ones. Name nav items for their contents ("Progress", "Library"), not vague umbrellas.

## Process

- **Prototype interactively.** An interactive demo is worth a million static designs; a working prototype sets a bar that prevents a mediocre final build.
- **Design interaction and visuals together.** You shouldn't be able to tell where one ends and the other begins; motion is not a layer added after the pixels.
- **Test with real people in real context**, and review motion with fresh eyes in slow motion or frame by frame.
