# CLAUDE.md

This is a hand-painted 2D top-down fantasy action-adventure RPG. **Sword combat is the signature feature, and movement feel is the top priority.** The bar is premium, AAA-level game feel. The references are Hyper Light Drifter, CrossCode, Death's Door and Tunic, used for quality and principles, not to copy.

## Read these before starting work

- `docs/combat-director-brief.md`: the user's original brief. It is the source of truth for requirements.
- `docs/combat-design.md`: the technical design and plan. It covers frame data, animation strategy, assets, architecture, testing, phases and open decisions.

## Status

The project is in Phase 0 (planning). There is no Godot project yet; the repo holds only docs.

Next step: the user answers the open decisions in `docs/combat-design.md` §12. After that, Phase 1 (movement and combat core) starts.

## Decisions so far

- **Engine:** Godot 4 with typed GDScript. Target 4.7.x; the minimum is 4.3 (2D physics interpolation). The user's installed version isn't known yet, so ask.
- **Timing:** combat logic runs at a fixed 60 Hz. Frame data is counted in frames at 60 fps, where frame 0 is the tick an action starts.
- **Data, not code:** attack timing, damage, knockback, hit-stop and combo windows all live in Resources. Don't write special-case code for individual attacks.
- **Art:** the user generates assets with GPT-6 SOL from prompts Claude writes. Every prompt must carry the full rule set in `docs/combat-design.md` §8.5.
- **Workflow:** Claude Code runs on the user's computer in this folder, and the Godot editor stays open on the same folder. The user plays with F5.

## Ground rules

- Don't add content (weapons, enemies, abilities, levels) until the basic sword combat is genuinely satisfying.
- Never fake character animation in code: no rotating, tweening or stretching a still sprite to fake a swing. Code is for effects only (§7 of the design doc).
- Be honest about asset limits, and list exactly which frames and assets are needed.
- Weight comes from the contact frame and the enemy's reaction, not from slow player startup.
- Verify through gameplay (scripted input plus frame capture), not just by reading code.
- Claude can't feel the game, so the user tunes feel. Ask them specific questions (§10.4).
- Don't use the Higgsfield connector unless the user asks; it spends their credits.

## Running Godot from the command line

The path to the Godot executable on the user's machine hasn't been recorded yet. Ask the user, then write it here.

- **Windows:** use the `*_console.exe` from the Godot download, so output shows in the terminal.
- **macOS:** `/Applications/Godot.app/Contents/MacOS/Godot`

Run these from the repo root, replacing `GODOT` with the path. Scene paths are examples until the project exists.

- **Run the game:** `GODOT --path .`
- **Run one scene:** `GODOT --path . res://arena/combat_arena.tscn`
- **Capture frames deterministically** (Movie Maker mode, numbered PNGs): `GODOT --path . --write-movie out/capture.png --fixed-fps 60 --quit-after 600 res://arena/combat_arena.tscn`

Notes:

- The editor imports new art files. After new art is added, focus the Godot editor so it imports them before running from the command line.
- Keep `out/` (captures) out of git.
