# Adding real assets

## Mixamo footballer (FBX, no Blender needed)

1. On mixamo.com pick a character and download it: Format **FBX Binary**, **With Skin**.
   Save it as `public/assets/mixamo/player.fbx`.
2. For each animation, search, tick **In Place** when it's offered, then download
   it as FBX Binary, **Without Skin**:
   - `idle.fbx`: "Soccer Idle" or "Idle"
   - `run.fbx`: "Running" or "Soccer Run"
   - `kick.fbx`: "Soccer Pass" or "Strike Forward Jog"
   - `tackle.fbx`: "Soccer Tackle" or "Slide"
3. Use this manifest. `height` measures the model and scales it to that many
   metres, so you don't need to know its units (it overrides `scale`).

```json
{
  "player": {
    "url": "assets/mixamo/player.fbx",
    "height": 1.75,
    "facing": "+z",
    "tintMaterial": "Top",
    "clips": {
      "idle": "assets/mixamo/idle.fbx",
      "run": "assets/mixamo/run.fbx",
      "kick": "assets/mixamo/kick.fbx",
      "tackle": "assets/mixamo/tackle.fbx"
    }
  },
  "keeper": null,
  "ball": null,
  "arena": null
}
```

A clip value ending in `.fbx`/`.glb` is loaded from that file. Hip drift is removed
automatically (set `"inPlace": false` to keep it). Set `tintMaterial` to part of the
shirt material's name to recolour it per team; the browser console logs material
names if you are unsure which one to use.

## Soccer Game Pack and animation variants

Each clip entry accepts a single file/name or an array of files/names:

```json
"tackle": ["assets/mixamo/tackle.fbx", "assets/mixamo/pack/soccer_tackle_2.fbx"]
```

The renderer picks uniformly from the successfully loaded variants each time an action
starts (or a looping state is entered), then holds that choice until the next play.
Contact timing and duration come from that same variant. Files are cached across
characters; each rig receives its own retargeted clips. Missing files are skipped;
an empty pool uses the renderer's existing-clip fallbacks. Empty arrays are allowed.
Hip X/Z travel is stripped by default, including pack clips; no physics is driven by
these animations. Opposite keeper sidesteps are mirrored if only one file loads.
Every dive variant is checked before stripping travel: `gk_dive_l` travels in local
+X (the player's left for this +Z-facing rig), `gk_dive_r` in -X. Wrong-side variants
are mirrored individually, preserving the rest of each pool; a missing side is generated.

The following filenames are relative to `assets/mixamo/pack/`:

| Key | FBX file(s) | Visual trigger |
| --- | --- | --- |
| header | header_soccerball, header_soccerball_2 | Header contact, or new possession above 1.5 m, outside the keeper's own box |
| receive | receive_soccerball | First possession of an incoming teammate pass below 0.5 m |
| chest_knee | kneeing_soccerball, kneeing_soccerball_2 | Incoming teammate pass first touch from 0.5 through 1.5 m; header above 1.5 m |
| bicycle | scissor_kick | First-time volley on a ball above 1.4 m; anticipates contact while charging off the ball |
| juggle | kick_up_soccerball, stall_soccerball, stall_soccerball_2, stall_soccerball_3, stall_soccerball_4 | Occasional idle routine during draft or kickoff setup (`restart: kickoff`), excluding the taker and keepers |
| trip | soccer_trip | Stun begins (tackled/fouled player) |
| fallen | fallen_idle | After trip; holds until stun ends |
| get_up | standing_up | After fallen; deliberate kick/tackle or possession interrupts recovery |
| jog_back | jog_backward | Backward movement relative to facing |
| jog_diag_fl | jog_forward_diagonal_2 | Forward-left movement relative to facing |
| jog_diag_fr | jog_forward_diagonal | Forward-right movement relative to facing |
| jog_diag_bl | jog_backward_diagonal_2 | Backward-left movement relative to facing |
| jog_diag_br | jog_backward_diagonal | Backward-right movement relative to facing |
| strafe_l | jog_strafe_left | Left movement relative to facing |
| strafe_r | jog_strafe_right | Right movement relative to facing |
| gk_sidestep_l | goalkeeper_sidestep | Keeper shuffles left across own box |
| gk_sidestep_r | goalkeeper_sidestep_2 | Keeper shuffles right across own box |
| gk_scoop | goalkeeper_scoop | Keeper gains possession of a ball below 0.65 m |
| gk_block | goalkeeper_body_block, goalkeeper_body_block_2, goalkeeper_body_block_3 | Keeper deflects a nearby free ball (pickup cooldown begins without possession) |
| gk_miss | goalkeeper_miss | Incoming shot crosses behind keeper within goal mouth, or opponent scores |
| gk_drop_kick | goalkeeper_drop_kick | Keeper shoots from own box |
| gk_directing | goalkeeper_directing, goalkeeper_directing_2 | Idle keeper without possession organises when the ball is in the opponent's half, after a random delay |
| gk_pass | goalkeeper_pass | Keeper ground/through pass from own box (unspecified kind is ground) |
| gk_place | goalkeeper_placing_ball, goalkeeper_placing_ball_2 | Keeper owning the goal kick enters the kickoff setup wait (`restart: goalkick`); one-shot around placement contact |
| throw_in | throw_in | First committed kick by the owner at a throw restart |

Existing keys gain variants while retaining their original files; the two conditional
shot keys are listed alongside them:

| Key | Added pack FBX file(s) | Trigger |
| --- | --- | --- |
| idle | offensive_idle | Standing idle variant |
| jog | jog_forward | Forward jogging variant |
| pass | kick_soccerball, kick_soccerball_2 | Pass strike |
| shoot_jog | strike_foward_jog | Shot committed while moving above 3.6 m/s |
| shoot_penalty | soccer_penalty_kick | Shot below 0.8 m/s with charge at least 0.7, or a stationary free/corner restart shot |
| tackle | soccer_tackle, soccer_tackle_2, soccer_tackle_3 | Forward lunge/tackle |
| gk_catch | goalkeeper_catch, goalkeeper_catch_2, goalkeeper_catch_3 | Keeper receives a ball at least 0.65 m high; catch_3's run is stripped |
| gk_idle | goalkeeper_idle, goalkeeper_idle_2 | Stationary keeper without possession |
| gk_throw | goalkeeper_overhand_throw | Keeper lob/cross from own box |
| gk_dive_l | goalkeeper_diving_save | Keeper dives toward local +X (left); retains original dive |
| gk_dive_r | goalkeeper_diving_save_2 | Keeper dives toward local -X (right); retains original dive, mirrored to correct its +X drift |

`shoot_jog` and `shoot_penalty` are conditional shot pools, falling back to `shoot`.
They deliberately have separate keys so random selection cannot play a penalty kick
on a running shot. Selection reads the committed strike's charge and starting velocity;
charge has already been cleared by the simulation when the shot animation begins.
The ordinary `shoot` file and its charging wind-up remain available.

Directional movement uses eight 45-degree sectors with 6 degrees of hysteresis at
each boundary. Keeper box shuffles still use sidesteps in the two lateral sectors.
Gait changes preserve cycle phase and crossfade over 0.22 s. Playback uses the chosen
variant's measured ground speed; diagonal jog rates can exceed the older 1.45x cap
to keep their ~2.1-2.6 m/s authored strides aligned with game movement.

Juggles use the requested setup-wait option: animation-only FBXs contain 53 skeleton
tracks and no ball mesh/prop, so no extra ball is displayed and the match ball is
untouched. They last one routine, recur after 5-13 seconds, and yield immediately to
movement, possession or play. The kickoff team's slot-0 outfielder is excluded even
while the setup ball has no owner. Goal celebrations retain `celebrate`.

Reaction run-ups are skipped using measured foot, hand or head motion; short visual
locks keep locomotion responsive. Trip/fallen/get-up are a renderer-only sequence;
the simulation's stun duration, ball trajectories and restart rules are unchanged.
Throws therefore retain the existing simulation release position/trajectory.
First-touch height and incoming direction use the preceding rendered ball snapshot;
a contact occurring entirely between rendered frames can be missed. Replay frames
use the same state-based triggers and may choose a different cosmetic variant.

Offline inspection found 53 animation tracks in every readable pack FBX. The 27
previously unused files were inspected with the installed Three.js FBXLoader; net
hip travel below is in source FBX units, before stripping X/Z (vertical motion is kept).

| File | Length (s) | Hip delta X / Y / Z |
| --- | ---: | --- |
| goalkeeper_body_block_3 | 3.400 | +167.7 / -6.7 / +45.1 |
| goalkeeper_catch_3 | 2.667 | +145.8 / -12.0 / +835.1 |
| goalkeeper_catch_4 | parse failure | unavailable |
| goalkeeper_directing | 7.167 | +2.6 / +7.6 / +367.1 |
| goalkeeper_directing_2 | 7.700 | +27.8 / 0 / -192.4 |
| goalkeeper_diving_save | 3.200 | +443.4 / -5.2 / +368.0 |
| goalkeeper_diving_save_2 | 3.233 | -553.3 / -7.7 / +209.2 |
| goalkeeper_pass | 3.133 | +33.0 / +3.3 / +161.2 |
| goalkeeper_placing_ball | 3.500 | -1.8 / +7.7 / +245.8 |
| goalkeeper_placing_ball_2 | 7.967 | +15.7 / +7.7 / +674.1 |
| jog_backward_diagonal | 0.733 | -236.6 / 0 / -232.4 |
| jog_backward_diagonal_2 | 0.833 | +294.8 / 0 / -299.7 |
| jog_forward | 0.833 | 0 / 0 / +424.8 |
| jog_forward_diagonal | 0.800 | -322.2 / 0 / +322.3 |
| jog_forward_diagonal_2 | 0.833 | +318.9 / 0 / +314.9 |
| kick_up_soccerball | 1.333 | -11.8 / -2.4 / -15.5 |
| kneeing_soccerball | 0.900 | 0 / 0 / 0 |
| kneeing_soccerball_2 | 0.600 | +1.4 / +1.3 / -0.3 |
| offensive_idle | 10.367 | -1.9 / -0.4 / -1.6 |
| scissor_kick | 2.767 | +39.2 / 0 / -45.0 |
| soccer_penalty_kick | 1.500 | -160.7 / +15.8 / +592.1 |
| stall_soccerball | 3.267 | 0 / 0 / 0 |
| stall_soccerball_2 | 2.133 | 0 / 0 / 0 |
| stall_soccerball_3 | 2.300 | 0 / 0 / 0 |
| stall_soccerball_4 | 1.767 | +4.1 / -0.9 / +2.0 |
| strike_foward_jog | 1.267 | +0.2 / -2.9 / +602.9 |
| transition | 0.767 | -5.1 / +2.4 / -2.3 |

52 of the 54 pack files are now mapped. `goalkeeper_catch_4.fbx` still fails Three.js
FBX parsing (unknown property type) and is excluded. `transition.fbx` is a short,
nearly stationary pose transition with no clear football action; it stays unmapped.
Catch_3 now participates with X/Z motion stripped; contact is measured after horizontal
pinning so its long run does not dominate the contact heuristic. Knee first touches
use knee-bone speed; other contacts retain foot, hand or head speed timing.
No browser/visual playback review has been performed. Check diagonal cadence and
crossfades, bicycle landing/recovery, catch_3 pose alignment, directing loop seams,
goal-kick placement and the ball-free juggle/stall presentation in game.

## More characters (character select)

`characters` in the manifest lists the models players can pick on the CHARACTER
screen. Each one only needs its own model file: animations, `height`, `facing` and
`tintMaterial` come from `player` (override any of them per character if needed).

The six installed choices are below (paths relative to `public/assets/mixamo/`):

| Id | Name | Mixamo character | File | tintMaterial | tintMesh | Size (MB, decimal) |
| --- | --- | --- | --- | --- | --- | --- |
| striker | Striker | Remy | player.fbx | Top (inherited) | — | 28.34 |
| playmaker | Playmaker | Brian | player2.fbx | Ch01_body | Ch01_Shirt | 54.87 |
| captain | Captain | Kate | player4.fbx | Ch21_body | Ch21_Shirt | 53.09 |
| defender | Defender | Josh | player5.fbx | Ch23_body | Ch23_Shirt | 53.40 |
| runner | Runner | Shannon | player6.fbx | Ch38_body | Ch38_Shirt | 52.06 |
| sweeper | Sweeper | Jody | player7.fbx | Ch38_body | Ch37_Shirt | 52.96 |

The old `winger` entry was removed: `player3.fbx` is an animation duplicate of
`volley.fbx`, not a character. The file is retained and is not selectable.

To add another model, download **FBX Binary**, **With Skin**, T-pose from Mixamo,
save it here and add a unique id/name/url to `characters`. Reload the game; absent
files show as "not installed". Models load on first selection (or at startup for
a saved choice), with a loading note and tile state. The wrapping, scrollable
CHARACTER list is generated from the manifest; it has no fixed slot count.

The new FBXs contain embedded textures and total about 266.39 MB. Transfer size,
texture memory and synchronous FBX parsing remain follow-up performance concerns;
the loading indicator covers the asynchronous wait but cannot animate during
main-thread parsing. No texture-compression pipeline is included.

Mixamo animations work on any Mixamo character: bone names are matched (including
`mixamorig1:`-style prefixes) and hip movement is rescaled to the new rig's leg length.
For the colour pickers to work, mesh names should follow Mixamo's usual `Body`, `Hair`,
`Bottoms`, `Shoes` (also `ChNN_Body`, pants/shorts and sneakers). The shirt is found
by a case-insensitive `tintMaterial` substring. These new models share their body
material across clothing and skin, so `tintMesh` additionally restricts team tint
to an exact shirt mesh name; omitting it preserves material-only matching.

Offline FBXLoader inspection found these mesh/material names:

- Remy: `Shoes/Shoesmat`, `Tops/Topmat`, `Body/Bodymat`, `Bottoms/Bottommat`,
  `Eyes/Bodymat`, `Hair/Hairmat`, `Eyelashes/Eyelashmat`.
- Brian: `Ch01_Body`, `Ch01_Shirt`, `Ch01_Sneakers`, `Ch01_Pants` use `Ch01_body`;
  `Ch01_Eyelashes` uses `Ch01_hair` (no separate hair mesh).
- Kate: `Ch21_Pants`, `Ch21_Shirt`, `Ch21_Body`, `Ch21_Shoes` use `Ch21_body`;
  `Ch21_Eyelasshes`, `Ch21_Hair` use `Ch21_hair`.
- Josh: `Ch23_Shoes`, `Ch23_Body`, `Ch23_Suit`, `Ch23_Belt`, `Ch23_Shirt`,
  `Ch23_Pants` use `Ch23_body`; `Ch23_Eyelashes`, `Ch23_Hair` use `Ch23_hair`.
  Only the shirt is team-tinted; the suit and belt retain their original appearance.
- Shannon: `Ch38_Shorts`, `Ch38_Shirt`, `Ch38_Socks`, `Ch38_Body`, `Ch38_Shoes`
  use `Ch38_body`; `Ch38_Eyelashes`, `Ch38_Hair` use `Ch38_hair`.
- Jody: `Ch37_Pants`, `Ch37_Sneakers`, `Ch37_Shirt`, `Ch37_Body`, `Ch37_Zipper`
  use `Ch38_body`; `Ch37_Eyeleashes`, `Ch37_Hair` use `Ch37_hair`.

```json
"characters": [
  { "id": "striker", "name": "Striker", "url": "assets/mixamo/player.fbx" },
  { "id": "playmaker", "name": "Playmaker", "url": "assets/mixamo/player2.fbx", "tintMaterial": "Ch01_body", "tintMesh": "Ch01_Shirt" },
  { "id": "captain", "name": "Captain", "url": "assets/mixamo/player4.fbx", "tintMaterial": "Ch21_body", "tintMesh": "Ch21_Shirt" },
  { "id": "defender", "name": "Defender", "url": "assets/mixamo/player5.fbx", "tintMaterial": "Ch23_body", "tintMesh": "Ch23_Shirt" },
  { "id": "runner", "name": "Runner", "url": "assets/mixamo/player6.fbx", "tintMaterial": "Ch38_body", "tintMesh": "Ch38_Shirt" },
  { "id": "sweeper", "name": "Sweeper", "url": "assets/mixamo/player7.fbx", "tintMaterial": "Ch38_body", "tintMesh": "Ch37_Shirt" }
]
```

## General format

Drop `.glb` files in this folder and point `manifest.json` at them. Any entry left
`null` (or that fails to load) falls back to the built-in placeholder.

```json
{
  "player": {
    "url": "assets/player.glb",
    "scale": 1,
    "facing": "+z",
    "tintMaterial": "Jersey",
    "clips": { "idle": "Idle", "run": "Run", "kick": "Kick", "tackle": "Slide" }
  },
  "keeper": null,
  "ball":  { "url": "assets/ball.glb", "scale": 1 },
  "arena": { "url": "assets/arena.glb", "scale": 1 }
}
```

- **player / keeper** – rigged character, origin at the feet, about 1.8 units tall.
  `facing` is the axis the model looks along (`+z` is the glTF/Blender-export default).
  Materials whose name contains `tintMaterial` are recoloured per team.
  `clips` maps game states to animation names in the file; missing clips are skipped.
  `keeper` falls back to `player`.
- **ball** – centred on the origin, radius 0.11 units at scale 1 (`C.ballR`).
- **referee** – loads `public/assets/referee/drone.glb`. The game keeps the model's geometry and materials,
  uniformly fits it to a 1.25 m maximum span, and plays its embedded animation during matches.
  Set `facing` to the model's forward axis if it differs from `+z`.
- **goal** – replaces the procedural goal frame and net at both ends (visual only).
  Origin anywhere: it is placed automatically with the front of the posts on the goal
  line, centred, standing on the turf. `facing` is the way the open mouth looks
  (default `+z`). Without `scale`/`height` it is stretched to the gameplay goal box
  (7.2 m wide, 2.8 m high, 2.2 m deep) so the frame matches where the ball bounces.
  `frame` (optional regex) names the meshes to fit by, so extras such as rear
  stanchions are left out of the fit. A missing file falls back to the built-in goal.
- **arena** – replaces the placeholder pitch, walls, goals and stands. It is visual
  only: gameplay dimensions live in `src/config.ts` (pitch 44 × 26, goals 7.2 wide,
  2.8 high, centred on the origin, long axis on X).


## Credits

- `referee/drone.glb`: [Futuristic flying animated Robot - Low Poly](https://sketchfab.com/3d-models/futuristic-flying-animated-robot-low-poly-c5b92c281dc444448e64c0607719c7a2)
  by [Shayan](https://sketchfab.com/mshayan02), licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
  The GLB is Sketchfab's conversion of the original FBX. The game places and uniformly scales it at runtime.
- `goal/`: "Football Goal" (https://sketchfab.com/3d-models/football-goal-bf20be6e423e41d191134ab77c590992)
  by romullus (https://sketchfab.com/romullus), licensed under CC-BY-SA-4.0
  (http://creativecommons.org/licenses/by-sa/4.0/). Used unmodified apart from runtime
  scaling; see `goal/license.txt`.
