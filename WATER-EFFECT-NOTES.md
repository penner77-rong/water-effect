# Water effect: implementation and collaboration notes

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

This document preserves the earlier accepted version in `index.html` and records the newer mouse-wake experiment in `water-interaction-prototype.html`, which is now the preferred interaction effect. The exact earlier restore point is `water-baseline-2026-10-23.html`; the prototype has not replaced `index.html`. This is a design and implementation record, not a frozen copy of the prototype; save a separate version or commit if you need an exact restore point for it.

## The intended experience

- A pale, transparent-looking lake moves gently even without interaction. The underlying “Hello world” remains visible.
- Moving the pointer disturbs the surface like a small boat. The wake should read as water, not as a drawn V, a laser, or a plastic sheet being stretched. Some exaggerated refraction is welcome when it improves visibility.
- A click creates a water disturbance and randomized star glints and particles. The waves should look like changes in the water surface, not outlined circles or a shockwave.
- Interactive waves fade smoothly and are removed after three seconds; a later interaction must not revive an earlier wave. The click size varies, with a randomized 90–320 px *target*, not a guaranteed measured diameter.
- The color panel adjusts the hue, saturation, value, and opacity of the darker surface shading. Bow and side controls adjust the wake's pressure sources.

## How the effect is built

Think of the page as three **conceptual** visual contributions, not necessarily three separate HTML layers:

1. **Background:** the page artwork and text, sampled by the renderer.
2. **Ambient surface:** a WebGL shader uses moving noise and a cellular pattern to create irregular caustic-like highlights, darker shading, and subtle refraction.
3. **Interaction:** a simulated height field adds moving surface disturbances. Its height and slope affect the shading and the displacement of the sampled background. Click stars and particles are separate HTML/CSS effects layered on top.

The cellular pattern is a better description than a regular grid: it is made from irregular neighboring regions, whose boundaries form a shifting network. The implementation compares nearby cell distances and warps the sampling coordinates with moving noise. “Caustics” describes the light patterns that suggest light being focused by water; the visual is an artistic approximation rather than a physical light simulation.

### Earlier interactive waves (`index.html`)

The implementation uses a **height field**: a two-dimensional array of numbers representing the surface's relative height. For each simulation step, neighboring heights influence one another, so an initial disturbance propagates and gradually damps. The result is a visual approximation, not a full fluid-dynamics or physically accurate boat simulation.

- **Movement:** pointer samples place short-lived pressure shapes around the current direction, including bow and side contributions. Each pending sample is injected once; the surface continues to propagate afterward without continuously pushing at that old position.
- **Click:** one soft initial displacement is injected into its own simulated field. The selected size changes the source and propagation characteristics; the expanding shape is produced by the simulation rather than by drawing an animated circle.
- **Appearance:** differences in neighboring heights approximate a surface slope (a normal). That slope refracts the background, while positive and negative heights contribute highlight and darker shading. Height data is encoded using two texture channels so small changes survive better than with one 8-bit channel.
- **Lifetime:** each click has its own simulation layer. Nearby movement samples are grouped into a wake layer for up to 500 ms, and each group has its own birth time. A smooth fade is applied to each layer; at three seconds the layer and its stored state are removed. Consequently, later interactions cannot re-amplify that old stored wave. Individual movement samples near the end of a group can live slightly less than three seconds.

The simulation is currently advanced once per animation frame. Its apparent propagation and damping can therefore vary with frame rate; the three-second expiration is based on elapsed time. This matters when comparing performance on different devices.

### Newer mouse wake (`water-interaction-prototype.html`)

The preferred prototype keeps the same ambient shader, click disturbance, appearance pipeline, and three-second layer lifetime. It changes **how pointer movement excites the height field**. It does not draw V-shaped lines or replace the idle surface.

1. Each pointer event gives a position, elapsed time, movement direction, and speed in simulation-grid coordinates. The first position only establishes a starting point; an event after a long pause starts a fresh stroke rather than connecting two distant positions.
2. The segment between consecutive positions is sampled at roughly regular distances, rather than depositing only at the latest pointer event. This makes a fast drag less likely to leave gaps when the browser delivers fewer events. Faster movement increases the source strength within a capped range.
3. Each path sample makes a **one-time, direction-aligned displacement**: a soft depression with a low surrounding shoulder. That paired crest/trough profile launches a wavefront while avoiding the solid dark streak produced by a row of purely negative pressure marks. A smaller one-time depression at the newest pointer position helps the leading edge read immediately.
4. The simulation propagates those initial disturbances through neighboring height values. Their overlap produces the visible wake and its V-like envelope; no V outline is drawn. Nearby samples share a wake layer for up to 500 ms, while older layers keep aging and fading independently. A sample is never reinjected just because a later pointer event arrives.
5. The preferred wider opening sets **wake-layer stiffness to `0.072`**, up from the earlier/default `0.035`. This increases the wake's outward propagation relative to pointer travel, so its sides spread farther apart. Click-layer stiffness and the ambient water are unchanged. This is a visual approximation of a boat wake, not a calibrated physical Kelvin-wake angle.

The bow/side panel still affects the wake: the counts influence sample spacing; their depths influence displacement strength; and their radii and widths influence the disturbance footprint. In this prototype, those controls are **not** literal counts of separately drawn bow or side wave lines. The earlier `index.html` method instead deposits several bow- and side-offset pressure shapes around each current pointer position. Keep both descriptions because they correspond to different working versions.

The practical lesson from this iteration is to separate the **input source** from the **wave renderer**. The renderer can stay the same while the source changes from a few positioned pressure lobes to regularly spaced, one-time wavelets along the path. Once the shape felt right, the V opening could be adjusted through wake-only propagation without changing click ripples, idle caustics, or wave lifetime.

### Sound added to the preferred prototype

The first audio experiment synthesized filtered noise in the browser. It was too quiet and sounded more like wind than water, so it remains a discarded sound approach; the visual effect was not changed by this revision.

The current prototype uses two interactive CC0 recordings in `audio/`:

| Cue | File | Behavior and source |
| --- | --- | --- |
| Idle water | — | The surface is deliberately nearly silent. The previous lake-ambience recording remains in `audio/` as an unused reference, but it is no longer played. |
| Mouse movement | `audio/water-stirring.mp3` | A recording of a hand moving gently through water starts only while the pointer sweeps across the surface. Faster movement produces a louder, slightly brighter flow; after motion stops, the sound fades smoothly within about a third of a second. [Recording by Daen23](https://freesound.org/people/Daen23/sounds/431627/). |
| Mouse click | `audio/water-drop.mp3` | One short drop sound plays with each click; slight playback-rate variation prevents identical repetition. [Recording by mattfinarelli](https://freesound.org/people/mattfinarelli/sounds/533146/). |

These are compressed preview copies of recordings whose source pages identify them as CC0. The page uses normal local `<audio>` playback, so it also works when the HTML file is opened directly from disk. Audio is opt-in because browsers commonly block audible autoplay; turning **Sound on** arms the interaction sounds but does not start an ambient loop. The page pauses sound while hidden. When publishing the page, include the entire `audio/` directory beside the HTML file. Sound design is independent of the water-height simulation and should be judged by listening on the intended device.

The movement sound now uses a higher `18%–90%` volume range, based on a lightly smoothed pointer-speed value, and a wider playback-rate range. It reacts quickly to a faster sweep but avoids audible jitter from individual pointer events. When movement ends, it holds very briefly then fades over roughly `380ms`; it never becomes an idle ambient loop. The click-drop voice was raised to `88%`.

### More vivid still-water surface

To move the idle surface closer to the vivid shallow-pool reference, the preferred prototype now uses a deliberately simpler hierarchy: a full-surface turquoise water body, a sparse independent field of low-contrast, rotated shadow pills, and one restrained white-cyan caustic line at each boundary. This replaces the earlier double-ridge treatment, which read as cell walls rather than concentrated light. The caustic field was also reduced in density so the composition has more breathing room; both its drift and the pill field remain irregular rather than grid-like. The default depth color is `#279FAE` at `34%` opacity, editable in the existing color panel.

### Revision: full-region water shading

The sparse-pill experiment above is superseded in the current prototype. The idle shader now separates water depth from caustics: a slow, broad tonal field creates uneven deep-water regions while a finer cellular field supplies the subdued light paths above it. This prevents the earlier mistake of darkening every outlined region equally. There is no capsule or ring-shaped interior mask. The depth-opacity control scales the transparent base tint and the broader regional shading.

### Revision: pool depth, sunlight, and refraction

The latest idle treatment treats the page as a shallow pool viewed from above. A gentle depth slope is combined with the broad tonal field, so lower areas can read slightly deeper without relying on a uniform dark overlay. A fixed angled sunlight vector illuminates the procedural surface normal; it modulates caustic intensity, with small extra focal variation where the surface bends. Refraction now uses a modestly stronger normal and micro-wave offset, while interaction ripple displacement remains separately controlled. These layers share the same moving surface field so the water color, light, and distortion evolve together.

### Revision: clear-water palette

The depth field now uses transparent **shallow cyan**, **mid teal**, and **deep blue-green** values instead of mixing a near-black color over the whole page. The darkest values remain regional and translucent, while sunlight and caustics are deliberately restrained. This preserves the clear, bright quality of a shallow pool: depth is readable, but the page and title remain visible beneath the surface rather than looking grey, muddy, or covered by fog.

### Revision: broad refraction and sparse glints

The still-water field now has a second, lower-frequency normal dedicated to refraction. It creates broad, slow, visible distortion across the page and title, while the previous smaller normal retains fine shimmer. The caustic network is reduced in scale and masked into broken glints so it cannot become an all-over cell-wall pattern. Directional, softly warped sunlight pools add a separate large-scale lighting cue; water depth, light, refraction, and glints are intentionally independent layers so no single cellular pattern has to carry every visual job.

### Checkpoint: clean-water surface — 2026-10-25

`water-interaction-prototype-checkpoint-2026-10-25.html` preserves the approved clean-water composition before further experiments. This checkpoint has broad readable refraction, a clear three-value palette, broken secondary caustic glints, and subtle moving sunlight pools. The next experiment keeps those layers intact and changes only the ripple-field boundary condition: wave energy that reaches the container edge is softly reflected back with reduced energy, creating a physical return wave without an illustrated edge highlight.

### Experiment: slope-driven surface shimmer

The working prototype adds a sparse specular-reflection layer above the existing sky-reflection pools. It uses the same moving surface normal, a fixed sun direction, fine irregular grain, and the interaction-ripple normal to reveal brief cool glints only on favorably angled slopes. This deliberately differs from caustics: it is a reflection of light at the water surface, not an outlined cellular texture. Its intensity is capped below white to protect the clean, cloud-reflection character of the approved checkpoint.

The initial version was intentionally too restrained to read on the pale teal base. The active tuning widens the reflection lobe and raises the glint contribution. Its grain is now domain-warped by the same low- and mid-frequency flow fields that drive the water, so fine flecks are transported and reshaped with the surface rather than behaving like a separate cloud-texture layer.

The cellular-streaklet experiment was rejected because its isolated loops and lines read as bugs rather than water. The active shimmer pass now takes finite differences from a fine, advected noise height field to produce micro surface slopes, combines them with the existing broad and interactive slopes, then illuminates only slopes aligned with the fixed sun vector. It uses no cellular, dot, or line mask: any visible glint is a direct consequence of a moving surface facet facing the light.

That micro-facet experiment was also removed after visual review: at a readable intensity it formed an uncomfortable dense speckle field. The active still-water surface intentionally returns to the clean reflected-sky treatment—broad cloud-like light pools, transparent depth variation, and refraction—with no extra high-frequency shimmer overlay. Any future sparkle pass must be opt-in, low-density, and judged first against this calm baseline.

### Experiment: sparse diamond sunlight glints

The current opt-in sparkle pass contains only a small set of irregularly positioned, compact filled diamonds. Candidate locations are heavily discarded, so it does not form a dense pore-like field. Their coordinates are advected by the same broad and fine flow vectors used by the surface, plus local ripple displacement; their visibility is modulated by the simulated sun-facing slope. There are no ring, outline, streak, or blurred-halo elements in this pass.

The second tuning constrains the diamonds to the existing sunlit reflection pools and adds small brightness variation sampled from their flowing coordinates. This prevents bright isolated shards from appearing over calm or dark areas, while allowing an occasional glint to reveal itself briefly as the same water field moves.

### Revision: sunlight-first composition

The idle surface now establishes a broad, moving warm-cyan sunlight footprint before rendering its compact diamond glints. Mid and deep water colors were raised toward clear turquoise and their blend strength reduced, preventing the wide cool shadow regions from reading as an overcast or windy lake. The diamonds are now gated primarily by the direct sunlight footprint; they are secondary evidence of a sunny surface rather than the source of the sunny feeling.

The follow-up relief tuning restores transparent tonal structure around that sunlight: fewer broad teal troughs are strengthened outside the direct reflection footprint, while the adjacent sun pools are brighter and warmer. This produces intentional light–dark pairing for water volume without redrawing cellular borders or filling the surface with high-frequency texture.

The first relief tuning was still too close in value to the base teal to be legible. The active revision deliberately narrows and brightens the sunlight footprint, recenters it in the visible field, and gives its complementary trough regions a stronger transparent deep-teal mix. This is a compositional light/shadow adjustment, not a new texture layer.

### Decision: return to the clean reflected-sky baseline

The high-contrast trough and direct-sun experiments were rejected because the dark expanding regions read as ink diffusing in water rather than optical depth. The active surface returns to the saved clean-water palette and broad reflected-sky light treatment. The later physical boundary reflection remains enabled, but all experimental diamond sparkle and direct-sun overlay passes are disabled. This baseline is the correct calm starting point for any future lighting work.

### Revision: clearer translucent water defaults

The active default is now a lighter cyan (`#50B8C4`) at `26%` depth opacity. The shader's base tint and regional mid/deep mixes were reduced and shifted upward in value, so shadowed areas remain transparent blue-green rather than reading as dark dyed water. Refraction and the wave-field geometry are unchanged; this is a material-clarity adjustment only.

### Current active method: unified flowing lake — 2026-10-29

This records the current implementation without replacing the earlier experiments above. Where values differ, this section is the current tuning reference.

#### One optical water body, not stacked visual layers

The screen is treated as one transparent body of water. Its appearance comes from several optical contributions combined in the fragment shader rather than from separate-looking sheets:

1. The source canvas uses the page-background gradient requested for the scene: `#E1F0F8` at the top-left through to `#94CDEC` at the bottom-right.
2. A transparent shallow-to-deeper water tint varies across that same surface. It provides depth through colour and shadow variation while preserving the clear, translucent quality of the background.
3. Broad and fine height/noise fields perturb the sampling coordinates, so the title and background refract as if viewed through moving water.
4. Soft sky reflection, sunlight, and local shade are mixed into the water response rather than painted as opaque objects over it.

The key visual rule is to preserve all of these cues—depth, reflection, sunlight, shadow, and refraction—while keeping the water clean. Removing them makes the result clearer but also makes it read as a flat coloured page rather than water.

#### A shared current gives passive effects one direction

The current direction is approximately top-left to bottom-right (`0.78, 0.62`). Macro flow, refraction, caustic breakup, depth variation, reflected sky, and passive edge activity all use that direction. This prevents each decorative effect from drifting independently and makes the distortion of the title a useful cue for the whole lake's movement.

Cloud reflection is generated from several low-contrast, time-varying noise fields: a broad base, a breakup field, and fine wisps. A secondary cross-current warps those fields so their silhouette changes as they travel. This is deliberately different from moving a single large cloud image across the screen, which reads as a static plastic film. The reflection remains soft and expansive, and becomes a little more visible where the surface has more motion.

#### Water meeting the container wall

Passive edge activity is now a physical storytelling cue instead of a decorative outline. Broad, irregular disturbances are seeded only on the downstream right and bottom edges. The wave solver reflects their energy back into the container, creating return ripples that remain aligned with the shared current. Their timing, width, and placement vary slightly so the boundaries do not pulse with a rigid, repeating pattern.

#### Interaction lifecycle: independent, finite wave instances

Each interaction creates its own timestamped wave source. Sources are removed after their own lifetime, rather than being re-used by a later interaction. This prevents the old failure mode in which a past ripple appeared to return when the next mouse action happened.

| Source | Current lifetime | Behaviour |
| --- | ---: | --- |
| Mouse click | 3 seconds | A randomized radial disturbance in the height field, with a smooth fade rather than a drawn 2D ring. |
| Mouse movement / canoe wake | 4 seconds | Directional one-time wake samples based on the pointer path; no location continues emitting after the cursor has passed. |
| Passive downstream edge impact | 4.2 seconds | Broad incoming disturbance followed by container-edge reflection. |

The mouse path is treated as a moving canoe vector. Its speed and direction control a directional pressure pattern in the simulation: the bow response opens ahead of travel and the hull wake trails along the path. This is rendered through the same height/refraction/light pipeline as the lake, not as visible V-shaped strokes. Clicks remain radial because they represent an impact rather than a moving body.

#### Interaction-aligned sound

The audio is opt-in. Still water is intentionally near-silent. Mouse movement controls a looping water-stirring recording, while clicks trigger randomized water-drop voices.

- When sound is enabled, the stirring loop is started silently and kept warm in the browser. This avoids the start-up delay that occurs if playback begins only after the first pointer movement.
- Pointer speed drives a smoothed motion-energy value. That value controls target volume and a small playback-rate shift, so a faster sweep sounds more energetic without repeatedly restarting the file.
- The loop has a short attack and a longer release, allowing the sound to follow movement while avoiding abrupt cut-ins and cut-outs.
- Click sounds use several rotating audio voices with small random changes in gain and pitch, preventing identical repeated drops.

An earlier procedural Web Audio noise experiment was intentionally abandoned: despite low latency, it read as an enclosed-room effect rather than natural water. The current approach favours a real water recording with continuous gain control.

## What slowed us down

- We sometimes substituted a recognizable outline for the desired material: a drawn V or radial band communicated the shape but did not feel like water.
- On the pale background, refraction alone was too subtle, but stronger white highlights could become broad, clipped, or beam-like. Visibility needed to come from a balanced combination of slope, highlight, shadow, and refraction.
- Changing wave shape, strength, lighting, and decay together made it hard to tell what improved or worsened a version.
- A visual fade is not necessarily a state reset: old wave data must actually expire or be cleared, otherwise later activity may reveal it again.
- Nominal duration did not guarantee useful visibility for that duration: damping and limited precision made some waves perceptually disappear early.
- Static screenshots helped identify shapes, but could not establish timing, propagation, or whether a ripple returned. Repeated visual testing in motion was essential.

## Vocabulary for describing visual effects

These are useful words, not prerequisites. A sketch, analogy, or “I don't know the term” is equally useful; the implementation can suggest the terminology.

| Term | Plain-language meaning | Example in this project |
| --- | --- | --- |
| Cellular / Voronoi-like pattern | Irregular neighboring regions, rather than evenly spaced grid squares | The idle water's changing network |
| Caustics | Concentrated light patterns produced by refraction or reflection | The bright network suggesting light on water |
| Height field | A map of relative surface height across an area | The simulated interactive water surface |
| Displacement / refraction | Shifting where the underlying image is sampled, making it appear bent | Text warping beneath ripples |
| Surface normal / slope | The direction the surface locally faces; approximated from nearby heights | Helps determine the distortion caused by a wave |
| Crest / trough | Raised and lowered parts of a wave | Light and dark parts of a disturbance |
| Wake / bow wave | Disturbance behind or around a moving object / the wave near its leading edge | Pointer movement acting as a boat |
| Amplitude / wavelength | Strength of a wave / spacing between successive crests | Stronger waves and wider-spaced ripples |
| Propagation / damping | A wave spreading / its energy decreasing | Waves move out and lose intensity |
| Envelope / easing | A time-based curve controlling how a property grows or fades | Smoothly fading each interaction over its lifetime |
| Shader | A small GPU program that calculates pixels or geometry | Rendering water, caustics, and refraction |
| Particle system | Many small, independently animated visual elements | Randomized click stars and glints |
| Layer (visual vs. state) | A composited visual contribution vs. a separately managed set of simulation data | Background/water/effects vs. per-click and grouped-wake fields |

## How to brief a future visual effect

Use everyday language first, then add whatever details you know. A useful brief covers:

1. **Intent:** What should it feel like? What is the metaphor?
2. **Trigger and behavior:** What starts it, where does it appear, what follows the pointer, and what happens after the interaction stops?
3. **Appearance:** Shape, color, contrast, texture, scale, and how strongly it affects existing content.
4. **Timing and variation:** Onset, duration, fade, speed-dependent behavior, and what should be randomized. Say whether a number is an exact constraint or a rough target.
5. **Boundaries:** What must stay unchanged, what must never happen, and whether physical accuracy or visual plausibility matters more.
6. **Acceptance:** A few repeatable actions and the result you expect from each, including desktop/mobile or reduced-motion requirements if relevant.

Example:

> When I move the pointer, it should feel like a small boat gliding over transparent water. I want a visible but soft wake on both sides of the path, plus some refraction of the text. Faster movement may leave a longer wake. Each disturbance should fade smoothly and be gone within three seconds, without returning on the next movement. The effect should remain visible over empty pale background. Avoid drawn V outlines, long white beams, and a continuously emitting point. Preserve the idle water and the existing color palette. Please first explain the proposed approach; implement only after we agree.

## A faster iteration loop

1. Save a named baseline or commit before experimentation, so a good version can be restored exactly.
2. Agree on the desired result and a short list of failure cases. Ask for explanation first if the underlying method is uncertain.
3. Change one dimension at a time—e.g. wave shape first, then visibility, then duration—and keep before/after versions when practical.
4. Test a single click, a slow movement, a fast movement, a turn, stopping, and a second interaction after the first has expired. Test over both text and empty background.
5. Share a short recording and describe *when* the problem appears: “at the start,” “after stopping,” or “when I move again.” Distinguish “too bright,” “too wide,” “too long,” and “too faint” where possible.
6. Ask the assistant to state what changed, what was actually verified, and what remains uncertain. A visual or numerical target should not be described as guaranteed unless it was measured.

For future work, a useful request is: “Preserve the current version as a baseline. Change only [one aspect]. Compare before and after using [gesture and timing]. Don't change [protected aspects]. Explain the result before we decide the next step.”
