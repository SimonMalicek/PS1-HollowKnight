#PS1-HollowKnight
####Video Demo: [Link to Video]

#### Description:

Game Controls:
WASD - Movements
Mouse - Look
Spacebar - Jump
Left Mouse Click - Attack

0. Executable Build
I've added the Windows executable as a zip file into the root folder.

1. Project Objective & Scope
This project is a functional remake of the 2013 game-jam prototype "Hungry Knight," which served as the mechanical proof-of-concept for the indie title "Hollow Knight."

The objective was to scope aggressively and build a complete, shippable game loop within the CS50 final project constraints. The core loop is a survival-timer mechanic: the player's health (represented as time) constantly decays. To survive, the player must dispatch enemies, which drop health orbs to replenish the timer. The primary goal is survival against a clock.

The project leverages my existing experience in Unreal Engine 5 to maximize development velocity, focusing on pragmatic implementation over theoretical "best practices" that would be overkill for a solo developer.

2. Core Technology: Unreal Engine 5 & Blueprints
Unreal Engine 5 was the strategic choice for this project. My professional and academic background with the engine allowed me to bypass the initial learning curve and focus directly on implementation.

For a solo developer, UE5's Blueprint Visual Scripting system is a massive force multiplier. While the engine's C++ API offers raw performance, the visual scripting of Blueprints allows for extremely rapid iteration on gameplay logic—player controls, AI, UI, and game state—without the costly overhead of C++ compilation cycles.

For a project of this limited scope, the development velocity gained from Blueprints far outweighs any negligible runtime performance cost. This is a critical trade-off for shipping a project quickly.

3. Technical Implementation: The PS1 Aesthetic
A key goal was achieving a "PS1-era" low-fidelity aesthetic. This was implemented efficiently not with complex post-process materials or custom shaders, but by directly manipulating UE5's rendering pipeline via console commands. This is a lightweight, non-destructive, and highly effective method for this style.

These commands are executed at runtime (e.g., from the PlayerController's BeginPlay event) to globally alter the render settings:

r.ScreenPercentage.MaxResolution 360

Effect: This is the primary lever. It forces the game's internal render resolution to a maximum of 360p, creating the characteristic low-resolution pixelation of the era.

r.Upscale.Quality 0

Effect: This disables all modern temporal upscaling and smoothing (like UE5's default TSR). The result is a sharp, nearest-neighbor 'crisp' pixel look when the 360p image is scaled to the monitor's native resolution.

r.PostProcessAAQuality 0

Effect: Disables standard anti-aliasing (FXAA/TAA), preserving the hard pixel edges.

r.MotionBlur.Quality 0

Effect: Disables modern motion blur, which was not a feature of that hardware generation.

r.DepthOfFieldQuality 0

Effect: Disables cinematic depth of field, ensuring the entire scene remains in sharp, uniform focus.

This approach is a prime example of an 80/20 solution: it achieves 90% of the desired look with 10% of the development effort that a full post-process material graph would require.

4. Architectural Breakdown
The project architecture is deliberately simple, clean, and built on standard Unreal Engine patterns.

4.1. Player & Input System
The player uses Unreal's standard and robust PlayerController / Character split. This separation of concerns is a core engine pattern.

BP_PlayerController: This class is responsible only for raw input handling via Unreal's Enhanced Input system. It maps hardware actions (WASD, Mouse, Spacebar, Left-Click) to abstract "Input Actions" (e.g., IA_Move, IA_Look, IA_Jump, IA_Attack). It knows nothing about what the player is, only how to process input.

BP_ThirdPersonCharacter: This is the player's physical representation (the Pawn). It consumes the abstract Input Actions from the Controller. It contains:

The CharacterMovementComponent (handles all physics, gravity, and movement logic out-of-the-box).

The Skeletal Mesh and Animation Blueprint.

The attack logic (e.g., "play attack montage," "run collision trace for damage").

The health/timer variable and the logic to handle its decay.

This decoupled architecture means input mappings can be changed in the PlayerController without ever touching the Character's game logic.

4.2. Enemy AI & Behavior
Enemy units (BP_EnemyCharacter) are also subclasses of the base Character class. This is an efficient choice, as it allows them to leverage the same built-in CharacterMovementComponent for pathfinding and navigation.

The AI is intentionally simplistic to meet the project scope, avoiding the setup and processing overhead of a full Behavior Tree (which I explored in my thesis but was unnecessary here).

The logic is a simple, state-less loop:

On Event Tick (or a slightly slower timer for performance), get the Player's current location.

Call the AI Move To function, targeting the player's location.

This "dumb" but effective "follow" behavior is extremely cheap and perfectly functional for the game's needs. Each enemy has a simple integer "Health" variable. The OnTakeAnyDamage event is bound to decrement this variable. When Health reaches zero, the enemy triggers its death logic (spawning an orb and destroying itself).

4.3. Game Loop: Health Orbs & Timer
The core survival mechanic is managed by the BP_GameMode and the player's state.

Health as Time: A "Health" float variable, representing the timer, is managed on the BP_ThirdPersonCharacter. A 1-second TimerHandle repeatedly calls a function to decrement this value. This function also checks for game-over (Health <= 0) and updates the UI progress bar.

Health Orbs (BP_HealthOrb): On death, the BP_EnemyCharacter spawns a BP_HealthOrb actor. This actor is a simple StaticMesh with physics simulation enabled and a SphereCollision component set to "OverlapOnlyPawn."

To create the dynamic "burst" effect, the orb's BeginPlay event calls AddImpulse on its static mesh, using a RandomUnitVector and a random force magnitude. This scatters the orbs unpredictably.

When the player's BP_ThirdPersonCharacter overlaps the orb's collision volume (OnComponentBeginOverlap), the orb calls a function on the player (e.g., HealPlayer) to add to the "Health" timer, and then immediately calls DestroyActor on itself.

4.4. UI (User Interface)
The UI uses UMG (Unreal Motion Graphics) for simplicity.

WBP_MainMenu: A simple widget spawned by a MainMenu_GameMode. It handles "Play" (which calls OpenLevel) and "Quit" (QuitGame).

WBP_PlayerHUD: Spawned by the gameplay BP_GameMode and added to the player's viewport. It contains the main "Time" progress bar. This bar's "Percent" value uses a Property Binding to a function on the BP_ThirdPersonCharacter that returns CurrentHealth / MaxHealth. This "pull" model ensures the UI is always in sync with the player's state without requiring complex event dispatching.

5. Asset & Content Pipeline
As a solo developer, building production-ready assets is a major bottleneck. The pipeline was focused on integration, not creation.

3D Models: All 3D models were sourced from Sketchfab. Assets were imported, and simple rough materials were created to fit the low-fi aesthetic.

Credit:
Hollow Knight - jacc_art_ https://sketchfab.com/3d-models/hollow-knight-cell-shading-30b57e9bcc5c47df9649826349dd95d4
Shroom Guy - wersaus33 https://sketchfab.com/3d-models/shrumal-ogre-hollow-knight-8cdb18ec9ebf4a01b47988b02791d8d8
Environment - Dasha Klyusova https://sketchfab.com/AnoFail/models

Audio: Sound effects and music were sourced from the "Hollow Knight" game.

Final Thoughts:
I had fun developing the atmosphere and bringing the "Hungry Knight" into the 3D world. In the future, I could add enemies chasing and attacking the knight and more levels with different environments.