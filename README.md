# PICKLE CITY VR

A fast, vertical, open-city WebXR action game starring one very serious anthropomorphic pickle.
Runs in any modern browser: desktop (keyboard + mouse) or VR headset (Quest Browser, SteamVR/Chrome/Edge, etc.).
No Unity/Unreal/native app. Everything (textures, models, audio) is generated procedurally in code — there are **no external asset files**.

## Run it locally

It is a static site (ES modules + import map). Serve the folder over HTTP — opening `index.html` via `file://` will not work.

```bash
cd pickle-city-vr
npm run dev            # = npx http-server -p 8080 -c-1   (or: python3 -m http.server 8080)
# open http://localhost:8080
```

`http://localhost` counts as a secure context, so WebXR works on the same machine. 

### Three.js source
`index.html` first looks for a local copy at `vendor/three/` and otherwise loads Three.js r170 from jsDelivr.
To go fully offline / self-hosted:

```bash
npm i three@0.170.0
mkdir -p vendor/three/build vendor/three/examples
cp node_modules/three/build/three.module.js node_modules/three/build/three.core.js vendor/three/build/
cp -r node_modules/three/examples/jsm vendor/three/examples/
```

## Entering VR

1. Put on the headset and open the page in its browser (or use a PC-VR browser with the headset connected).
2. If a headset is detected, a big **ENTER VR** button appears on the menu. Otherwise the game falls back to desktop mode automatically.
3. Click it. The game loads (LOADING PICKLE CITY), then click **ENTER VR** once more (browsers require a click to start a session).
4. Allow any browser prompt for VR/“immersive” access. No camera/microphone permission is needed. Audio starts on your first click.

**Testing from a headset on your LAN:** WebXR requires HTTPS (or localhost). Options: `adb reverse tcp:8080 tcp:8080` on a Quest and open `http://localhost:8080`; or run `npx http-server -S -C cert.pem -K key.pem` with a self-signed cert; or deploy (below).

## Deploy (HTTPS)

Any static host with HTTPS works: GitHub Pages, Netlify, Cloudflare Pages, Vercel, S3+CloudFront. Upload the project as-is (no build step). Make sure `index.html`, `css/`, `src/` (and `vendor/` if you self-host Three.js) are served. If the page is served over plain HTTP the menu shows “WebXR needs HTTPS” and desktop mode still works.

## Desktop controls (remappable in CONTROLS)

| Key | Action |
|---|---|
| W A S D | Move |
| Mouse | Aim · LMB fire · RMB aim down sights |
| Shift | Sprint |
| Space | Jump · in the air: open the glider |
| Ctrl | Crouch · while sprinting: slide |
| R | Reload |
| E | Interact: doors, buttons, grab/throw crates |
| G | Glider toggle |
| 1–6 / Q / wheel | Weapon select |
| V | Third-person view · F3 FPS counter · Esc pause |

## VR controls (right-handed; “left-handed” in the menu swaps sides)

| Input | Action |
|---|---|
| Left stick | Move (or hold forward to aim the teleport arc, release to blink-teleport) |
| Right stick | Snap turn (default 30°) or smooth turn |
| Left stick click | Toggle sprint |
| X | Crouch toggle · while moving fast: **slide** (dropping into a physical crouch while running also slides) |
| Y | Pickle Menu: comfort/locomotion settings, height calibration, exit VR |
| A (right) | Jump · in the air: open glider · gliding: close |
| Trigger | Gun in hand: fire · Empty hand: press UI/buttons, or point at a dropped (or world) weapon and pull — it **flies into your hand** |
| Fast hand swing into a civilian | Punch (no grab needed): light = stagger, hard = they go flying. Civilians you shoot, hit or grab get **angry** and slowly chase you with fists (5 damage a hit, no weapons) |
| Grip | **Hold grip to hold things.** You start empty-handed; pouches sit at your left and right hip. Empty hand at a holster + grip = draw. You keep a gun only while gripping: let go at a holster = put away, anywhere else = **drop it**. Also grab crates, doors, and **any part of a civilian** (release to throw) |
| B (right) | Reload (or reach your free hand to the magazine well and grip) |
| Free hand grip on the foregrip | Two-handed hold (SMG, rifle, shotgun, DMR) |
| Glider | Hold both grips on the bar: lower right hand = turn right, reach forward = dive, pull in = flare. Sticks also steer |

Comfort: snap/smooth turn, turn speed, snap angle, move speed, vignette strength, seated/standing, height calibration, handedness, teleport locomotion, move-relative-to head or hand. The headset camera is **never** rotated by the game — only the play space moves, pivoting around your head.

## Cars, police & money
- **Traffic**: cars follow lanes, obey traffic lights, keep a following distance, turn at junctions, show brake lights and have a driver at the wheel. Shoot a car 5 times and it explodes.
- **Steal a car (VR)**: grab the driver with the grip (anywhere near the driver's seat) to yank them out, then **click a thumbstick** next to the car to get in. Click a thumbstick again to get out.
- **Drive (VR)**: right trigger = gas, left trigger = brake/reverse, left stick = steer, A = handbrake, B = horn.
- **Drive (desktop)**: F = get in/out (pulls the driver out), W/S = gas/brake-reverse, A/D = steer, Space = handbrake, H = horn, V = camera.
- **Police**: hurt/kill more than 3 innocents and a police car chases you; a cop gets out and shoots. Get far away to lose them.
- **Bank**: the green BANK sign (yellow dot on the minimap). Point a gun at the teller (about 0.7 s) for $300–$800. Pointing a gun at a random civilian has a 40% chance of $100.
- Wrist display: health, shield, money, weapons and a minimap.
- You no longer start holding a gun: draw from a hip holster (VR) or press 1–6 (desktop).

## Gameplay quick-start

- You spawn on a street in the south-west of a 7×7-block city (`~560 m`). A **BIG DILL .44** revolver is near the spawn, other weapons are on rooftops / the construction crane.
- Neon **jump pads** chain: Sprint → Pad → Launch → Glider → Rooftop. Locked pads are opened by pressing the rooftop **buttons** (poke in VR, E on desktop).
- Pickups: green cross = health, yellow = ammo, blue = shield, orange bolt = speed boost, magenta beam = weapon.
- Civilians are peaceful and unarmed. Shooting near them makes them run and shout. If you shoot, punch or grab one, **that one gets angry**: it slowly chases you and swings fists (5 damage per hit). They never carry weapons.

## File structure

```
index.html            page shell, HUD / menu / loading DOM, Three.js import map
css/style.css
src/main.js           bootstrap
src/Game.js           renderer, system wiring, loading sequence, frame loop
src/core/             Utils, SettingsManager, AudioManager (procedural WebAudio), PerformanceManager, Particles, Geo
src/world/            Constants, CityLayout (data), Materials (procedural PBR), Batcher, ChunkBuilder, Districts,
                      CollisionWorld (AABB), CityStreamingSystem, WorldManager (sky, sun, traffic, birds)
src/player/           PlayerController, LocomotionSystem, CameraController, DesktopController,
                      VRPlayerController, VRHandController, HandModel, PickleAvatar, GliderSystem
src/weapons/          WeaponDefs, WeaponModels, WeaponInventory, WeaponSystem
src/gameplay/         HealthSystem, JumpPadSystem, PickupSystem, InteractionSystem
src/npc/              NPCSystem (pooled/instanced/LOD), NPCNavigation, NPCReactionSystem
src/ui/               UIManager (DOM), VRUIManager (wrist display + pop-up menu + world prompts)
```

## Performance recommendations

- Quality **Auto** adapts pixel ratio / shadows / civilian count from measured FPS; on standalone headsets set **Low** and shadows **Off** for the steadiest 72–90 Hz.
- Chunk streaming keeps only a 3×3 area at full detail; far blocks are cheap shells. NPCs, cars, pads, pickups, tracers and particles are instanced/pooled.
- Fixed foveation is enabled; framebuffer scale follows the quality preset.
- Serve with gzip/brotli; the whole game is a few hundred KB of JS.

## Placeholder / procedural assets

There are no binary assets. Placeholders that you may want to replace with authored art later: all textures (canvas-generated), all meshes (boxes/lathes/merged primitives), the pickle avatar (faceted ellipsoid + boxes), weapons (original low-poly designs: PEBBLE-9, BIG DILL .44, BRINE-9 SMG, GHERKIN AR-30, BRINE-BUSTER 12, LONG-HAUL DMR), civilians (instanced primitives), and all sounds (synthesised buffers, including voices for NPC shouts as filtered noise “murmurs”).

## Status / known limitations

This build was written and statically checked (syntax + import/export cross-check + unit tests of the layout and collision logic) in a sandbox with no network access, **so it has not been run in a real browser or headset yet**. Expect to fix a few runtime errors on the very first launch — open the browser console (`window.pickleCity` is exposed for debugging) and report anything red.
Known simplifications: warehouses are solid masses, traffic cars pass through each other at intersections, civilians have no voice lines (synth shouts only), no multiplayer.

## Guns: bullet drop & manual reload
- Bullets are no longer instant: each weapon has a muzzle velocity (`vel` in `WeaponDefs.js`) and the shot falls under gravity, so aim above distant targets (sniper most of all).
- VR manual reload (magazine guns): **B** drops the magazine, reach the **belt pouch** (front of your waist) with the off-hand and hold **grip** to pull a fresh mag, bring it to the gun's mag well to seat it. Empty chamber = auto rack. Pressing B again with the mag out does an assisted reload. Revolver/shotgun: grab one shell at a time from the pouch and push it into the gun.
- Empty trigger in VR is a dry click (no auto reload). Desktop: **R** reloads as before.
- Models were rebuilt with extra detail (serrations, rails, vents, sights, scope turrets, brushed metal / stippled polymer / grained wood).

## City & interiors remodel
- Facades: individual bricks, stone window surrounds, sills, curtains/blinds/AC units, storey lines, cornices, plinths and bump mapping.
- Ground floors are furnished and lit: shops (stocked shelves, chilled cabinets, checkout, posters), restaurants (tables, pendant lamps, kitchen pass, bar stools), the bank (marble hall, queue ropes, desks, ATMs, vault door) and walk-in **apartment lobbies** (reception, mailboxes, lift, sofas, chandelier) on many apartment blocks. Interior walls are plastered with wainscot, skirting and crown moulding.
- Your apartment has trim, curtains and radiators, a full kitchen, bedroom, workspace and entry details.

## Joining the city
You start in your apartment. Look at the **TV**: aim at **JOIN SERVER** and pull the trigger (VR) or click / press E (desktop). A short "CONNECTING…" screen and you arrive in Pickle City. The old walk-out-the-door teleport is gone. The Pickle Menu's GO HOME returns you to the apartment.

## Multiplayer (artifact room)
When the page is opened as a published artifact by signed-in people who can open it (the owner's organisation, or people it is shared with), everyone who has it open at the same moment shares one lobby through the artifact **room** capability (`src/net/NetworkSystem.js`).
- Every player in the city appears as a pickle with a coloured cap, name plate (PICKLE-XXXX) and health bar; in VR you see their head, hands and held weapon, on desktop the gun follows their aim; drivers appear in a simple car.
- Shots (tracers + sound) and damage are synced: hit the other player to hurt them (headshots 2.2×); when your hit kills someone you earn **$500**. You respawn like normal.
- The TV browser shows ONLINE with the player count and names, or OFFLINE when no room is available (public link / older viewer): the game then plays single-player as before.
- Not synced (each player has their own copy): civilians, traffic, police, pickups and doors. Players in apartments are hidden from the city.
- Contributors and above can send shots/hits; Viewers/Commenters can watch but not fight (platform rule). Presence runs about 12 times a second; shots are rate-limited to stay inside the room's ~40 messages/second budget.

## Public multiplayer server (anyone can join with a link)
`server/server.cjs` is a zero-dependency Node program that **serves the game and relays multiplayer** (WebSocket at `/ws`). Run it anywhere that keeps a Node process alive and gives you HTTPS (WebXR needs HTTPS):

```bash
node server/server.cjs            # http://localhost:8080  (PORT=… MAX_PLAYERS=32 ALLOW_ORIGIN=…)
```

Deploy options (the game then lives at a normal public URL; send it to people and they join the same city):
- **Render.com**: push this folder to a GitHub repo, "New → Blueprint" (uses `render.yaml`), or "New Web Service" with start command `node server/server.cjs`. Free plans sleep after idle, so the first visit takes ~30 s.
- **Fly.io / Railway / any VPS or Docker host**: `Dockerfile` included (`docker build -t pickle . && docker run -p 8080:8080 pickle`).
- Put it behind HTTPS (Render/Fly/Railway do this automatically). The client picks `wss://` automatically.
- Game on a different host than the relay? Open the game with `?server=wss://your-relay.example/ws` or set `window.PICKLE_SERVER`.

The relay keeps no world state: it forwards each player's pose (presence) and shot/hit/frag events, rate-limits each connection (40 msgs/s), caps the lobby (32) and only allows the known event topics. Civilians, traffic and police are still local to each player. Inside claude.ai the artifact uses the built-in artifact room instead.

## Mobile / touch controls
On phones and tablets (or with **TOUCH CONTROLS: ON** in the main menu) PLAY uses on-screen controls and goes fullscreen/landscape:
- **Left side**: floating move stick (push to the edge to sprint; also steers/gas/brake in a car).
- **Right side**: drag anywhere to look.
- Buttons: FIRE, AIM (toggle), JUMP, CROUCH (toggle), RELOAD, USE (doors, TV JOIN SERVER button, pickups), CAR (get in/out), SWAP weapon, GLIDE, VIEW (3rd person), II (pause).
- Multiplayer, guns and driving all work the same as desktop. Performance: set Quality to Low in Settings on older phones.
