# Combat Design and Development Plan

**Status:** Phase 0. This is a draft for discussion. Anything marked **OPEN** needs the user's answer before work depends on it (§12).
**Last updated:** 2026-11-05
**Requirements:** `docs/combat-director-brief.md` holds the user's brief. "Brief §N" means section N of that file.

## 1. Where the project stands

- **Greenfield:** as of 2026-11-05 there is no code, art, animation or audio. There's nothing existing to audit, so everything below is a proposal for the foundation.
- **Engine:** Godot. The user has it installed locally.
- **Assets:** the user generates them with GPT-6 SOL, from prompts Claude writes.

## 2. Core principle

**Weight comes from the moment of contact and the enemy's reaction, not from slow player animations.**

In all four references, attacks start fast. The weight comes from hit-stop, the enemy's reaction, sound and camera all landing on the same frame. That's how combat can be weighty and responsive at once, instead of trading one for the other. Every system here is built around one synchronized contact frame.

Death's Door and Tunic are 3D games. Their smooth rotation and blended transitions come free with 3D skeletal animation. 2D sprites can't blend between animations: cross-fading two sprite frames just looks like ghosting. That fluidity has to be designed in instead (§6).

## 3. Engine and technical foundation

- **Engine and language:** Godot 4 with typed GDScript. Target 4.7.x; the minimum is 4.3, the version that added 2D physics interpolation. **OPEN:** which version the user has installed.
- **Fixed 60 Hz simulation:** combat logic runs in `_physics_process` at 60 ticks per second, and physics interpolation keeps rendering smooth on high-refresh screens.
- **Frame data:** counted in frames at 60 fps. Frame 0 is the tick an action starts.
- **Data-driven:** attacks, combos, impact profiles and enemy profiles are custom `Resource` files (`.tres`). They use `Curve`s for easing and can all be tuned in the Inspector without code changes.
- **Hit-stop uses per-entity time, never `Engine.time_scale`:** that would also freeze camera shake, input buffering and UI.
- **Proposed resolution:**
  - 1920×1080 reference resolution, scaling up to 1440p and 4K.
  - Art painted at 2×.
  - The character is about 1/8 of screen height. Confirm this along with the art direction.

## 4. Movement (top priority)

Starting targets, all tunable:

- **Locomotion:**
  - 360° analog movement (8-way on keyboard).
  - Top speed in about 5 frames, full stop in about 4.
  - A short skid on hard reversals at speed, so it feels deliberate rather than floaty or icy.
- **Facing:** 8 sprite directions, with about 10° of hysteresis at each boundary so the sprite doesn't flicker when the stick sits near a diagonal.
- **Dodge:**
  - Starts moving on frame 0 and is invulnerable for roughly frames 0 to 11.
  - Travels about 3 body-lengths, then a short recovery.
  - Attacking late in the dodge gives the dash-slash.
  - A short cooldown keeps it a decision rather than spam.
  - The player passes through enemies while dodging.
- **Attacks move the player:**
  - Every attack has a lunge (a distance plus an easing curve).
  - Running attacks keep part of their momentum.
  - The lunge stops at ideal spacing when an enemy is in front, instead of sliding through it.
- **Collision:** a small collider at the feet, sliding along walls, and corner correction that nudges the player around corners instead of snagging.

## 5. Combat design

### 5.1 Prototype moveset

| Attack | Description |
|---|---|
| A1 horizontal cut | Forehand, sweeping right to left across the body. Fast. |
| A2 reverse cut | Backhand, left to right and rising, with the body counter-rotating. |
| A3 diagonal finisher | High right to low left, stepping or leaping in. Heavy. |
| Dash-slash | Attack out of a dodge. Starts from A1-like timing with a longer lunge. |

### 5.2 Frame data (starting values, frames at 60 fps)

| | A1 | A2 | A3 |
|---|---|---|---|
| Wind-up, fresh / chained | 5 / 3 | 5 / 3 | 10 / 7 |
| Active (hit) frames | 3 | 3 | 4 |
| Recovery | 14 | 14 | 24 |
| Next attack can start from | frame 11 | frame 11 | none (end of combo) |
| Can dodge out from | frame 8 | frame 8 | frame 20 |
| Can walk out from | frame 16 | frame 16 | frame 30 |
| Hit-stop | 4 | 4 | 8 (kills +3) |
| Reaction power (§5.7) | 1 flinch | 1 flinch, plus push | 3–4 (knockback, knockdown or launch) |

- **Chained attacks:** a chained attack is started from the previous attack's combo window. It enters partway into its wind-up: A2 at its frame 2, A3 at its frame 3. That works because the previous attack's follow-through already served as the wind-up (§6.2).
- **Hit-stop pauses the frame count.** An attack's frame count stops during hit-stop, so cancel windows land at the same point in the animation whether the attack hits or misses.
- **Combo reset:** the combo goes back to A1 if the next attack hasn't started by the end of recovery plus a grace of about 8 frames (tunable).
- **Rhythm:** on a miss, it's about 29 frames (roughly 0.5 s) from the first press to the finisher's contact. That makes a "da-da… DUM" beat.

### 5.3 Input and aiming

- **Instant acknowledgment:** the wind-up pose shows on the frame the attack starts.
- **Input buffer:** about 10 frames for both attack and dodge. If both are buffered, dodge wins.
- **Re-aiming:** the attack's direction is read when the attack starts, not when it was buffered, so the player can re-aim between hits.
- **Aim:** the exact analog angle (**OPEN:** gamepad stick vs. mouse), plus soft aim-assist toward the best target in a cone of about ±35° at short range.
- **Sprite vs. hit angle:** the body sprite uses the nearest of 8 directions. The slash effect and the hitbox use the exact angle.
- **Mirrored facings:** W, SW and NW also mirror the attack's sweep direction, so the slash and hitbox always match the sprite.

### 5.4 Hit detection

- **Swept arc:** the hitbox is a swept arc on the ground plane. Each active tick covers everything between the blade's previous angle and its current one, so fast swings can't skip past an enemy. Thrusts and overhead strikes use a forward capsule instead of an arc.
- **One hit per swing:** each target can be hit only once per swing, tracked by a per-swing hit record.
- **Close calls favor the player:** enemy hurtboxes are slightly generous and the player's hurtbox is slightly tight.

### 5.5 Hit-stop and impact

- **Durations (starting values):**
  - Light hits: 4 frames.
  - Finisher: 8 frames.
  - Kills: +3 frames.
  - The last enemy of an encounter also gets a short slow-motion, about 0.3× speed for about 0.25 s.
- **During hit-stop:**
  - The attacker holds the contact frame.
  - The victim holds its hurt frame and shakes about ±2–3 px.
  - The impact sound plays immediately.
  - Input keeps buffering and camera shake keeps running.
- **Impact, then release:** knockback starts when hit-stop ends.
- **Several enemies, one swing:** use the longest hit-stop, not the sum.
- **Scope, set per impact profile:**
  - Local: only the attacker and victim freeze.
  - Global: everything freezes except camera, UI, audio and input.
- **Camera and rumble:** a directional kick in the hit direction plus a small trauma-based shake, stronger for finishers. Controller rumble scales with the impact.
- **Hit flash:** 1–2 frames of white on the victim, then a brief tint.
- **Effects:** sparks are oriented along the slash, and their strength communicates the attack's weight. They shouldn't just fill the screen.

### 5.6 Knockback

- **Direction:** a blend of the attack's forward direction and the blade's tangent at contact. A1, a right-to-left cut, pushes enemies forward and to the left, so the push reads as caused by the blade.
- **Speed:** knockback force divided by the enemy's mass, decaying with friction.
- **Launch:** uses a fake height axis. The sprite rises while its shadow stays on the ground, then it lands with a bounce and dust.

### 5.7 Reaction resolution

**Inputs:**

- From the attack: reaction power and poise damage.
- From the enemy: armor, current poise and mass.
- From the hit: which side it came from (front or back, relative to the enemy's facing) and whether it is lethal.

**Steps:**

1. **Lethal hit:** play a death reaction, chosen per enemy type and scaled by the attack's power.
2. **Poise breaks:** play the poise-break stagger, which is long and leaves the enemy open to punishment.
3. **Otherwise:** compute effective power = reaction power − armor.
   - If it's 0 or less, it's an armored hit: flash, shudder and sparks, with no interruption.
   - If it's higher, play the flinch, stagger, knockback or knockdown/launch.
4. **Pick the variant:** front or back, from the hit side. Left and right come from mirroring.

Reaction power levels: 0 none, 1 flinch, 2 stagger, 3 knockback, 4 knockdown/launch.

### 5.8 Prototype enemies

| Enemy | Physical profile | Reactions |
|---|---|---|
| Light skirmisher | Armor 0, low mass, low poise | Flinches on every hit, which interrupts its attacks. A3 knocks it down or launches it. On death it flies back and tumbles. |
| Armored brute | Armor 2, high mass, high poise, super armor while attacking | Light hits are armored hits (flash, shudder, sparks, metallic clang) while its poise drains. A poise break gives a big stagger and a punish window. A3 pushes it a little. On death it collapses heavily with a ground shake. |
| Training dummy (arena only) | Never attacks, infinite health | Reacts like the light enemy and shows each hit's data on screen. |

Both real enemies have readable telegraphs (a clear wind-up pose plus a glint) and leave a punish window after they attack.

## 6. Animation strategy

### 6.1 Phases

Every attack runs anticipation → acceleration (smear) → contact → follow-through → recovery. The acceleration through the middle of the swing comes from timing: the anticipation pose holds, one very fast smear frame follows, then the contact pose.

### 6.2 Flow without blending: each follow-through is the next wind-up

Sword positions use a clock face on the ground plane: 12 is the facing direction, 3 is the character's right and 9 is their left. The character is right-handed.

- **A1:** winds up with the sword back at about 4 o'clock, sweeps through 12, makes contact around 10, and ends at about 8–9 on the left side. That end pose is A2's wind-up.
- **A2:** from about 8–9, a backhand through 12 that rises as it goes, ending high on the right (about 2 o'clock, above shoulder height). That end pose is A3's wind-up.
- **A3:** from high right, a diagonal cut down to low left (about 8 o'clock, near the ground) with a big step in, ending in a planted finish.

Chained attacks enter partway into their wind-up (§5.2). The result reads as one continuous figure-eight instead of three separate clips, and it needs no extra frames.

### 6.3 Smooth motion over held poses

The character's world position moves every tick along authored curves (lunges, dodges) while sprite frames hold. That's how Hyper Light Drifter feels fluid with very few drawings.

### 6.4 Timing fitted to the art

Each clip carries markers: wind-up end, contact, and follow-through end. The animator scales each segment's frame durations to fit the attack's frame data. That gives two guarantees:

- The contact drawing always lands on the first active frame.
- Timing can be retuned without new art.

### 6.5 Directions

There are 8 facings: S, SE, E, NE and N are painted; SW, W and NW are mirrored (**OPEN:** 8 vs. 4 directions). Mirroring means the sword visually swaps hands when facing left, which most 2D games accept.

### 6.6 Transitions

- **Idle or run → attack:** goes straight to the wind-up pose. A strong, distinct pose reads as intentional rather than as a pop.
- **Attack → run:** the walk-out window skips the rest of the recovery and goes into the run start.
- **Run → stop:** dedicated stop and skid frames.
- **Later polish:** choose attack entry variants based on which foot is forward.

### 6.7 Quality checks for every animation

- **Contact sheet:** all frames side by side.
- **Onion-skin overlay:** checks spacing and arcs.
- **Solid-silhouette sheet:** checks readability.
- **GIF at game timing.**
- **In-engine capture** with the hitbox overlay on.

## 7. Code vs. authored animation

- **Fine to do in code:**
  - hit flash, hit-stop, shake
  - knockback physics, launch height
  - slash arcs rotated to the exact aim angle
  - dodge afterimages, particles, camera
  - possibly a physics-driven cape or scarf
- **Never in code:** rotating, tweening or stretching a still sprite to fake a swing or a pose. Body poses must be authored frames.
- **Placeholder mannequin:**
  - Phases 1 and 2 use a simple mannequin drawn from basic shapes, with real key poses and timing.
  - It exists only to tune timing and is labeled as a placeholder.
  - It is never presented as finished animation.

## 8. Assets

### 8.1 Needed for the prototype

| Asset | Frames per painted direction |
|---|---|
| Player locomotion: idle, run, stop/skid, dodge | ~6, 8, 3, 7 |
| Player attacks: A1, A2, A3, dash-slash | ~7, 7, 10, 6 |
| Player hurt, death | ~3, 8 |
| Light enemy: idle, move, telegraph and attack, flinch, knockback, knockdown/get-up, death | ~33 |
| Heavy enemy: the same set, plus the poise-break stagger | ~35 |

- **Player:** about 300 frames at 8 directions (5 painted, with death in only 2 directions). At 4 directions (3 painted), about 185.
- **Enemies:** about 100 frames each at 4 directions (S, E and N painted, W mirrored).
- **Effects** are painted once, then rotated and scaled in the engine:
  - slash arcs for A1, A2, A3 and the dash-slash
  - light and heavy impact bursts
  - armor sparks
  - dust: footsteps, dodge, skid, landing
  - the finisher's ground crack
  - a death burst
- **Attack frame breakdown:** about 2 anticipation, 1 smear, 1 contact, 2 follow-through and 1 recovery. A3 has a longer coil and recovery.

### 8.2 Audio

- **Placeholders:** Claude can synthesize placeholder sounds to tune timing, but they won't be premium.
- **Final sound:** should come from a real library (the free Sonniss GDC bundles are a good start) or a sound designer.
- **Layers:**
  - a whoosh, timed to the smear frame rather than the button press
  - the impact (light and heavy)
  - a material layer (flesh, armor)
  - extra low end for the finisher
  - a kill accent
  - footsteps and the dodge
- **Variation:** each sound gets 3–5 variations, with pitch and volume variation and round-robin playback.

### 8.3 Risks with AI-generated frames

1. **Same design on every frame.** The user reports that GPT-6 SOL is strong here; the pilot will confirm it.
2. **Registration.** Image models don't keep the feet in the same spot at the same scale on every frame. The ingest tool (§8.6) handles this, so it's solvable in code.
3. **Motion quality.** This is the real risk. Image models paint poses; they don't understand timing, spacing or arcs. Every key pose is specified exactly (§8.5) and all timing lives in data. But only seeing it in motion tells you whether a swing reads.
4. **Brush texture flickering between frames.** This is reduced by painting the character cleaner than the environments, locking the palette, and painting at 2× then downscaling.

### 8.4 The pilot (checkpoint before mass production)

The pilot runs one attack through the whole pipeline:

1. Approve a model sheet showing the character in 5 directions.
2. Generate A1 in one direction. SE, the hardest common view.
3. Run it through the ingest tool.
4. Put it in the engine at real timing.
5. Judge it in motion.

- **If it passes:** scale up production.
- **If the motion fails:** fall back to a 3D pose-guide pipeline.
  - Claude keyframes a simple 3D mannequin in Blender for exact arcs and foreshortening in every direction.
  - Claude renders pose guides from it.
  - GPT-6 SOL paints over the guides.
  - It's the same principle Dead Cells and Supergiant's games use: get the motion from 3D, keep the look 2D.
- **Alternative:** a human 2D animator for just the hero's attacks, with AI doing the painting.

### 8.5 Prompt rules (draft; finalize with the art direction)

Every GPT-6 SOL prompt carries these rules:

- **View:** the same 3/4 top-down view on every asset, orthographic, with no perspective distortion.
- **Facing names:** S faces the viewer (down-screen), N faces away, and E faces screen-right. Diagonals sit between them.
- **Scale and placement:** the same canvas size and character height on every frame, with the feet on a fixed ground line. Attach a template image with guide lines.
- **Background:** a flat chroma color or transparency.
  - no ground
  - no cast shadow; the engine draws it
  - no effects; those are separate assets
  - no text, labels, numbers or frame borders
- **Lighting:** one key light from the upper left, matching the world, plus soft ambient light.
- **Style:**
  - Hand-painted, with strong silhouettes.
  - Characters are painted cleaner and flatter than environments.
  - A fixed palette, as a hex list taken from the model sheet.
- **Weapon:** the same sword design and length on every frame, held right-handed.
- **Sheet layout:** N frames in one row, in equal cells, ordered left to right.
- **Per-frame pose spec:**
  - facing and animation phase
  - weight and foot placement
  - hip and shoulder rotation
  - sword-hand position
  - sword angle as a ground-plane clock position (§6.2), plus which way the edge faces
  - head and gaze
  - direction of cloth and hair motion
- **References:** always attach the approved model sheet, plus the previous frames for continuity.

### 8.6 Ingest tool (planned: Python, in `tools/`)

The tool will:

- slice generated sheets into frames
- validate or remove the background
- align frames on the feet anchor and normalize scale
- lock colors to the character palette
- trim frames and pack them into atlases
- write frame metadata for Godot
- produce the review sheets from §6.7

## 9. Architecture

### 9.1 Modules

| Brief §8 system | Module (proposed class) | Responsibility |
|---|---|---|
| Player combat controller | `PlayerMotor` | Locomotion, facing, dodge, applying lunges, collision. |
| | `CombatController` | State machine (idle, move, attack, dodge, hurt). Reads the input buffer, selects attacks from combo data, enforces windows. |
| | `InputBuffer` | Records each press with its tick and direction. Configurable windows. |
| Attack definitions and combo data | `AttackData`, `ComboData` | Resources (§9.2). |
| Animation controller | `DirectionalAnimator` | Picks the clip by facing and handles mirroring. Fits art timing to frame data (§6.4), fires frame events, and exposes hand and blade-tip sockets. |
| Hitbox and hurtbox management | `HitboxSystem`, `Hurtbox` | Swept arcs and capsules, per-swing hit records, teams. |
| Enemy reaction controller | `ReactionController` | Poise, armor and mass. Resolves reactions (§5.7) and applies knockback and launch. |
| Impact feedback manager | `ImpactDirector` | One `on_hit` entry point. Drives hit-stop, camera, effects, rumble, audio and slow-motion from an `ImpactProfile`. |
| | `HitstopClock` | Per-entity time, with local and global scopes. |
| | `CameraRig` | Follow with look-ahead, directional kick, trauma-based shake. |
| Combat audio manager | `CombatAudio` | Event-based layered sound with variation and round-robin. |
| Test arena | `CombatArena`, `DebugTools` | See §10.3. |

### 9.2 Data (Resources)

- **`AttackData`:**
  - animation clip
  - timing: wind-up, active and recovery frames, plus the chain entry frame
  - windows: the frames from which the next attack, a dodge-cancel or a move-cancel can start; buffer length
  - lunge: distance, curve and frames; how much momentum carries over
  - aim-assist angle and range
  - hit shape: arc angles, radii and sweep curve, or a capsule for thrusts and overheads
  - damage, poise damage, reaction power, knockback force, tangent blend, launch height
  - its `ImpactProfile`, slash effect and swing sound
- **`ComboData`:** the ordered attacks and the reset grace. Branches (the dash-slash entry, later variations) are added as data, not code.
- **`ImpactProfile`:**
  - hit-stop frames and scope
  - victim shake
  - camera kick and trauma
  - rumble
  - flash frames
  - slow-motion
  - effects and sounds
- **`EnemyProfile`:**
  - health, mass and armor
  - maximum poise, plus regeneration delay and rate
  - reaction clips by level and side
  - death variants
  - material, which selects effects and sounds
- **`WeaponData`** (later): a moveset reference and damage scaling, so a new weapon is data plus art.

### 9.3 Proposed folder layout

```
res://
  core/            fixed-tick helpers, HitstopClock, utilities
  combat/
    data/          Resource scripts + .tres instances
    player/        PlayerMotor, CombatController, InputBuffer
    animation/     DirectionalAnimator, clip resources
    hit/           HitboxSystem, Hurtbox
    reactions/     ReactionController
    feedback/      ImpactDirector, CameraRig
    audio/         CombatAudio
  enemies/         shared enemy base + skirmisher, brute, dummy
  arena/           CombatArena scene, DebugTools, tuning panel
  art/             processed sprites/atlases (output of tools/)
  sfx/             sound files
  tests/           automated tests + recorded input sequences
tools/             Python asset pipeline (contains .gdignore so Godot skips it)
docs/              design docs
```

## 10. Testing and tuning

### 10.1 What Claude can verify

- **Scripted play:** play the game through recorded input sequences and capture every frame with Movie Maker mode, which is deterministic and goes frame by frame.
- **Visual review:** review attacks as contact sheets and GIFs.
- **Synchronization:** check that the contact frame, the start of hit-stop and the start of the enemy's reaction land on the same tick, and that hit-stop lasts exactly as specified.
- **Latency:** measure the time from input to the first visual change.
- **Automated tests:**
  - buffering and cancel windows
  - combo chaining and reset
  - hit registration at full swing speed, with no skipping past enemies
  - determinism

  The test framework (GUT or gdUnit4) gets chosen in Phase 1.

### 10.2 What needs the user

Feel. Claude can't hold the controller. The user plays in the arena with the tuning panel and reports back using the checklist in §10.4.

### 10.3 Combat arena

- **Spawning:** spawn the light enemy, the heavy enemy or the training dummy. Toggle enemy AI on and off, and toggle player invulnerability.
- **Overlays:**
  - hitboxes and hurtboxes
  - a frame-data HUD showing the current action, the frame number and the phase in color
  - an input log that shows buffered presses
- **Time controls:** slow motion (0.5× and 0.25×) and frame-by-frame stepping.
- **Tuning panel:** live sliders for timing, hit-stop, knockback, shake and movement values, saved back to the Resources.
- **A/B toggles:** switch off hit-stop, shake, flash or sound one at a time, to feel what each one contributes.
- **Input recording:** record and replay input sequences. The tests use these too.

### 10.4 Feel checklist (for the user's play sessions)

Rate each item from 1 to 5 and add one sentence.

1. Does starting and stopping feel instant, without being floaty or icy?
2. **Dodge:**
   - Are the distance, speed and recovery right?
   - Does the dash-slash come out when you expect it?
3. Does each attack come out when you press? Does anything feel late?
4. Do A1, A2 and A3 feel clearly different from each other?
5. Does the finisher feel heavy enough? Or too slow?
6. Mashing vs. rhythmic pressing: any dropped inputs, or unwanted extra attacks?
7. Is hit-stop too sticky or too light? Rate light hits and the finisher separately.
8. Do enemy reactions look caused by the blade? Is the knockback distance right?
9. Is there too much camera shake, or too little?
10. Do the whoosh and the impact land at the right moment?
11. After a fight, do you want to fight another enemy right away?

## 11. Development plan and checkpoints

0. **Decisions and docs** (in progress).
   - Done: this doc, `CLAUDE.md` and the brief are committed.
   - Remaining: the user answers §12, then Claude finalizes the prompt rules and writes the model-sheet prompt.
1. **Movement and combat core,** with the placeholder mannequin:
   - project setup and the fixed-tick foundation
   - `PlayerMotor` and the dodge
   - `InputBuffer`, `CombatController`, `AttackData` and `ComboData`
   - `DirectionalAnimator` with placeholder clips
   - swept hitboxes and hurtboxes
   - debug tools, the first version of the arena, and automated tests

   **Checkpoint:** the user confirms that movement and attack timing feel right.
2. **Impact and reactions:**
   - `HitstopClock`, `ImpactDirector` and `CameraRig`
   - flash, particles, knockback and launch
   - poise
   - both enemies and the dummy, with telegraphed AI
   - placeholder audio

   **Checkpoint:** hitting the dummy is satisfying even with placeholder art.

   The art pilot (§8.4) runs alongside this phase, and its result decides the animation pipeline.
3. **Real art and audio:**
   - the full player set and both enemies
   - painted effects
   - real sound
   - every attack's art timing fitted to its frame data
4. **Tuning** against brief §10 and the feel checklist.

Content expansion (more weapons, enemies, abilities, parry, and so on) starts only once an ordinary fight is enjoyable on its own.

## 12. Open decisions

1. **Godot:** which version is installed, and where is the executable? 4.3 is the minimum and 4.7 is ideal. Record the path in `CLAUDE.md`.
2. **Directions:** 8 directions (5 painted, 3 mirrored) or 4 (3 painted)? The recommendation is 8.
3. **Animation pipeline:**
   - Start with the GPT-6 SOL pilot, with the 3D pose-guide fallback?
   - Is the user open to a Blender step, or to a human animator for the hero's attacks?
4. **Controls:** gamepad-first, or mouse aiming?
5. **Art direction:** the hero, the sword and the painted-style target. These are needed before the model-sheet prompt can be written.
6. **Higgsfield connector:** use it for the pilot (images, image-to-3D, audio)? It spends the user's credits, so only use it if the user says so.

## 13. Decision log

| Date | Decision |
|---|---|
| 2026-11-05 | Engine: Godot, which the user has installed. Typed GDScript, a fixed 60 Hz simulation, and data-driven Resources. |
| 2026-11-05 | Workflow: Claude Code runs on the user's computer in the project folder, alongside the Godot editor. The plan lives in the repo (`CLAUDE.md` plus `docs/`) for handoff between sessions. |
