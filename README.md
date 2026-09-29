# The Places That We Leave - Audio-reactive scene
 
A TouchDesigner scene built for the song **"The places that we leave"**.

**Built with:** TouchDesigner 099.2025.33070 · touchdesigner-mcp component 1.6.0
 
Everything is procedural: geometry, textures, sky and foliage are all generated inside the project. There are no external image or model assets.
 
---
 
## Requirements
 
- **TouchDesigner 2025.x** (built and tested on build 2025.33070). The Non-Commercial license works.
- A reasonably strong GPU. The scene runs at about 20–30 fps live; exports render frame by frame at a full 60 fps.
- **A song of your own.** The audio file is not included (see [Using your own song](#using-your-own-song)).
---
 
## Quick start
 
1. Open `theplacesthatweleave.toe`.
2. Select `/project1/audiofilein1` and point its **File** parameter to your audio file.
3. Open a viewer on **`/project1/scene/blockout_out`**. This is the final image (1280×720).
4. Press play.
The audio file is **locked to the timeline**, so scrubbing the timeline moves the audio, sun, lights and insects together.
 
 
## How it works
 
### Network
 
```
/project1
├── audiofilein1        the song (play mode: locked to timeline)
├── song_map            sections, lyric lines, bar numbers, scene cues (reference table)
├── audio/              audio analysis
├── scene/              everything visual
├── record_src / record_out / record_ctrl   full-length video export
└── fast_in / fast_out / fast_ctrl          sped-up copy of an exported video
```
 
### Audio: `/project1/audio`
 
**`bus`** (all channels 0–1):
 
| Channel | Source |
|---|---|
| `kick` | below 120 Hz |
| `body` | 120–800 Hz |
| `mid` | 800 Hz–4 kHz |
| `air` | above 5 kHz |
| `energy` | slow overall loudness |
| `beat` | kick transients |
| `songpos` | playback position, 0→1 |
 
**`drive`** holds smoothed channels made for motion:
 
| Channel | From | Smoothing (rise / fall) | Moves |
|---|---|---|---|
| `wind` | body | 0.8 / 2.2 s | leaf and grass sway |
| `gust` | beat | 0.2 / 0.9 s | flutter and ripples on kicks |
| `energy`, `kick` | bus | 0.3 / 1.2 s | sun glow |
 
### Timeline logic
 
These are text DATs used as Python modules.
 
- **`scene/sunset`:** the master clock.
  - `p()` gives the song position.
  - `sunEl()` gives the sun elevation, which sinks for the whole song.
  - `glow()`, `ambient()` and `porchFill()` give the light colour and strength.
  - The sky and all lights read from here, so they always agree.
- **`scene/lantern_ctrl`:**
  - `lamp()` sets the lantern intensity: a constant filament flicker plus a scripted blink.
  - `t()` gives the song time in seconds.
- **`scene/geo_util`:** a shared box builder with UVs, used by all procedural geometry.
- **`scene/tree_gen`:** the procedural tree recipe (branches and canopy leaf placement).
### Scene: `/project1/scene`
 
- **Camera (`cam`):** fixed, 90° field of view, with lens shift (`winy`) instead of tilt so vertical lines stay straight.
- **Sky:**
  - `sky_glsl` draws the sky gradient, clouds and sun. Its `uSunAudio` input lets the sun breathe with the music.
  - `sky_env` is a 360° copy of the sky, used for reflections and sky lighting.
  - `fog_glsl` adds distance haze into the sunset.
- **Geometry:** all built by script SOPs.
  - Porch: `deck`, `frame`, `ceiling`, `stairs`, `house`, `window`, `winframe`
  - Props: `chair` (rocking chair) and `blanket` (draped throw with fringe)
  - Lantern: `lantern` (iron body, chain, open glass bottom), `lantern_glass`, `lantern_bulb`
  - Landscape: `ground`, `path`, `ruts`, `poles` (with wires), `treeline`, `tree`
- **Foliage:**
  - `leaves`: about 56k instanced leaf cards
  - `grass`: about 38k instanced clumps
  - Both are animated by `foliage/leaf_anim` and `foliage/grass_anim`, and the `foliage` COMP has **Wind** and **Gust** parameters.
- **Life:**
  - `dust`: faint grey motes near the lantern
  - `moths` (5) and `flies` (9): they circle the lantern and die off one by one over the song
- **Materials and textures:**
  - `mats/` holds the PBR materials (`p_*`).
  - `tex/` generates every texture procedurally: wood, peeling paint, blanket, dirt, field, glass and leaf.
  - `tex/` also makes the three lantern **gobos**, which project the lantern frame's soft shadows.
- **Lights:**
| Light | Role |
|---|---|
| `sun_light` | low sunset key light with soft shadows; follows the sun |
| `sky_ibl` | environment light from the live sky |
| `lantern_spot`, `lantern_up`, `lantern_wall` | lantern light, shaped by the gobos |
| `lantern_light` | soft lantern fill |
| `bounce_deck`, `bounce_wall` | warm bounce light; follows the lantern |
 
- **Post chain:**
```
render → ssao → fog_glsl (over sky) → dust_add (+ dust_render) → bloom → blockout_out
```
 
 
## License
 
Do whatever you want with it.
 

