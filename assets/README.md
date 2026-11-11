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
- **arena** – replaces the placeholder pitch, walls, goals and stands. It is visual
  only: gameplay dimensions live in `src/config.ts` (pitch 44 × 26, goals 7.2 wide,
  2.8 high, centred on the origin, long axis on X).
