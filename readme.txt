===============================================================================
                                  A E R W Y N
                    A browser-based open-world fantasy RPG
                          Single self-contained HTML file
===============================================================================

CONTENTS
  1. Quick start
  2. Controls
  3. Externally loaded assets
  4. Technical architecture
  5. Performance system
  6. Save system  (IMPORTANT - read this)
  7. Complete game guide (beginning to end)
  8. Reference tables
  9. Known limitations


===============================================================================
1. QUICK START
===============================================================================

Open aerwyn.html in a modern browser. Chrome on Android and desktop Chrome,
Edge, Firefox and Safari are all supported.

On the title screen, read the small line of text beneath the "Enter the World"
button. It tells you whether your browser will save your progress. If it warns
you that saving is unavailable, see section 6 before playing.

Press "Enter the World". The game requests fullscreen and, on mobile, tries to
lock to landscape orientation. First load takes a few seconds while textures
are generated and the forests are grown.

Everything is contained in one HTML file of roughly 210 KB. There are no image
files, no audio files, and no model files shipped alongside it.


===============================================================================
2. CONTROLS
===============================================================================

MOBILE
  Left half of screen ...... drag to walk (a joystick appears where you touch)
  Push the stick far ....... run
  Right half of screen ..... drag to look around
  Crossed swords button .... Strike
  Star button .............. Cast the selected spell
  Arrow button ............. Leap
  Spell bar (bottom) ....... tap to select Flame / Frost / Mend
  Prompt text .............. tap it to interact (talk, open, enter)
  Top-left buttons ......... satchel (inventory) and map

DESKTOP
  W A S D / arrows ......... move
  Shift .................... run
  Mouse drag ............... look around
  Click or E ............... Strike
  Q ........................ Cast the selected spell
  Space .................... Leap
  1 / 2 / 3 ................ select Flame / Frost / Mend
  F ........................ interact
  I ........................ satchel
  M ........................ map
  Escape ................... close panels

A quick mouse click strikes; a click-and-drag rotates the camera. The game
distinguishes them by drag distance and hold duration.


===============================================================================
3. EXTERNALLY LOADED ASSETS
===============================================================================

This is the complete list. Everything not listed here is generated at runtime
by code inside the HTML file.

--- JAVASCRIPT LIBRARIES ---

  Three.js r128 (core renderer)
    https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js
    License: MIT

  GLTFLoader (loads .glb model files)
    https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js
    License: MIT

  SkeletonUtils (deep-clones rigged models so each copy has its own skeleton)
    https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/utils/SkeletonUtils.js
    License: MIT

  BufferGeometryUtils (merges geometry; used by the tree generator and to
  reduce per-character mesh counts)
    https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/utils/BufferGeometryUtils.js
    License: MIT

--- 3D MODELS ---

  Fox.glb  - the ONLY external 3D model in the game
    https://cdn.jsdelivr.net/gh/KhronosGroup/glTF-Sample-Models@master/2.0/Fox/glTF-Binary/Fox.glb
    Source:  Khronos Group glTF Sample Models repository
    License: CC0 / public domain (see the Khronos repository for full details)
    Rigged quadruped with three animation clips: Survey, Walk, Run.
    Used for:
      - Gray Wolf   (scaled to 1.15 m, tinted grey  #9a9186)
      - Dire Wolf   (scaled to 1.72 m, tinted dark  #4a453f)

  No other models are downloaded. Every humanoid - the player, villagers,
  bandits, skeletons, the troll and the Barrow King - is generated in code by
  the "character forge" described in section 4.

  Design note: an earlier version used Three.js's Soldier.glb for humanoids. It
  is a modern military soldier and looked wrong in a medieval fantasy setting,
  so it was removed. No CC0 rigged fantasy characters were available on a CDN
  that could be linked directly and verified, so humanoids are procedural
  instead. This was a deliberate trade of fidelity for thematic coherence.

--- FONTS ---

  Cinzel (headings, UI labels) and Cormorant Garamond (body text)
    https://fonts.googleapis.com/css2?family=Cinzel:wght@500;700&family=Cormorant+Garamond:ital,wght@0,400;0,500;1,400&display=swap
    License: SIL Open Font License

--- OFFLINE BEHAVIOUR ---

  If any of the above fails to load, the game still runs. Model loading has a
  20-second timeout and falls back to procedurally generated wolves. Fonts fall
  back to Georgia and generic serif. Only Three.js itself is mandatory.

--- WHAT IS *NOT* DOWNLOADED ---

  All textures ......... generated pixel-by-pixel or drawn on canvas at load
  All sound and music .. synthesised with the Web Audio API
  All terrain .......... generated from noise functions
  All humanoids ........ generated by the character forge
  All buildings ........ generated from primitives
  All particle effects . generated at runtime


===============================================================================
4. TECHNICAL ARCHITECTURE
===============================================================================

--- WORLD GENERATION ---

  The world is a 2400 x 2400 unit island built from a deterministic value-noise
  function (a hash-based fBm with 5 octaves, plus a ridged variant for
  mountains). The same seed always produces the same world.

  Terrain height combines:
    - multi-octave fBm for rolling hills
    - a ridged-noise mountain band to the north
    - two subtracted gaussian basins forming lakes
    - a radial falloff mask producing coastline and open ocean

  The terrain mesh is a 240 x 240 segment plane (about 10 m per quad) with
  per-vertex colours blending sand, grass, dirt, rock and snow by height, slope
  and a noise tint.

  IMPORTANT SUBTLETY: because the mesh only has vertices every 10 m, the drawn
  surface between vertices is the flat interpolation of its corners, not the
  analytic noise curve. Characters are placed using surfaceH(), which samples
  that same interpolated surface with memoised corner heights. Using the
  analytic curve instead caused feet to sink up to 4.9 m into hillsides.

--- TREES ---

  Trees are grown recursively, not assembled from cones. A trunk splits 2-3
  ways per level through four generations; each branch tapers, bends and curls
  on its own deterministic seed. Leaf clusters are alpha-cut cards showing 150
  hand-drawn overlapping leaflets with veins and gradients, mounted as crossed
  quads at branch tips and mid-level forks.

  Five variants exist (three broadleaf, two conifer with drooping limbs). Each
  variant's branches are merged into a single geometry and drawn with
  InstancedMesh, so an entire species is one draw call.

--- CHARACTER FORGE ---

  Humanoids are built from anatomical parts rather than boxes:
    - torso is a lathed profile that curves like a ribcage
    - limbs are tapered bones with rounded joint caps, merged to one mesh each
    - layered gear: leather cuirass, belt, hanging tassets, fur mantle, cloak,
      domed pauldrons, vambraces, boots with fur cuffs
    - heads have a cranium, jaw, hair cap, optional beard and long hair
    - skeletal variants get a ribcage, spine and glowing eye sockets
    - optional crown and horns

  Parameters control height, build (lean / normal / broad / hulk), skin, hair,
  cloth, leather and metal colours, and which gear pieces are present. A
  reduced-detail mode drops trim pieces and lowers segment counts for mobile.

  Animation is procedural: hips, knees, torso, head, shoulders and elbows are
  driven by sine-based gait cycles blended with attack and cast poses.

--- TEXTURES ---

  All textures are generated at load. Two techniques are used:
    - per-pixel painting for noise-based maps (terrain detail, granite, bark,
      needles, cloth weave, worn leather, water normals)
    - canvas vector drawing for detailed art (leaf clusters, grass blades,
      birch lenticels, flower petals)

  All textures use trilinear mipmapping and maximum anisotropic filtering.

--- LIGHTING AND SKY ---

  A custom GLSL sky dome shader renders a gradient with a sun disc and glow.
  A full day/night cycle runs every 10 minutes, interpolating three palettes
  (day, dawn/dusk, night) across sky, fog, sun and ambient light. Stars fade in
  at night, clouds drift, and house windows only glow after dark.

--- AUDIO ---

  Everything is synthesised with the Web Audio API:
    - filtered noise with a slow LFO for wind
    - a four-chord triangle-wave pad progression on an 8-second cycle
    - sparse bell tones from an A-minor pentatonic scale
    - noise bursts shaped by biquad filters for sword swings and impacts
    - a descending sawtooth for casting
    - an ascending arpeggio for level-ups
    - a rain hiss layer driven by weather intensity
    - a night shimmer layer that rises after dark

  A single 2-second noise buffer is generated once and replayed from random
  offsets, so repeated hits do not sound identical and no allocation occurs
  during combat.

--- COMBAT ---

  Melee uses a forward-arc hit test (dot product > 0.35, range 2.6 units)
  triggered on the exact contact frame of the swing animation. Landing a hit
  applies 55 ms of hit-stop plus camera shake. Spell projectiles travel with
  physics and test against enemies each frame.

  Effect lights come from a pool of four permanently in the scene. Adding or
  removing a light at runtime forces Three.js to recompile every material,
  which caused a visible hitch on every attack; pooling eliminated it.


===============================================================================
5. PERFORMANCE SYSTEM
===============================================================================

The game detects mobile devices and selects a quality tier at startup,
controlling render resolution, shadow map size and softness, tree and grass
density, rain particle count, water mesh resolution, cloud count, character
detail level, AI update range, and how often water, smoke, fire and weather
simulate.

On top of that, an adaptive guard samples the framerate every two seconds and
adjusts automatically. It degrades in this order:

    render resolution  ->  shadows  ->  draw distance  ->  leaf cards

and reverses each step when the framerate recovers above 55 fps. Mobile starts
one rung down the resolution ladder and climbs to the sharpest setting if the
device can sustain it.

Other optimisations:
  - enemies beyond 120 m (mobile) run no AI, animation or material work
  - hurt and frost tinting only touches materials when the state changes
  - water uses a scrolling normal map instead of recomputing vertex normals
  - static sub-parts within each animated joint are merged into single meshes
  - undead eye glow is emissive geometry, not point lights (this alone removed
    30 lights from the scene)


===============================================================================
6. SAVE SYSTEM  -  IMPORTANT
===============================================================================

The game autosaves every 15 seconds, when you background the tab, and when you
close it. It stores position, level, XP, gold, health, mana, inventory,
equipped gear, quest progress, kill count, world time and your explored map.

It tries four storage backends in order, probing each one for real by writing,
reading back and deleting a test value:

    1. Host-provided storage (when embedded in an app that supplies it)
    2. localStorage
    3. IndexedDB
    4. sessionStorage  (survives reloads, lost when the tab closes)
    5. Memory only     (lost immediately - you are warned)

THE FILE:// PROBLEM
  If you open the downloaded HTML file directly from your device, Chrome treats
  it as an "opaque origin" and may block localStorage, IndexedDB and
  sessionStorage entirely. This is a browser security policy, not a bug in the
  game. When it happens, no automatic saving is possible.

  The title screen tells you if this is the case. If it does:

    - Play normally.
    - Before you stop, open the satchel and press EXPORT. This downloads
      aerwyn-save.json containing your entire game state.
    - Next time, press "Continue from a save file" on the title screen, choose
      that file, then press "Enter the World". You resume exactly where you
      left off.

  To get seamless autosave instead, serve the file over http rather than
  opening it as a local file. Any simple local web server will do.

MANUAL CONTROLS
  The satchel contains SAVE NOW, EXPORT and IMPORT buttons, plus a plain
  description of which backend is currently in use.


===============================================================================
7. COMPLETE GAME GUIDE
===============================================================================

This walks the whole game from first step to ending. Spoilers throughout.

--- THE MAP ---

  Coordinates are (x, z). North is negative z, east is positive x.

    Spawn ................ ( 90,  250)  meadow south of the village
    Green Hollow village .. ( 90,  210)  Maren, Bram, Halda
    Fishing hamlet ........ (300,  205)  Oss, Nettle
    Wolf meadow ........... (-260,  90)  west
    Bandit camp ........... (330,  -30)  east, behind a stake wall
    Roadside bandits ...... (190-250, 90-140)
    Barrow entrance ....... (-180, -220) northwest hills
    Dire wolf ridge ....... (-120 to -30, -300 to -370)  north
    Highland troll ........ (-330, -140)
    Ruined watchtower ..... (-40,  420)  southern moor, haunted
    Wardstone I ........... (-150, 300)  blue,   south by the pines
    Wardstone II .......... (250, -230)  amber,  eastern rise
    Wardstone III ......... ( 20,  -60)  violet, valley heart

  Open the map (M or the diamond button) at any time. It reveals only where you
  have walked and shows landmarks, live enemy positions and loot.

--- STEP 1: THE VILLAGE ---

  You begin in a meadow with a campfire. Walk north to Green Hollow.

  Talk to ELDER MAREN. She asks you to deal with the wolves.
  Accept: this opens the whole main questline. She gives 15 gold and 2 salves.

  Also talk to:
    BRAM THE SMITH - offers "The Road Toll" (clear the bandit camp)
    HALDA THE TRADER - buys and sells; your shop
    Walk east to the hamlet for OSS and NETTLE, who offer more work and hints.

  You start with a Worn Shortsword, Traveler's Garb and 2 Minor Salves.

--- STEP 2: WOLVES AT THE GATE  (kill 5 wolves) ---

  Head west to about (-260, 90). Wolves are fast (5.4) but weak (42 hp).
  Fight them one at a time; they will chase you a long way (26 unit aggro).

  Reward: 60 gold, 90 XP. You should reach level 2-3.
  Drops: Wolf Pelts (sell for 9 each), occasional Minor Salves.

  Return to Maren.

--- STEP 3: THE THREE WARDSTONES  (wake 3 shrines) ---

  Maren explains the wolves were driven down by something worse. Three
  wardstones ring the valley. Visit each and touch it:

    (-150, 300)   south, by the pines
    ( 250, -230)  eastern rise
    (  20,  -60)  valley heart

  Each grants 60 XP. Completing all three gives 220 gold and 300 XP.

  This is also your best early exploration - it charts most of the map and
  takes you past several enemy groups.

  Return to Maren. She now reveals the barrow.

--- STEP 4: SIDE WORK (recommended before the barrow) ---

  You want to be around level 6-8 with better gear before the endgame.

  THE ROAD TOLL (Bram) - kill 7 bandits.
    Bandit camp at (330, -30), plus roadside bandits around (190-250, 90-140).
    Bandits have 60 hp and hit for 12. Drops include the Iron Longsword
    (+9 attack) and Padded Jerkin (+30 health).
    Reward: 180 gold, 240 XP.

  HEAD OF THE SNAKE (Bram) - kill the Brigand Chief.
    He is in the camp at roughly (332, -33). 110 hp, hits for 20.
    Guaranteed Stolen Signet; often drops Hunter's Saber (+15) or Ringmail.
    Reward: 260 gold, 320 XP.

  WHAT DROVE THEM DOWN (Sable) - kill 3 dire wolves.
    Northern ridge, around (-120 to -30, -300 to -370).
    95 hp, 17 damage, very fast (6.4) with a 30 unit aggro range. Dangerous in
    the open - fight them near rocks so they approach one at a time.
    Guaranteed Wolf Pelt, often a Dire Fang (sells for 60).
    Reward: 240 gold, 300 XP.

  THE RESTLESS BARROW (Sable) - destroy 8 skeletons.
    Skeletons are at the barrow mouth (-180, -220), inside the barrow, and
    around the ruined watchtower at (-40, 420). 50 hp, 11 damage, slow (3.2).
    They drop Grave Dust and Mana Draughts, rarely a Hunter's Saber.
    Reward: 120 gold, 180 XP.

  There is also a second troll in the highlands at (-330, -140) worth 120 XP
  and good practice for the Warden.

  SHOPPING at Halda:
    Iron Longsword ... 120g   (+9 attack)   - buy early
    Hunter's Saber ... 260g   (+15 attack)
    Padded Jerkin .... 140g   (+30 health)
    Ringmail ......... 320g   (+65 health)  - strongly recommended
    Emberbrand ....... 620g   (+26 attack, glows and casts light)
    Greater Salve .... 70g    (restores 110 hp) - carry at least three
    Mana Draught ..... 40g    (restores 50 mana)

    Sell your trophies: Wolf Pelts, Grave Dust, Stolen Signets, Troll Hide and
    Dire Fangs exist purely to be sold. Selling gives half the listed value.

--- STEP 5: THE WARDEN BELOW ---

  Maren sends you to the barrow at (-180, -220). Look for the stone mound with
  a dark mouth and two guardian stones lit by torches. Skeletons guard the
  entrance. Interact at the mouth to enter.

  Inside is a full cave: rough rock floor and ceiling, pillars, stalagmites,
  glowing blue crystal clusters and braziers. Six more skeletons roam here.
  There is a chest at the far end containing 220-380 gold, 120 XP and the
  BARROW EDGE (+34 attack, the best weapon in the game, glows cold blue).

  THE BARROW-WARDEN is a Cave Troll: 180 hp, 26 damage per hit, 2.8 unit reach.
  He is slow (2.6). Strike and retreat; do not stand in his reach. Frost is
  very effective - it slows him to 42% speed for 3.5 seconds, letting you
  circle him freely.

  He drops Troll Hide, usually a Greater Salve, and has a good chance of
  dropping Emberbrand or Warden Plate (+120 health).

  Reward for the quest: 300 gold, 420 XP.

--- STEP 6: THE CROWN BELOW  (the finale) ---

  At the far end of the barrow is a sealed stone door. It WILL NOT OPEN while
  the Warden lives - the game checks and tells you so. Kill him first.

  Interact with the door. It breaks with a shudder and the quest begins.

  Beyond is the inner sanctum: a domed chamber with a stone throne and a ring
  of six braziers.

  THE BARROW KING - 420 hp, 34 damage, 500 XP, 300-500 gold.

  Fight notes:
    - He has a health bar at the top of the screen.
    - He is immune to knockback.
    - At 66% and at 33% health he detonates a frost shockwave. It deals 22-34
      damage if you are within 9 units and shakes the screen. You are warned by
      a banner just before ("The barrow answers him" / "His crown burns cold").
      Back away when he approaches those thresholds.
    - He is faster than the troll (3.4) with a long 34 unit aggro range, so you
      cannot simply outrun him inside the chamber.

  Recommended loadout:
    Level 8+, Barrow Edge or Emberbrand, Warden Plate or Ringmail,
    3+ Greater Salves, a Mana Draught, and Mend selected as a backup heal.

  Tactics: open with Frost to slow him, close and strike two or three times,
  then disengage before his swing lands. Use Flame while retreating. Drink at
  half health rather than gambling on one more exchange.

  He always drops the Barrow Edge, Warden Plate, a Greater Salve and the Shard
  of the Crown (sells for 300).

  Quest reward: 900 gold, 1200 XP.

--- THE ENDING ---

  When he falls, the game shows a four-paragraph epilogue over a resolving
  musical cadence, along with your final statistics: level, gold carried, deeds
  completed, foes felled and percentage of the region charted.

  Press "Walk On" to return to the world. The realm stays open - you keep
  everything and can continue exploring, hunting and completing any quests you
  skipped. Your save records that you finished.

--- OPTIONAL: FULL COMPLETION ---

  All eight quests: Wolves at the Gate, The Restless Barrow, The Road Toll,
  Head of the Snake, What Drove Them Down, The Three Wardstones, The Warden
  Below, The Crown Below.

  Other goals: chart 100% of the map, collect every weapon and armour set,
  reach the highest mountain in the north, find the ocean shore at the island's
  edge, and see the world during a rainstorm at dusk.


===============================================================================
8. REFERENCE TABLES
===============================================================================

--- ENEMIES ---

  Name             HP    Damage  Speed  XP    Gold      Aggro
  Gray Wolf         42    8       5.4    22    2-8       26
  Dire Wolf         95   17       6.4    60   10-26      30
  Bandit            60   12       3.8    34    8-24      22
  Brigand Chief    110   20       4.2    75   30-60      24
  Risen Bones       50   11       3.2    30    4-14      20
  Cave Troll       180   26       2.6   120   30-70      24
  The Barrow King  420   34       3.4   500  300-500     34

--- WEAPONS ---

  Worn Shortsword    +0 attack     20g   starting weapon
  Iron Longsword     +9 attack    120g   shop, bandit drop
  Hunter's Saber    +15 attack    260g   shop, skeleton/brigand drop
  Emberbrand        +26 attack    620g   shop, troll drop; glows, casts light
  Barrow Edge       +34 attack    900g   barrow chest, Barrow King drop

--- ARMOUR ---

  Traveler's Garb     +0 health    25g   starting armour
  Padded Jerkin      +30 health   140g   shop, bandit drop
  Ringmail Hauberk   +65 health   320g   shop, brigand drop
  Warden Plate      +120 health   780g   troll drop, Barrow King drop

--- CONSUMABLES AND TROPHIES ---

  Minor Salve       restores 45 hp     25g
  Greater Salve     restores 110 hp    70g
  Mana Draught      restores 50 mana   40g
  Wolf Pelt         sell only          18g
  Grave Dust        sell only          22g
  Stolen Signet     sell only          45g
  Troll Hide        sell only          90g
  Dire Fang         sell only         120g
  Shard of the Crown sell only        600g

  Selling yields half the listed value.

--- SPELLS ---

  Flame   14 mana   30-42 damage    fast projectile
  Frost   18 mana   22-30 damage    slows the target to 42% speed for 3.5 s
  Mend    22 mana   heals 60-80     instant, no projectile

--- PROGRESSION ---

  Starting stats: 100 health, 60 mana, 18 attack
  Level 2 requires 100 XP; each level costs 1.5x the previous
  Each level grants +20 max health, +12 max mana, +4 attack, and a full refill
  Dying costs 15% of your gold and returns you to the campfire at spawn
  Health regenerates slowly; mana regenerates faster

--- WORLD ---

  World size ......... 2400 x 2400 units
  Terrain mesh ....... 240 x 240 segments (about 10 m per quad)
  Day length ......... 10 real minutes for a full cycle
  Weather ............ rain arrives and clears on its own cycle
  Map resolution ..... 96 x 96 exploration grid
  Enemies ............ 43 across 7 types
  NPCs ............... 5 across 2 settlements
  Quests ............. 8


===============================================================================
9. KNOWN LIMITATIONS
===============================================================================

  - The humanoid characters are procedurally generated and will not match the
    fidelity of hand-sculpted game assets. This was a deliberate choice to keep
    the art style coherent; see section 3.

  - Autosave depends on browser storage permissions. Opening the file directly
    from disk may disable it entirely. See section 6.

  - There is no collision with trees, rocks or buildings. You can walk through
    them. Only cave walls and terrain constrain movement.

  - Enemies do not path around obstacles; they move directly toward you.

  - First load takes a few seconds while textures are generated and forests are
    grown. This is one-time work, not a frame-rate cost.

  - The world is deterministic and identical on every playthrough.

  - Orientation lock is best-effort; some mobile browsers ignore it.


===============================================================================
Built with Three.js r128. One file, no build step, no bundler, no dependencies
beyond the CDN scripts listed in section 3.
===============================================================================
