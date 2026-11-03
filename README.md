# AIV X — Your Everyday AI

AIV X is AI Verse's everyday AI website: chat, uploads, web search, deep research, image creation,
documents, spreadsheets, presentations, Build, app modes, and per-user memory.

It runs as its own website with its own API:

| Piece | Where it lives |
|---|---|
| Website (`index.html`, `icon.svg`, `manifest.webmanifest`, `404.html`) | Its own GitHub repo, e.g. `domvq/AIVX`, on GitHub Pages |
| AIV X API (`aivx-worker.js`) | Its own Cloudflare Worker, e.g. `aivx-api` |
| AIV X app key | `aivx_iGsNWk4eHXokoW8awHmMVZpm` — in `index.html` **and** the Worker's `AIVX_APP_KEY` secret |
| Accounts + memory | Firebase (Auth + Firestore) |

## 1. Deploy the AIV X API (Cloudflare)
1. Workers & Pages → Create → Worker → name it `aivx-api` → paste `aivx-worker.js` → Deploy.
2. Settings → Bindings → Add → **Workers AI** → variable name `AI`.
3. Settings → Variables and Secrets → add secrets:
   - `AIVX_APP_KEY` = `aivx_iGsNWk4eHXokoW8awHmMVZpm`
   - `SUBSCRIBER_KEYS` = your AIV Pro keys, comma-separated
   - (optional) `BRAVE_KEY` for better web search
4. (Optional, recommended) Storage → KV → create `aivx-limits`, then bind it to the Worker as `LIMITS`.
   This blocks heavy free use on the server side (default 60 AI calls per visitor per day; change with a `FREE_DAILY_CAP` variable).
5. `SITE_ORIGINS` in `aivx-worker.js` is already set to aiv-x.com.
6. Copy the Worker URL (`https://aivx-api.domhasaball.workers.dev`) into `AIV_API_URL` at the top of `index.html`.

## 2. Turn on accounts (Firebase)
1. console.firebase.google.com → Add project → add a **Web app** → copy its config into `FIREBASE_CONFIG` in `index.html`.
2. Authentication → Sign-in method → enable **Google**, **Apple**, **Phone**, **Email/Password**.
3. Authentication → Settings → Authorized domains → add `aiv-x.com`, `www.aiv-x.com`, and `domvq.github.io`.
4. Firestore Database → Create → Rules:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{db}/documents {
       match /users/{uid}/{doc=**} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```
Until `FIREBASE_CONFIG` is filled in, AIV X runs in demo mode (accounts saved in the browser only).

## 3. Publish the website (GitHub Pages)
1. Create a new repo (e.g. `AIVX`) and upload these files.
2. Settings → Pages → Deploy from branch → `main` / root.
3. Custom domain: enter `aiv-x.com` in Settings → Pages. At your domain registrar, add four A records for `@` (185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153) and a CNAME record for `www`
   pointing to `domvq.github.io`. GitHub adds the `CNAME` file for you.

## Rotating the app key
Generate a new value, put it in both `AIVX_APP_KEY` (Worker) and `AIVX_APP_KEY` (index.html), redeploy both.
The app key identifies AIV X to its API; the origin check plus the optional KV limit are what stop other sites from using it.
