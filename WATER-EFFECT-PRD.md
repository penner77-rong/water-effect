# Water Effect — Product Requirements Document

## Current baseline — 2026-11-05

This section describes the current implementation and takes precedence over the dated history below.

- The editable source in the Portfolio workspace is `water-interaction-prototype.html`. The publishable copy is `hello-world-water-effect/index.html`. Synchronize these before each release.
- A dated workspace checkpoint preserves this version as `water-interaction-prototype-checkpoint-2026-11-05.html`.
- The water uses a CPU height-field simulation and WebGL shading. Simulation advances at 60 fixed steps per second, with catch-up capped at four steps after a long frame. The grid follows the viewport aspect ratio.
- Slow pointer strokes have a wider, stronger V. Fast strokes narrow it smoothly. The default slow opening is 52 degrees and the fast opening approaches 23.4 degrees. One widening depression forms the wake, with gradual asymmetric bends based on path distance.
- Clicks produce a soft radial depression plus randomized stars and particles. The click displacement was doubled to a randomized 0.070–0.100. Default click lifetime is 3 seconds and wake lifetime is 4 seconds, both adjustable.
- Ambient shader motion and edge disturbances remain. The separately added breeze/wrinkle experiment was removed at the user's request.
- Ripple lighting exposes a shadow picker with hue, saturation/value, hex, opacity, and a browser-supported eyedropper. Highlight brightness and interaction refraction have independent sliders.
- Background gradient editing supports linear gradients with multiple draggable stops, numeric positions, per-stop color and opacity, reversal, and 90-degree rotation. The initial diagonal runs from #E1F0F8 to #94CDEC. Transparent stops blend over white.
- Background text supports multiple lines, a local font family, automatic or explicit pixel size, weight, and color. Installed font enumeration uses a browser permission request. Manual family entry is available when enumeration is unavailable. Font availability belongs to each visitor's device.
- Uploaded images fill the background and take precedence over the gradient and text. Reset removes the image and reveals the current editor values.
- Audio is opt-in, using local stirring and drop recordings. Sound follows interaction.
- Settings are session-only and reset on reload. Uploaded images and local fonts are not bundled into the HTML by editing the panel.

### Verification and remaining work

Chrome checks verified shader startup and live shadow, opacity, highlight, refraction, gradient, and text updates. Still-frame tests covered clicks, straight strokes, and curved strokes. Fixed-step checks compared 30, 60, and 120 Hz input frame sequences. The local font permission flow has not been verified interactively.

The material remains an artistic water approximation. Continuous bevel-like edges can still appear on some strokes. Mobile gestures, performance across devices, and appearance across browsers need further review. Reduced-motion mode currently renders a single frame, so it does not provide animated interaction. These are limitations of this baseline, not completed acceptance checks.

## Earlier record

The following material preserves previous requirements and iterations. Use the current baseline above to resolve conflicting values or behavior.

**Status:** Current prototype baseline
**Product area:** `Hello world` landing scene
**Primary implementation:** [`index.html`](index.html)
**Related technical history:** [`WATER-EFFECT-NOTES.md`](WATER-EFFECT-NOTES.md)

## 1. Product vision

Turn the `Hello world` page into a clear, sunlit lake that feels alive before a person interacts with it and responds naturally when they do.

The intended impression is not a literal fluid-dynamics simulation. It is a visually convincing, expressive water material: transparent, luminous, dimensional, and calm—then visibly responsive to movement, clicks, and the container edges.

## 2. Goals

- Make the page immediately read as a water surface rather than a coloured background, grid, plastic sheet, or static texture.
- Keep the underlying page and title visible through clear, translucent water.
- Use refraction, depth, reflected sky, lighting, shade, and motion together to create dimensionality.
- Give the entire lake one coherent passive flow direction.
- Make mouse movement feel like a small canoe travelling over water.
- Make clicks feel like a brief impact on the surface.
- Use subtle, interaction-aligned sound without making the idle scene noisy.
- Preserve performance suitable for a full-screen desktop browser experience.

## 3. Experience principles

1. **Material before decoration.** Water cues must come from distortion, changing light, depth, and physical response—not from a visible grid or a collection of drawn lines.
2. **Clear, not empty.** Transparency should reveal the page beneath while retaining enough colour, shadow, reflection, and contrast to read as water.
3. **One water body.** Passive motion, reflections, shading, refraction, edge reactions, and user disturbances must feel part of the same surface.
4. **Exaggerate selectively.** The effect may be stronger than real life so people notice it, but should not become laser-like, opaque, plastic, smoky, or visually busy.
5. **Finite causes, finite effects.** Every user-generated wave has an isolated source and a defined lifetime; old waves must never return because of a later interaction.

## 4. Target experience

### Idle state

The page resembles a bright, transparent lake under a soft sky:

- Background transitions diagonally from `#E1F0F8` at top-left to `#94CDEC` at bottom-right.
- The water has gentle depth variation without looking muddy or like a separate second layer.
- The title is visibly, but softly, refracted by the moving surface.
- Soft reflected-cloud forms drift and reshape with the water rather than sliding as a static overlay.
- Light/shadow variation and sparse glints add sunlight and dimension without forming a cellular grid.
- Passive water reaches downstream container edges and produces irregular return ripples.
- The scene is near-silent by default.

### Mouse movement

The pointer behaves as a tiny canoe:

- Pointer direction and speed determine the wake direction and strength.
- A bow response opens ahead of travel; trailing/hull disturbance follows the travelled path.
- Faster movement produces a longer, more energetic wake; slower movement produces a shorter one.
- Wakes affect the same refractive/light-responsive water surface as the idle lake.
- A location receives a one-time impulse as the pointer passes; it must not continue emitting waves afterward.
- Each wake fades smoothly over **4 seconds**.
- A natural water-stirring sound responds continuously to movement energy when sound is enabled.

### Mouse click

Clicks behave as a surface impact:

- A click creates a radial height-field ripple—not a visible rotating 2D ring.
- Ripple scale varies per click; the visual target range is approximately **90–320 px** in diameter before propagation.
- A click may create subtle randomized star glints and bioluminescent-like particles, kept sparse enough to avoid visual clutter or trypophobia.
- Each click ripple fades smoothly over **3 seconds**.
- A short water-drop sound plays when sound is enabled.

### Container edges

- The lake’s passive current moves downstream toward the right/bottom boundaries.
- When water reaches these boundaries, broad irregular disturbances reflect into return ripples.
- Edge activity must follow the same overall flow direction, not appear as unrelated random splashes.
- Passive edge impacts use a roughly **4.2-second** life and variation in timing, width, and placement.

## 5. Functional requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-01 | Render a full-screen transparent water surface over the Hello world composition. | Must |
| FR-02 | Refract the background/title using a time-varying height/noise field. | Must |
| FR-03 | Render transparent depth tint, shade, reflected sky, and light response as one optical surface. | Must |
| FR-04 | Maintain a shared passive current direction across water features. | Must |
| FR-05 | Produce passive downstream edge impacts and reflected return waves. | Should |
| FR-06 | Convert pointer movement into directional canoe-style wake impulses. | Must |
| FR-07 | Generate finite click ripples with randomized scale and smooth fade-out. | Must |
| FR-08 | Keep interaction wave instances independent and remove them at expiry. | Must |
| FR-09 | Provide opt-in movement and click audio that follows interaction intensity. | Should |
| FR-10 | Respect reduced-motion preferences by reducing or stopping non-essential animation. | Should |
| FR-11 | Provide a control panel for background-image upload/reset and ripple behaviour tuning. | Should |

## 6. Visual requirements and guardrails

### Required visual cues

- Visible refraction in the title/background.
- Light-to-deep colour variation without obscuring the page.
- Soft, expansive reflected-sky/cloud presence whose shape evolves with the current.
- Localized sunlight/glint response that is sharp enough to read, but sparse.
- Edge reflections/return ripples that make the container feel physical.

### Avoid

- Even cellular grids, bright double-line cell walls, or overly dense caustics.
- Pill-shaped shadows used as isolated decorative marks.
- Thick white highlights, broad clipped blooms, or “lightsaber” beams.
- Simple 2D V strokes, rotating rings, or atomic-bomb-like shockwaves.
- Cloud layers that slide rigidly as a film, look like smoke, or dominate the scene.
- Excessive small particles or repeated marks that create a trypophobic pattern.
- Dark, murky, ink-in-water colour treatment.

## 7. Technical approach

### Rendering

- Use a WebGL fragment shader to compose the water surface.
- Build optical response from broad and fine procedural fields:
  - height/noise fields for displacement;
  - refractive sampling for title/background distortion;
  - transparent shallow/deep tint for depth;
  - sunlight and shade fields for volume;
  - deforming, low-contrast reflected-cloud fields;
  - sparse glints tied to the same surface motion.
- Advect passive fields using one shared flow vector, currently approximately `(0.78, 0.62)`.

### Wave state

- Store clicks, wakes, and passive edge impacts as timestamped independent sources.
- Each source has a birth time, duration, strength, position, and—where relevant—direction/speed.
- Prune expired state rather than reusing it. This prevents stale waves from reappearing on the next interaction.
- Render physical-looking height/refraction/light response from wave state rather than overlaying line graphics.

### Audio

- Audio is opt-in due to browser autoplay restrictions and user control.
- Keep the movement audio loop silently pre-warmed after the user enables sound to reduce first-interaction latency.
- Use smoothed gain and a slight playback-rate response to pointer speed; do not repeatedly restart the audio file.
- Use rotating click-audio voices with slight gain/pitch variation.
- Do not use the rejected procedural-noise approach as the default natural-lake sound.

### Visitor customisation

- Visitors may upload a local image as the background beneath the transparent water layer.
- The image is cover-cropped to the viewport; the water shader and interaction system are not replaced or recoloured.
- A reset action restores the original diagonal-gradient `Hello world` composition.
- Exposed controls focus on meaningful water behaviour: click-ripple size and fade duration, drag-wake strength/spread/fade duration, and passive edge-motion intensity.

## 8. Non-functional requirements

- Maintain a stable animation loop on modern desktop browsers.
- Do not block pointer interaction with decorative layers.
- Avoid visible flicker, abrupt wave disappearance, recurring expired waves, or abrupt audio cuts.
- Degrade gracefully if WebGL or audio is unavailable: preserve the base page and title.
- Keep all assets local or appropriately licensed before public deployment.

## 9. Acceptance criteria

The prototype is ready for the next design review when:

1. At rest, a reviewer can identify the scene as transparent moving water without relying solely on title distortion.
2. The title remains readable, while its distortion visibly indicates the shared flow direction.
3. The lake looks clear and sunlit—not murky, flat, cellular, smoky, or plastic.
4. Cloud reflection evolves softly with the surface instead of moving as a rigid overlay.
5. Mouse movement creates a visible directional wake that remains for about four seconds and then fades smoothly.
6. A pointer path does not keep generating waves after the pointer has passed a location.
7. Clicks create randomized, water-like radial disturbances that disappear smoothly within about three seconds.
8. Earlier ripples do not reappear after a new mouse movement or click.
9. Edge-return ripples are noticeable in motion, varied, and aligned with the passive flow.
10. With sound enabled, movement audio begins promptly, changes smoothly with movement, and releases naturally after motion stops; the idle state remains nearly silent.

## 10. Out of scope for this phase

- Scientifically accurate Navier–Stokes fluid simulation.
- 3D objects, fish, flowers, or a physical pool container model.
- Mobile-specific interaction and performance tuning.
- A complete production accessibility/audio settings system.
- Publishing, hosting, or analytics.

## 11. Open tuning questions

- What is the final balance between cloud visibility and transparent-water clarity?
- How much edge activity is enough to communicate movement without distracting from the title?
- Which water-stirring recording best communicates an outdoor lake rather than an enclosed space?
- Which controls should be exposed in the tuning panel, versus kept as implementation parameters?
- What visual threshold makes a glint readable without making the surface feel like plastic or introducing dense patterns?
