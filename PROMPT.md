You are an expert game developer. Build a complete, polished, playable
browser game as a SINGLE index.html file (inline CSS + JS). It must run
by double-clicking the file and when hosted as static files on GitHub
Pages/Netlify. No build step, no backend, no external assets.

## Game: "<TITLE YOU INVENT>" — a top-down survivors-like roguelite
with procedurally generated worlds. Fast, readable, endlessly replayable
runs of 10-20 minutes (plus an endless mode). Tone: neon/synthwave,
juicy and satisfying.

## Hard technical constraints
- Vanilla JS (ES2020+), HTML5 Canvas 2D, CSS. No frameworks.
- No external imports, with ONE exception: Trystero (live co-op section
  below), loaded via dynamic import() from a pinned-exact-version ESM CDN
  URL inside try/catch, so the game is fully functional if it fails
  to load or is offline.
- All graphics procedural (shapes, gradients, glow, particles). All audio
  via WebAudio (procedural SFX + simple generative music loop), with mute
  toggle and volume slider.
- Responsive: desktop (WASD/arrows) and mobile (virtual joystick,
  touch-friendly UI). Handle devicePixelRatio, resize, and tab
  visibility (auto-pause).
- Performance: 60 FPS with 500+ enemies and 1000+ projectiles/gems.
  Fixed-timestep simulation (60 Hz) decoupled from rendering, object
  pooling, spatial hash grid for collisions, no per-frame allocations in
  hot paths, and render only what is on screen.
- ALL gameplay randomness uses a seeded PRNG (mulberry32) derived from
  the run seed; cosmetic randomness (particles, shake) uses a separate
  unseeded PRNG. The same seed + same inputs must reproduce the same
  world and wave schedule.
- Clean architecture in clearly commented sections: Core (loop, input,
  RNG, math), World Gen, Entities, Systems, Content data (data-driven
  weapons/enemies/upgrades/biomes as plain objects), UI/HUD, Audio, Save,
  Net, Main.

## Procedural world generation (a core feature)
- Infinite, chunk-based world (e.g. 32x32-tile chunks) generated lazily
  around the player(s) and unloaded when far away. Chunk content is a
  pure function of (runSeed, chunkX, chunkY), so any peer can regenerate
  it identically without network data.
- Use seeded value/simplex noise (implement it yourself) to produce:
  terrain zones, obstacles (rocks, pillars, walls, water/lava that block
  or slow movement), decorative props, and hazard patches (damage
  fields, slow fields, boost pads).
- Biomes: at least 5 distinct biomes (own palette, obstacle types,
  hazards, enemy roster weighting, and ambient music mood). The run
  progresses through biomes: every ~4-5 minutes a boss spawns, and
  defeating it opens a portal to the next biome. After the final biome,
  endless mode cycles biomes with ever-increasing scaling.
- Points of interest placed by the generator at deterministic positions:
  shrines (risk/reward buffs or curses), treasure chests, vaults guarded
  by elite enemies, healing springs, and rare "challenge zones" (e.g.
  survive 30s in a ring for a reward).
- The generator must guarantee the spawn area is clear and that
  obstacles never trap the player (verify connectivity near spawn and
  keep obstacle density bounded). Enemies must path sensibly around
  obstacles using simple steering/avoidance (no full A*).
- Obstacle collision uses the same spatial hash as other entities.
- Show the run seed on the pause and results screens, and let players
  enter a custom seed on the character select screen.

## Core gameplay
- Player auto-attacks; you move. Enemies swarm from outside the screen,
  escalating in waves, and are biome-themed.
- At least 8 enemy types with distinct behaviors (chaser, fast swarmer,
  tank, ranged shooter, splitter, charger, healer/buffer, exploder) plus
  bosses with telegraphed multi-phase attack patterns (one signature
  boss per biome).
- XP gems -> level up -> choose 1 of 3 random upgrades. At least 10
  weapons (5-8 levels each), 15+ passives, and weapon evolutions/
  synergies (max weapon + specific passive = evolved weapon).
- Continuous difficulty scaling (count, HP, speed, elite modifiers).
- Pickups: health, magnet, bomb, chests with bonus upgrades.
- Game feel: screenshake, hit-flash, damage numbers, particles, hit-stop
  on big kills, smooth camera, juicy level-up. All toggleable in settings
  (reduce motion, colorblind-friendly palette, show FPS).

## Progression & gamification
- Meta-currency earned each run -> permanent upgrade shop (stats,
  starting weapons, new characters, rerolls/banishes).
- 5+ characters with unique starting weapons/passives, unlocked via
  achievements.
- 25+ achievements with toasts and a gallery screen.
- Mutators: unlockable run modifiers for extra difficulty and bonus
  currency.
- Persistent stats screen, plus a prestige/ascension system that resets
  meta-progress for permanent multipliers.
- Save to localStorage with version number, try/catch, migration stub,
  and Export/Import as a text string.

## Async multiplayer (REQUIRED, no networking)
- Daily Challenge: run seed derived from the UTC date, identical for all
  players, with a fixed mutator set and fixed character option. Track
  personal best per day.
- Result sharing: at run end, generate a shareable URL (base64 in the
  hash) containing seed, character, score, time, kills, and final build.
  Opening such a link shows a "Challenge from a friend" screen with
  their result and a button to play the same seed.
- Friend comparison board stored locally from imported result links
  (per seed), plus a "ghost" display: a simple recorded path/timeline
  of the best run for a seed, shown as a faint marker during play.
- "Play this seed" and "Copy seed" buttons everywhere a seed is shown.

## Live co-op (OPTIONAL SECTION, 2-4 players, serverless)
- Use Trystero with a room code. UI: "Host Game" -> short room code +
  copyable invite link (#room=CODE); "Join Game" -> enter code; invite
  link auto-joins.
- HOST-AUTHORITATIVE: host runs the simulation. Clients send input state
  at ~20 Hz; host broadcasts compact snapshots at ~15-20 Hz (packed
  typed arrays, quantized positions, only entities near each player).
  The host sends the run seed once; clients regenerate world chunks
  locally from it. Clients interpolate (~100 ms buffer) and run
  prediction for their own movement only.
- Each player picks their own level-up upgrades through a non-blocking
  picker with a timeout that auto-picks. Shared XP pool, revive by
  proximity, enemy count/HP scaled by player count, ping display,
  reconnect restoring the player slot, late join allowed.
- Abstract networking behind a small Net interface (host, join, send,
  onMessage) so game logic never touches Trystero directly. Include a
  "Local test" mode via BroadcastChannel for two-tab testing.
- On connection failure show a clear error and offer solo play; note in
  the UI that strict networks may block peer connections.
- Cosmetic emote buttons only (no free text).

## UX
- Title -> character select (with seed entry) -> run -> results ->
  meta shop. Pause menu with settings (audio, screenshake, reduce
  motion, palette, FPS). First-run skippable tutorial. Clear HUD: HP,
  XP bar, timer, kills, weapon icons with levels, and off-screen arrows
  for bosses, chests, and portals. Keyboard shortcuts shown on screen.

## Quality bar
- No console errors, no TODOs, no unimplemented menu items. Every
  button works. Robust to window blur, resizing, corrupt saves, and
  network drops.
- Balance so a first run is winnable-ish at ~8-10 minutes while skilled
  play can go much longer.

## Output
Return ONLY the complete index.html in a single code block, followed by
a short list of: (1) how to run locally and deploy to GitHub Pages/
Netlify, (2) how to test async sharing and multiplayer, (3) known
limitations.