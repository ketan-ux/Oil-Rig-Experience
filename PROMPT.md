# PELICAN-9 · STORM SHIFT — enhanced build prompt

> This is the original "isometric oil-rig diorama" prompt, rewritten and extended.
> It keeps every rendering and layout rule of the original and adds: tap-to-focus
> exploration, a menu, a role-select screen, a first-person game with a weather
> deadline, storm work on a lifeline, four new job roles, a walk-in shelter and
> lower deck, fully synthesised audio, and a handful of quirks.
> `index.html` in this repository is a working implementation of it.

Create a complete, self-contained **single HTML file** that runs a living **3D Pixel Art (Pixel 3D)** offshore oil rig which can be enjoyed three ways: as a looping **diorama**, as a **tap-to-focus exploration**, and as a **first-person game** in which you work one crew member's shift and must finish before the weather turns.

Use **Three.js r170** via an ES-module `importmap` from `https://cdn.jsdelivr.net/npm/three@0.170.0/`. Use only Three.js core and the official `three/addons/` modules `OrbitControls`, `EffectComposer`, `RenderPixelatedPass`, `ShaderPass`, `OutputPass`. Fetch nothing else: every mesh, texture, particle, character, aircraft, shader, font choice and **sound** must be procedural.

Output **only** the complete, working HTML file.

---

## 1. Visual style: true 3D pixel art (not voxels)

Build clean low-poly meshes and turn them into pixel art with shading and post-processing.

1. **Diorama camera** — `THREE.OrthographicCamera` at true isometric elevation (≈ 35.264° pitch, 45° yaw). Snap the camera to the low-resolution texel grid in view space before each render so nothing shimmers while orbiting.
2. **First-person camera** — `THREE.PerspectiveCamera` (FOV 60–100°, default 78°) rendered through the *same* pixel pipeline, with exponential fog and a direction-based sky dome.
3. **Pixelation + ink** — `EffectComposer` → `RenderPixelatedPass` (`pixelSize` 3, adjustable 1–6; `normalEdgeStrength` 0.45; `depthEdgeStrength` 0.75). Compute depth edges from the *second difference* of depth converted to view-space units, so outlines are correct for both the orthographic and the perspective camera and never appear on flat sloped surfaces. Keep rain, glass, light beams and the sky out of the normal buffer (put them on a second layer).
4. **Dither + palette** — a `ShaderPass` after the pixel pass: ordered 4×4 Bayer dither indexed by low-res texel, then nearest-colour quantisation into a curated **32-colour** storm/daylight palette.
5. **Shading** — `MeshToonMaterial` with a 4-step nearest-filtered `DataTexture` gradient. All `CanvasTexture`s use `NearestFilter`, no mipmaps. One `BasicShadowMap` sun shadow.
6. **Canvas** — `image-rendering: pixelated; image-rendering: crisp-edges;`, device pixel ratio fixed at 1.
7. **Draw calls** — merge all static structure into one vertex-coloured batch; keep moving things as small separate groups.

---

## 2. The world (1 unit ≈ 2 m; a worker is 0.72 units tall)

Centre everything on a **28 × 6 × 28 ocean tile** resting on a dark basalt plinth. Do not leave decks empty: every area below needs props, signage and something to do.

**Ocean**
- Top surface: 112 × 112 grid driven on the CPU by a **6-component Gerstner solver** (so buoyancy, the normal pass and audio all agree) scaled by weather `S ∈ [0, 2]`: calm = concentric ripples radiating from the four legs; `S = 1` = ~1.1-unit rolling swells with whitecaps where the surface pinches; `S = 2` = ~2.3-unit rogue swells, foam sheets, horizontal spindrift, spray bursting up the legs, and periodic **green water sweeping the lower deck**.
- Cut-away sides: translucent teal walls whose top edge rides the wave profile; inside, a sea bed with rocks, kelp, leg footings, an export pipeline, a few fish and a sunken treasure chest.
- In first person, switch the same grid to an **open sea**: warp it out to ±110 units with fine cells near the rig, fade wave amplitude to zero at the rim, and continue with a flat fogged plane to the horizon. Hide the walls and plinth.

**Structure** (strict non-clipping coordinates)
- Four yellow legs at `(±5.5, y, ±5.5)` with X-bracing above and below the waterline, foam collars that ride the swell, and a marine riser under the derrick.
- **Lower deck** `y = 3.8`, 14 × 14: wellhead/BOP stack, two mud pits, vibrating shale shaker, two mud pumps with turning flywheels, mud-lab bench, barite pallet and hopper, a walk-in **generator room** with exhaust up the west flank, fire pump, compressor, chemical totes, workbench, sump pump.
- **Main deck** `y = 5.4`, 15 × 15, railed, reached by two exterior yellow stairways (south and north flanks).
- **Living quarters / storm shelter** at the north-west: two storeys, hinged storm door at `(-4.2, 5.4, -3.2)`, real window openings on the ground floor, upper-storey window grid that glows amber `#ffb347` with occupancy. The **ground floor is a real interior** you can walk into: mess table and benches, galley with coffee urn and oven, bunks, radio/control desk, lockers, cat bed, dartboard, and a “DAYS WITHOUT INCIDENT” sign driven by saved game results. Roof: HVAC fans, radome, radar, anemometer, flag.
- **Red lattice derrick** at `(0.5, 5.4, -1.5)` on a raised drill floor: crown block, travelling block that trips pipe in a four-phase cycle, rotary table, hanging tongs, doghouse, drawworks, monkey board with racked stands, ladder; pipe rack to the east.
- **Pedestal crane** at `(4.8, 5.4, -4.5)`: glazed cab, luffing lattice boom, hook block; shuttles a crate between two painted laydown squares and stows north-east in weather. Its sweep never crosses the derrick or helipad.
- **West davits** at `(-7.2, 5.4, 0)`: twin arms, winch house with crank wheel, cradle at `y = 4.6`, falls down to the support boat.
- **Flare boom** from the north-west corner out to `(-12.5, 10.5, -12.5)`: layered flame, point light, wind-driven smoke; the flame lies almost 70° over in a hurricane.
- **Cantilevered green helipad** at `(10.5, 6.2, 10.5)`: octagon radius 3.4 on a yellow truss from the south-east leg, hazard border, white “H”, safety net, windsock, fire monitor, stair from the deck corner, perimeter LEDs (green clear / amber occupied / flashing red in Cat-5). **Zero overlap** with deck, quarters or crane sweep.
- Also on the main deck: fuel tanks and bund, valve manifold, comms mast, deck piping (the welders' job), container stack, drums, tool crib, cable spool, lifeboat, floodlight poles, life rings, painted walkways and a muster square.

**Data the gameplay needs** (build it alongside the geometry)
- Walkable surfaces (rectangles, ramps, the pad disc) and solid boxes for collision; railings are implied by surface edges.
- A waypoint graph for crew path-finding, including both stairways and the helipad stair.
- **Lifeline anchors**: ~16 yellow posts linked by deck jacklines, on both decks, the stairs and the pad.
- Named points of interest for every task.

---

## 3. Helicopters (≥ 16 child meshes each, no box-only shapes)

Shared build: tapered nose cone, wrap-around tinted canopy with frame struts, cabin with sliding door, turbine cowling with twin exhausts, tapered tail boom, angled fin, stabilisers, 4-blade main rotor on a tilted swashplate with a blur disc, 2-blade tail rotor, red/green/white navigation lights.

- **Scout (coast-guard red/white, skids)** with an under-nose **searchlight gimbal** at local `(0, -0.45, 1.3)`. Attach a `SpotLight` and a translucent `ConeGeometry(2.2, 14, 16, 1, true)` to the gimbal; pre-transform the cone with `translate(0, -7, 0)` then `rotateX(-π/2)` so local +Z runs down the beam; each frame aim `gimbal.lookAt(spotTarget)` at a figure-eight point on the sea or deck with `y ≤ 5.4` and copy it to `spot.target`. The beam must never point at the sky. Behaviour: calm = high wide patrol, light off; storm = banked elliptical patrol, light sweeping (in first person it tends to find *you*); `S > 1.45` = climb over the derrick, land on the free pad, idle the rotors, **pilot sprints to the shelter**; `S < 1.40` = pilot runs back, spools up, rejoins the patrol.
- **Crew transport (yellow/navy, twin engine, sponsons, wheels, mechanised door)**: flies only when `S < 0.35`. Lands, **four arrivals** step down one by one and walk to their stations while **four off-shift hands** who walked up earlier board; then it leaves. If weather turns mid-swap it aborts cleanly (unboarded hands keep or hand back their jobs).

---

## 4. Crew: 16 hands, 10 roles, role-specific animation

Each worker is a jointed low-poly figure: head with face, role hat, torso with hi-vis stripes / life vest / harness / apron, shoulders + elbows, hips + knees, and tools that attach to the hands. Animation is a library of **named poses blended at ~15 Hz**, never a generic arm swing. When focused on, the action must be readable.

| Role (hat) | n | Station | Signature actions |
|---|---|---|---|
| Derrick roughneck (red) | 2 | drill floor | throw the tongs, whip the spinning chain, haul the elevators, signal “hoist”, hose the floor, wipe brow — keyed to the pipe-tripping phase |
| Crane operator (yellow) | 1 | seated in the slewing cab | feathers two levers only while the crane moves |
| Crane rigger (yellow) | 1 | cargo deck | real hand signals: hoist / swing / lower / stop; steadies and slings the load |
| Structural welder (blue + flip visor) | 2 | deck piping | crouch and weld with visor down (cyan sparks, flicker light, crackle), visor up to inspect, chip slag, change rod |
| Gauge inspector (white + clipboard) | 2 | tanks → manifold → mast | walk with clipboard, tap gauge twice, write, point-and-call, shade eyes at the radar |
| Helipad marshal (green, ear defenders, glowing wands) | 2 | pad stair | move-ahead / hover / land / hold / lift-off wand signals tied to the transport's flight phase |
| Winch crew (orange life vests) | 2 | west davits | crank the winch, ease the brake, lean over the rail, radio, direct the boat |
| **Storm bosun** (purple, harness) *new* | 1 | cargo deck | coil line, ratchet chain binders, tug-test lashings; **works through the storm on a lifeline**, braces in gusts |
| **Storm electrician** (aqua, headlamp) *new* | 1 | panels and junction boxes | screwdriver + torch at a panel (with the occasional zap), kneel at a junction box; **works through the storm** |
| **Mud engineer** (brown) *new* | 1 | lower deck | dip and sniff a sample, stir the pit, bench test, spin a valve |
| **Camp boss / cook** (toque) *new* | 1 | stern rail, galley | fishes off the stern on his break (mostly boots), sips coffee, feeds the gull |
| Scout pilot | 1 | cockpit | exits, sprints, shelters, returns |

**Behaviour (finite-state, driven only by `S`)**
- `S > 0.30`: the boat motors under the davits; the winch crew crank it out of the sea, with buoyancy blending smoothly into rigid suspension and locking at the cradle.
- `S > 0.45`: horn; regular crew stow tools and run for the door one hand on their hard hats; the door swings; each takes a seat, bunk or window inside and the windows glow. The cook and the cat go first.
- `0.45 < S < 1.40`: **bosun and electrician stay out**, each visibly roped to the nearest anchor, doing storm jobs. Above 1.40 they come in too.
- `S < 0.40`: all-clear chime; crew file back out; the boat is lowered.
- Never teleport anyone except behind a full-screen fade.

---

## 5. Quirks (keep them small and alive)

- **Gasket**, the rig cat, strolls the muster area and is always first through the door — the real weather forecast.
- **Gerald**, a seagull who supervises the fishing and takes his fee in bait; a small flock circles in fair weather and leaves before the storm.
- **Sir Quacksalot**, a rubber duck that rides out every hurricane; a belled marker buoy that rings with the swell.
- Once per Cat-5 something with suckers surfaces at the edge of the tile. HR's position is that it is a wave.
- A “NO SMOKING (we mean it)” sign by the flare; radio banter from named crew during shifts; pixel emote bubbles; the incident counter resets when you get caught on deck.

---

## 6. Weather: one variable, seven phases, 180 s

`S ∈ [0, 2]` drives everything. Auto mode follows a seamless loop, `T = 180 s`, each ramp a C² quintic:

| τ | Phase | S |
|---|---|---|
| 0.00–0.36 | Calm, shift change | 0 |
| 0.36–0.44 | Storm approaching: boat hoisted, crew shelter, searchlight on | 0 → 1 |
| 0.44–0.62 | Heavy storm, storm crew on lifelines | 1 |
| 0.62–0.68 | Escalation: scout lands, pilot shelters | 1 → 2 |
| 0.68–0.82 | Cat-5: lightning, torrential horizontal rain, green water | 2 |
| 0.82–0.90 | De-escalation: scout lifts off | 2 → 1 |
| 0.90–1.00 | Clearing: boat lowered, crew return | 1 → 0 |

Run the simulation on continuous time (never reset wave or animation clocks) so the loop has no cut. On any mode change (`Auto`, `Calm`, `Storm`, `Extra Storm`, `Manual`) ease `S` with a critically damped exponential (τ ≈ 2.8 s) and keep separate damped sub-states for waves, rain, wind, sky, hoist, shelter occupancy and scout landing. Sky: golden/cyan → `#141b29` → `#070a12` with branching lightning in Cat-5.

---

## 7. App flow

1. **Open → attract.** The diorama plays with a title and “TAP TO EXPLORE”. No sound until the first tap.
2. **Tap → explore.** Unobtrusive: drag to orbit, pinch/scroll to zoom, a hint that fades. Small amber diamonds float over everything tappable (16 crew, pilot, both helicopters, boat, crane, derrick, flare, cat, gull, duck, buoy…). **Tap one to focus**: the camera eases in and follows it, a vignette darkens the rest, and a card shows the role, the person's name, a **live line describing what they are doing right now**, a short description, previous/next, and “Play this role”.
3. **✕ (top-left) → menu.** Buttons: **Play**, **Diorama**, **Graphics** (quality preset, pixel scale, outlines, dither, palette, shadows, rain density, FOV, look sensitivity, head bob, storm shake), **Controls** (desktop and touch tables), **Audio** (master / weather / machines / music, mute).
4. **Play → role select.** One role in focus at a time on the live diorama, **left/right to change**, the camera on that worker performing their job. The weather eases to calm or storm to match the role. Card: title, shift type, brief, numbered task list, a quirk, time until the front, difficulty, best result.
5. **Start shift → first person**, behind a fade.
6. **✕ always returns to the menu.** `Esc`/`P` pauses a shift.

Diorama HUD (top-right, collapsed to a 28 px `⚙ HUD` pill with a `CALM / STORM / CAT-5` badge; expanded panel 225 px wide): weather mode, `S` slider, operations read-out (`CREW`, `BOAT`, `HELIS`), pixel scale, outlines, dither, auto-orbit, markers, and the loop timeline (play/pause, 0.5×–4×, scrubbable phase bar).

---

## 8. First-person core loop: finish the list before the weather changes

**Controller.** Walk 1.95 u/s, run 3.1 u/s, eye height 0.64, step-up 0.22, circle-vs-box collision, surface-following on stairs and ramps, head bob, footsteps that change with the surface and the rain. You replace one worker of the chosen role; everyone else keeps living around you. A small first-person view-model (sleeves, gloves, the role's tool or the thing you are carrying) animates with each action.

**Tasks** run in order. A floating amber marker and a screen-edge arrow with distance show the next one. Five interaction types, themed per job:
- **Hold** — keep the button down while a bar fills (read a gauge, set slips, brew coffee).
- **Timing** — stop a sweeping cursor in a shrinking green zone (weld beads, tong bites, dropping the falls on the swell); gusts speed the cursor up.
- **Crank** — tap fast or hold (winch, ratchet, valve, pull-start).
- **Signal / sequence** — press the shown arrows (crane and marshal hand signals, which the real crane and helicopter obey; “memorise” variants for breakers).
- **Carry** — pick up, haul (slower), put down (drums, barite sacks, the cat, a tray of coffee that spills if you run).

**The weather deadline.** Each task carries an average time estimate including the walk. For a fair-weather role the front arrives at `Σ estimates × 1.3 + 18 s`, then ramps `S` 0 → 1.1 over 34 s. For a storm role the shift *starts* at `S = 1` and Cat-5 arrives at `Σ × 1.25 + 26 s`, ramping over 30 s. A forecast bar shows the countdown. Finish early and the front arrives early.

**Hiding.** When the ramp starts: horn, banner **“WEATHER IS ABOUT TO GO HAYWIRE — take shelter soon”**, the bar pulses, the cat bolts. The last task is always *get inside the storm shelter*. If the ramp completes while you are outside you get a short red countdown with a heartbeat; run out and you are **caught on deck**. Reach the shelter with the list done to win (2–3 stars by time to spare); reach it with jobs undone for a one-star “safe… but”.

**Storm work on a lifeline.** Storm roles cannot leave the doorway until they clip to the yellow post beside it. The line is ~12 m: walk to the next post and clip again to move on. Wind slows you upwind and shoves you downwind; gusts are telegraphed by a gust meter, sound and screen streaks; **brace** to hold your ground. Unclipped near an edge or on a stair in a gust and you are warned, then you go **overboard**. Green water on the lower deck knocks you flat if clipped and sweeps you off if not. A headlamp follows your view.

**Shifts**

| Role | Weather | Job list |
|---|---|---|
| Roughneck | fair | grab the wrench · set the slips (hold) · bite the tongs ×3 (timing) · signal the driller · spin in the next stand (crank) · hose the floor |
| Crane rigger | fair | sling the crate · signal HOIST / SWING / LOWER (the crane obeys) · chain it down · stow a loose drum · ratchet lashings |
| Welder | fair | rods · weld the deck flange (timing, visor drops) · chip slag · weld the mud line on the lower deck · inspect |
| Gauge inspector | fair | five gauges across two decks, bleed the one in the red, file the round at the control desk |
| Helipad marshal | fair | shoo Gerald off the pad · wand the transport in (it waits for your signals) · chock the wheels · wave the off-shift crew aboard · wand it away · pad lights to amber |
| Winch operator | fair | radio the boat alongside · drop the falls on the swell · winch her up (the boat really rises with the bar) · pin both cradle locks · lash the gripes |
| Mud engineer | fair | sample · funnel test · three sacks of barite to the hopper · mixing valve · pump pressure |
| Camp boss | fair | brew · deliver coffee to three stations without spilling · find the cat and put her to bed · rescue the pie |
| **Storm bosun** | storm | clip on · bind the sliding crate · slam the crib door between gusts · catch the runaway drum · double the lashings · home before Cat-5 |
| **Storm electrician** | storm, lights out | clip on · reset the breakers (memory) · pull-start the generator on the lower deck · swap the mast fuse · relight the helipad so the scout can land · home before Cat-5 |

---

## 9. Controls

| | Desktop | Touch |
|---|---|---|
| Diorama | drag orbit · wheel zoom · click to focus · ←/→ cycle · Esc back | one-finger orbit · pinch zoom · tap to focus |
| Walk / look | WASD · mouse (pointer lock, drag fallback) | left stick · drag anywhere else |
| Run / brace | Shift · Space | RUN toggle (or push the stick far) · BRACE |
| Interact | E or mouse button | ACT |
| Lifeline | F (hold to unclip) | CLIP |
| Signals | arrows / WASD | on-screen arrow pad |
| Pause / leave | Esc or P · ✕ | ✕ |

Detect touch, show the touch layer only while playing, respect safe-area insets, and keep every HUD element clear of the centre of the screen.

---

## 10. Sound design (WebAudio only, no samples)

- **Weather bed**: sea rumble and wash, wind with gusting band-pass and a rigging whistle, rain hiss plus patter — all scaled by `S`, low-passed and replaced by room tone when you are inside the shelter. Thunder with distance delay and a reverb tail; wave crashes.
- **Positional machines** with distance falloff and stereo pan: generator hum, rotary table, flare roar, rotor chop for each helicopter, boat engine, winch, crane, welding arc. In the diorama the listener moves with the zoom, so focusing on a welder brings up the crackle.
- **Foley**: footsteps per surface (wet in rain), carabiner, ratchet, hammer on steel, valve squeak, breaker thunk, zap, pour, oven bell, splash, storm-door, gull, cat, buoy bell, PA horn and all-clear chime, garbled radio, heartbeat for the final countdown.
- **Music**: a small original 6/8 concertina shanty in fair weather that gives way to a low beating drone as the storm builds.

---

## 11. Technical requirements

- One file: HTML, CSS, GLSL and ES-module JavaScript.
- 60 FPS target: `InstancedMesh` for rain, spray, sparks, smoke, lightning and markers; pre-allocated temporaries; no per-frame allocation in hot loops; quality presets that scale the water grid, rain count and shadows.
- Resize updates both cameras, the composer and the pixel pass.
- Persist graphics, audio, last role, best results and the incident counter in `localStorage` (guarded).
- If Three.js cannot be fetched, say so on the loading screen instead of showing a blank page.

## 12. Acceptance checklist

- [ ] Opens on the diorama; first tap enters exploration and starts audio.
- [ ] Every worker can be focused and the “now” line matches the animation.
- [ ] ✕ → menu; Graphics, Controls and Audio all work and persist.
- [ ] Role select cycles ten roles with a live preview of each.
- [ ] Each shift can be completed, failed by being caught out, and (storm roles) failed by going overboard.
- [ ] Storm crew visibly work in the rain on lifelines in the diorama too.
- [ ] Weather mode changes never snap; the auto loop has no visible seam.
- [ ] Playable with keyboard + mouse and with touch only.
