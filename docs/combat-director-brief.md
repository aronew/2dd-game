# Combat Director Brief (original)

This is the user's vision statement and combat brief as given on 2026-11-05. The wording is unchanged; only markdown formatting was added. **This is the source of truth for requirements.** The technical response to it is in `docs/combat-design.md`.

## Vision

I want to make a hand painted 2-d top down fantasy action-adventure RPG that excels in sword fighting and combat the game should feel top tier AAA, movement should be fast and deliberate. all movement should be top priority. I use GPT-6 SOL for creating assets that are incredibly consisstent as long as you write the prompts with the rules and what's needed.

## COMBAT DIRECTOR BRIEF — PREMIUM 2D SWORD COMBAT

I want you to approach the melee combat system as if you are developing a premium action-adventure game with an exceptionally high standard for combat animation and game feel.

My goal is to create sword combat that is so satisfying that players actively seek out enemies just to experience the fighting.

The combat should be fast, fluid, responsive, weighty, visually expressive, and extremely satisfying.

Primary combat references:

- Hyper Light Drifter — fast, fluid, impactful combat.
- CrossCode — expressive sword combinations and attack variety.
- Death’s Door — weighty attacks, satisfying enemy reactions and responsive movement.
- Tunic — readable directional sword combat.

These are references for quality and design principles, not games to copy.

### 1. Combat philosophy

The combat must achieve two things simultaneously:

1. The player feels incredibly responsive and in control.
2. Every sword strike feels like it has physical force and consequence.

Avoid combat that looks like a character simply cycling through a handful of attack sprites.

I want a cohesive animation system in which the character’s body, weapon, movement, attack direction, and enemy reactions all work together.

### 2. Sword attack animation system

Build a proper directional melee attack system.

Every attack should have distinct animation phases:

- Anticipation: a brief preparation that establishes the direction and energy of the attack.
- Acceleration: the weapon moves rapidly through its attack arc.
- Contact: the strongest visual moment of the attack.
- Follow-through: the body and weapon continue naturally through the movement.
- Recovery: the character returns to a controllable state.

The attack should accelerate through the middle of the swing rather than moving at a constant speed.

I want:

- Directional attacks that respond to player facing and input.
- Distinct horizontal, vertical, diagonal and finishing attacks where appropriate.
- Different body poses and weapon trajectories for each attack.
- Smooth transitions between attacks.
- No visibly repetitive or robotic attack cycles.
- A satisfying final attack in each combo sequence.

### 3. Combo system

Implement a deliberate multi-hit combo system.

The player should be able to press the attack button repeatedly and chain attacks together.

Each attack in the sequence should feel different.

For example:

Attack 1: Fast horizontal slash.

Attack 2: Reverse slash with a different body rotation and sword trajectory.

Attack 3: Strong diagonal finishing strike with increased impact, knockback and hit-stop.

The combo system must support:

- Input buffering.
- Carefully timed combo windows.
- Directional variation.
- Smooth animation transitions.
- Movement during appropriate attack windows.
- A satisfying finishing attack.

Do not simply play the same animation multiple times with different damage values.

### 4. Enemy hit reactions

This is one of the highest priorities.

Enemies must visibly react to being struck.

Implement different reactions depending on attack strength, attack direction, enemy type and remaining health.

Possible reactions:

- Light flinch.
- Directional recoil.
- Heavy stagger.
- Knockback.
- Knockdown.
- Launch or aerial reaction where appropriate.
- Death animations with individual character and enemy responses.

The enemy should not simply lose health while continuing to stand perfectly still.

The attacker’s weapon, the enemy’s body and the impact effects must feel synchronized.

Different enemy sizes and weights should affect how they react.

### 5. Impact and game feel

Create a dedicated impact feedback system.

Explore and implement:

- Hit-stop on successful contact.
- Directional knockback.
- Weapon trails.
- Impact sparks and particles.
- Small, controlled camera shake.
- Stronger effects for finishing attacks.
- Distinct impact sounds.
- Enemy damage flashes.
- Appropriate sound and visual variation between light and heavy attacks.

Hit-stop should be brief and carefully tuned. It must enhance impact without making rapid combos feel sluggish.

Effects should communicate the strength of the attack rather than just fill the screen with particles.

### 6. Animation quality

Animation quality is a defining feature of this game.

I want you to evaluate:

- Frame timing.
- Silhouette readability.
- Body rotation.
- Weapon arcs.
- Anticipation and follow-through.
- Directional consistency.
- Animation blending.
- Idle-to-attack transitions.
- Attack-to-movement transitions.
- Enemy reaction timing.

If the existing character sprites or animations are insufficient to achieve this quality, identify that limitation.

Do not pretend that code-generated sprite movement is equivalent to genuinely well-authored animation frames.

If additional sprite frames, spritesheets, weapon effects or animation assets are needed, identify exactly what needs to be created.

Use procedural effects where they genuinely improve the presentation, but do not use them as a substitute for proper character animation.

### 7. Combat responsiveness

The player must never feel like they’re fighting the controls.

Implement:

- Responsive attack initiation.
- Input buffering.
- Appropriate animation cancel windows.
- Directional attack selection.
- Smooth transitions between locomotion and combat.
- Dodge integration.
- Appropriate recovery windows.
- Reliable hit detection.

Do not allow excessive animation lock or unnecessary delays between attacks.

The objective is fluid, skillful combat with meaningful commitment, not sluggish combat disguised as weight.

### 8. Technical architecture

Build this as a cohesive, reusable combat system.

Separate the relevant systems where appropriate:

- Player combat controller.
- Attack definitions and combo data.
- Animation controller.
- Hitbox and hurtbox management.
- Enemy reaction controller.
- Impact feedback manager.
- Combat audio manager.

Make attack timing, damage, knockback, hit-stop, combo windows and other important parameters configurable.

Avoid hardcoding individual attacks into an unmaintainable collection of unrelated conditions.

Make the system expandable for future weapons, enemies, abilities and combat upgrades.

### 9. Development and quality assurance

Do not attempt to build the entire combat experience in one rushed implementation.

First inspect the existing project, character assets, animation system, movement controller and combat code.

Identify what already exists, what needs improvement and what needs to be built.

Then develop a polished combat prototype containing:

- One fully animated player character.
- One basic sword with a three-hit combo.
- At least two enemy types with different physical reactions.
- Directional attacks.
- Hit-stop and impact effects.
- Combat audio.
- Dodge and movement integration.

Create a dedicated combat test arena where I can repeatedly fight enemies and evaluate the feel of every attack.

Test the combat through actual gameplay, not just code inspection.

Tune the system based on how it feels when played.

Do not move on to adding large amounts of content until the basic sword combat is genuinely satisfying.

### 10. Definition of success

I want to reach a point where fighting an ordinary enemy is enjoyable in itself.

A player should be able to:

- Move naturally into an attack.
- Chain multiple distinct sword strikes.
- Feel the weight of every successful hit.
- See enemies react convincingly.
- Experience satisfying finishing attacks.
- Immediately want to engage another enemy.

Do not settle for functional combat. Aim for exceptional combat feel.

Treat combat as one of the game’s signature features, not just another gameplay mechanic.

Before implementation, present your technical assessment and proposed development plan. Identify any asset limitations honestly, and explain how you intend to achieve the animation quality described above.
