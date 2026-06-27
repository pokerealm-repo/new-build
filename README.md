# PokéMMO — Realtime Multiplayer Overworld

A clean, self-contained recreation of the core of
[aaron5670/PokeMMO-Online-Realtime-Multiplayer-Game](https://github.com/aaron5670/PokeMMO-Online-Realtime-Multiplayer-Game):
a tile-based Pokémon overworld where **multiple real players share the same
world in real time**.

Built with **Phaser 3** (client) and **Colyseus** (realtime server), modernized
to a single repo with **Vite** instead of Webpack. No paid APIs — it runs
entirely on your machine.

![overworld](../docs/images/PokeMMO.gif)

## Features

- **Buy-in + gacha starter** — to play, connect a Solana wallet and pay a small
  **0.05 SOL buy-in** (→ treasury). That mints your starter as a **Poké Ball
  gacha pull** — a random low/medium-tier Pokémon revealed with an animated
  sprite. You can pull again any time for 0.05 SOL.
- **Create your trainer** — on the title screen, pick a name and one of **14
  characters** (plus an optional pastel outfit tint) with a live pixel-art
  preview. Your avatar is sent to the server and **everyone sees the trainer you
  chose** when you meet them.
- **Realtime multiplayer** — every other trainer you see is a live player.
  Avatar, position, facing, and walk animations sync over WebSockets via
  Colyseus.
- **Full-screen view** — the game fills the window at any resolution with a
  classic handheld-sized pixel-art camera zoom.
- **Music** — separate landing + world tracks that crossfade when you enter,
  with a speaker/volume control on the title screen and in the HUD (remembered
  between sessions). Drop your `.mp3`s in [client/public/audio/](client/public/audio/).
- **Solana wallet login** — connect Phantom / Solflare (or any injected
  `window.solana`) on the landing, or as a guest. Your short address shows on
  your name tag and your live **SOL balance** shows in the HUD.
- **A world to explore** — five connected places (**Pallet Town → Route 1 →
  Ashveld Town → Route 2 → Crysthaven City**) you can walk between *or* **instantly
  fast-travel** to from the 🗺 Travel menu. Routes have tall grass with their own
  **wild Pokémon** species; towns are safe hubs full of NPCs.
- **NPCs** — walk up to a shopkeeper and press **Space** (or the mobile **A**
  button) to talk: **Merchants** sell balls & potions, **Blacksmiths** sell
  weapons, **Breeders** sell **Pokémon**, and **Nurses** heal you for free.
- **Battle mode** — every trainer gets a random **starter Pokémon** on join.
  Visit the **Battle Arena** in Crysthaven City for FireRed-style turn-based
  battles (FIGHT / BAG / RUN) with animated sprites — win for **Gold + XP**, and
  your lead Pokémon **levels up**. Pick your fighter in the Pokémon tab.
- **A story quest — "The Lost Treasure"** — Prof. Oak, a riddling Hermit and an
  Old Sailor send you on a treasure hunt with **branching dialogue and choices**
  (answer the Hermit's riddle!). Clues fill a **Journal**, and each clue makes a
  **✨ dig site** appear in the world — dig it up for gold, rare items and the
  next clue, all the way to a legendary Pokémon.
- **On-chain economy + progression (mainnet)** — buy **Gold** with real SOL
  (your wallet signs + submits straight to the treasury `EeEM2sF…bALb`; we never
  custody keys, tx linked on Solscan), then spend Gold at shops on Poké Balls,
  Potions and Weapons. Throw a ball at a wild Pokémon to **catch** it for **XP +
  Gold** — caught and bought Pokémon fill your **collection**. **Level up** to
  grow HP / ATK / DEF (weapons add more); your level shows on your name tag. A
  tabbed **Menu** holds Trainer stats, your Pokémon, Bag, Shop and the SOL top-up.
- **Mobile** — auto-detects touch, rotates to landscape for play, and shows a
  virtual **joystick** + action buttons (Menu / Chat) with a slimmed,
  non-overlapping HUD. Preview the mobile layout on desktop with `?mobile=1`.
- **Two connected maps** — walk off the edge of **Town** to reach **Route 1**
  and back. Each player only renders the trainers on their current map.
- **Grid-style movement** — arrow keys / WASD, 4-direction walk cycles, camera
  follow, world-bounds + tile collision (classic "head overlaps the tile above"
  hitbox).
- **Tall grass** — step into the grass on Route 1 for a rustle (and the
  occasional "a wild Pokémon is near!").
- **Chat** — press <kbd>Enter</kbd> to chat with everyone in the world.

The art (Tuxemon sample tileset + character atlases) and the Tiled maps come
from the original open-source project.

## Wallet & economy

Configure brand/chain in one place: [client/src/config.js](client/src/config.js)
(treasury, Helius RPC, ticker, `COINS_PER_SOL`, SOL packs). All of it can be
overridden **without a rebuild** via a `window.POKE_CONFIG` block in
[client/index.html](client/index.html) — handy after deploy.

Wallet logic lives in [client/src/wallet.js](client/src/wallet.js): `connectWallet`,
`getBalance` (read-only via Helius), and `buy` (a `SystemProgram.transfer` the
wallet signs). It's the player→treasury direction only (buying). A treasury→player
payout (selling PokéCoins back for SOL) would need the treasury secret key on a
server and is intentionally not included.

## Run it

Requires Node 18+.

```bash
cd pokemmo
npm install          # installs the root dev runner (concurrently)
npm run install:all  # installs client + server deps
npm run dev          # starts the Colyseus server (:3020) and Vite client (:5285)
```

Then open **http://localhost:5285**. Open it in a **second tab** (or another
device on your network) to see two trainers in the same world.

You can also run the two halves separately:

```bash
npm run server   # Colyseus on :3020  (dashboard at /colyseus)
npm run client   # Vite dev server on :5285
```

## How the multiplayer works

```
Phaser client  ──(joinOrCreate "poke_world")──▶  Colyseus server (PokeWorld room)
   │  PLAYER_MOVED / PLAYER_MOVEMENT_ENDED            │  keeps last x/y/map/name
   │  PLAYER_CHANGED_MAP / PLAYER_CHAT                │  per session
   ◀── CURRENT_PLAYERS / PLAYER_JOINED / PLAYER_LEFT ─┤  rebroadcasts to the room
   ◀── PLAYER_MOVED / ... / ROSTER / CHAT ────────────┘
```

The client owns local movement; the server is the source of truth for *who is
where*. When you change maps the server sends you the full roster so you can
draw everyone already standing there.

## Project layout

```
pokemmo/
├─ server/                 # Colyseus realtime server
│  ├─ index.js
│  └─ rooms/PokeWorld.js   # join/leave + movement/chat relay
└─ client/                 # Phaser 3 + Vite
   ├─ index.html           # landing + HUD (status, count, chat)
   ├─ src/
   │  ├─ net.js            # single Colyseus connection, fan-out to subscribers
   │  ├─ main.js / boot.js # game factory + DOM glue
   │  ├─ scenes/           # BootScene (load) + WorldScene (play)
   │  └─ entities/         # Player + OnlinePlayer
   └─ public/assets/       # tileset, Tiled maps, character atlases
```

## Deploying

See **[DEPLOY.md](DEPLOY.md)** for a step-by-step guide (GitHub Desktop →
Render for the server → Vercel for the client). In short: the client is a static
Vite build (set `VITE_SERVER_URL=wss://your-server` at build time) and the server
is a Node web service that listens on `process.env.PORT` (a [render.yaml](render.yaml)
blueprint is included).
