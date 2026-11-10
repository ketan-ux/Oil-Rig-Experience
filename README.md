# PELICAN-9 · Storm Shift

A living 3D-pixel-art offshore oil rig in a single HTML file. Watch it as a diorama, tap the crew to see what each of them is doing, then take a job yourself in first person and finish your shift before the weather turns.

Everything is procedural — geometry, textures, shaders, animation and all of the audio. The only thing fetched is Three.js r170 from jsDelivr.

## Run it

Open `index.html` in a modern browser (Chrome, Edge, Firefox, Safari). It needs an internet connection the first time to load Three.js from the CDN. If your browser blocks module imports from `file://`, serve the folder instead:

```
npx serve .        # or: python3 -m http.server
```

## How it flows

1. **Diorama** — the page opens on the rig looping through calm → storm → Cat-5 → calm (3 minutes).
2. **Tap** — enters exploration: orbit, zoom, and tap any amber ◆ to focus on a worker, vehicle or oddity. The card shows who they are and what they are doing right now.
3. **✕ (top-left)** — opens the menu: Play, Diorama, Graphics, Controls, Audio.
4. **Play** — pick a role with ◀ ▶ while the camera shows that worker on the job, then start the shift.
5. **Shift** — first person. Work down the task list before the storm front arrives, then get inside the storm shelter. Two roles (Storm Bosun, Storm Electrician) start *in* the storm and work on a lifeline.

## Controls

| | Desktop | Touch |
|---|---|---|
| Diorama | drag to orbit · wheel to zoom · click a ◆ · ←/→ cycle · Esc back | drag · pinch · tap |
| Walk / look | WASD · mouse (click to capture) | left stick · drag elsewhere |
| Run / brace | Shift · Space | RUN · BRACE |
| Interact | E or mouse button (hold, tap or time it) | ACT |
| Lifeline (storm roles) | F to clip at a yellow post, hold F to unclip | CLIP |
| Hand signals, breakers | arrow keys or WASD | on-screen arrow pad |
| Pause · menu | Esc or P · ✕ | ✕ |

## The ten jobs

Roughneck · Crane Rigger · Welder · Gauge Inspector · Helipad Marshal · Winch Operator · Mud Engineer · Camp Boss · Storm Bosun · Storm Electrician.

The time you get is worked out from how long each task on the list should take on average, plus slack — so a long inspection round gets a later storm than a quick weld.

## Files

- `index.html` — the whole experience.
- `PROMPT.md` — the enhanced build prompt this was made from (the original diorama prompt plus exploration, menu, role select, first-person gameplay, storm work, new roles, sound design and quirks).

## Notes

- Settings, best results and the "days without incident" counter are kept in `localStorage`.
- In the browser console, `P9` exposes a few handles for tinkering, e.g. `P9.snap(2)` jumps the diorama to Cat-5 and `P9.W.speed = 4` fast-forwards it.
