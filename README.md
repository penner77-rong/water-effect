# Water Effect

A full-screen interactive water experiment built with HTML, JavaScript, WebGL, and two local audio files. Current documented baseline is November 5, 2026.

## Run

Open `index.html` in Chrome, or serve this directory with `python3 -m http.server 8000` and visit [localhost](http://localhost:8000). The project has no build step.

Move the pointer to create a wake. Click for a stronger radial ripple and star glints. Enable Sound for stirring and drop audio.

## Customize

Open Ripple controls using the bottom-right button.

- Background gradient supports draggable color stops, positions, opacity, reverse, and rotation.
- Background text supports multiline content, local fonts, size, weight, and color.
- Browse installed fonts requests browser permission. You can also enter an installed family name. Each visitor uses their own available fonts.
- Background image uploads fill the viewport. Reset reveals your gradient and text.
- Ripple lighting controls shadow color and opacity, highlight brightness, and refraction.
- Other sliders control click size and lifetime, wake strength, spread, slow V angle and lifetime, edge activity, and audio volume.

Changes apply immediately and last until reload.

## Files and context

- `index.html` is the website entry point.
- `audio/water-stirring.mp3` and `audio/water-drop.mp3` supply interaction audio.
- [Design and implementation context](WATER-EFFECT-NOTES.md) records the current baseline and earlier iterations.
- [Product requirements](WATER-EFFECT-PRD.md) records intended behavior and remaining gaps.
- [GitHub publishing guide](GITHUB-PUBLISHING.md) explains uploading and enabling the live demo.

## Known limitations

Desktop Chrome is the primary checked browser. Installed font enumeration requires browser support and permission. Fonts are rendered from the visitor's device. Reduced-motion mode uses a single static render. Mobile interaction and cross-browser performance need further review. Some waves retain an embossed appearance under certain settings.

## Audio provenance

The project notes record the source recordings as CC0.

- Stirring recording by Daen23. [Source](https://freesound.org/people/Daen23/sounds/431627/)
- Drop recording by mattfinarelli. [Source](https://freesound.org/people/mattfinarelli/sounds/533146/)
