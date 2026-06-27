# Deploying PokéMMO

Two pieces ship separately:

| Piece | What | Host | Folder |
|-------|------|------|--------|
| **Server** | Colyseus realtime room (WebSockets) | **Render** (Node web service) | `server/` |
| **Client** | Phaser game (static files) | **Vercel** (static) | `client/` |

You deploy the **server first**, copy its URL, then build the **client** pointing at it.

---

## Part 0 — Put the project on GitHub (GitHub Desktop)

The folder is already a Git repo with everything committed, so this is quick.

1. Open **GitHub Desktop**.
2. **File → Add Local Repository…** and choose the folder
   `…/USELESS JOURNEY/pokemmo`. It opens with **"No local changes"** (already committed).
3. Click **Publish repository** (top bar / the blue button).
   - Name: `pokemmo` (or anything).
   - **Uncheck** "Keep this code private" for the easiest Render/Vercel import
     (private also works — you just authorize access during import).
   - Click **Publish Repository**.

Your code is now at `https://github.com/<you>/pokemmo`. Every future
**Commit to main → Push origin** in GitHub Desktop auto-redeploys both hosts.

---

## Part 1 — Deploy the server on Render

1. Go to **https://dashboard.render.com** → **New +** → **Blueprint**.
2. Connect your GitHub and pick the **`pokemmo`** repo. Render reads
   [`render.yaml`](render.yaml) and proposes a service **`pokemmo-server`**.
3. Click **Apply**. Wait for the build (`npm install`) and first deploy.
4. When it's live, copy its URL, e.g. `https://pokemmo-server.onrender.com`.
   - Test it in a browser — you should see `{"status":"ok",…}`.

> Free Render services sleep after ~15 min idle; the first connection after a
> nap takes ~30 s to wake. Fine for testing; upgrade for always-on.

*(No Blueprint? Do it manually: New + → **Web Service** → pick the repo →
Root Directory `server` → Build `npm install` → Start `node index.js` → Create.)*

---

## Part 2 — Deploy the client on Vercel

1. Go to **https://vercel.com** → **Add New… → Project** → import the `pokemmo` repo.
2. In **Configure Project**:
   - **Root Directory** → click *Edit* → choose **`client`**.
   - Framework Preset should auto-detect **Vite** (Build `npm run build`, Output `dist`).
3. Open **Environment Variables** and add ONE:
   - **Key**: `VITE_SERVER_URL`
   - **Value**: `wss://pokemmo-server.onrender.com`  ← your Render URL, but with
     **`wss://`** instead of `https://` (it's a WebSocket).
4. Click **Deploy**. When done you get a URL like `https://pokemmo.vercel.app`.

Open it in two tabs (or send it to a friend) — you're live and multiplayer. 🎉

---

## Updating after launch

- **Code change** → in GitHub Desktop: **Commit to main → Push origin**. Render and
  Vercel both auto-redeploy.
- **Change treasury / RPC / X & buy links / music** without a rebuild → edit the
  `window.POKE_CONFIG` block in [`client/index.html`](client/index.html) and push.
- **Custom domain** → add it in the Vercel project's **Domains** tab.

## Troubleshooting: "Deployment Blocked — author does not have a Vercel account"

Vercel blocks a deployment when the **commit author's email** maps to a GitHub
account that isn't connected to your Vercel team. This is an authorization check,
**not** a build error — your code is fine.

Fix it one of these ways:
- **Easiest:** make sure every commit is authored by the GitHub account that owns
  the Vercel project. In **GitHub Desktop → Settings → Git**, set your name/email
  to that account's email (for `shelbybrothers` that's
  `100958717+shelbybrothers@users.noreply.github.com`). Commit + push again.
- Or in Vercel, click **Redeploy** on the blocked deployment (you're the project
  owner, so a manual redeploy is authorized).
- Or **Invite to Team** / connect the other GitHub account to Vercel.

To check who authored your commits: `git log --format='%h %an <%ae>'`.

## Notes
- The wallet treasury, Helius RPC, and `$POKE` links are all client-side config in
  [`client/src/config.js`](client/src/config.js) (overridable via `POKE_CONFIG`).
- The server holds no secrets and needs no env vars (Render supplies `PORT`).
- Player progress (gold/level/inventory) is saved in the browser's localStorage,
  keyed by wallet address (or "guest").
