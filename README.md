# MC Map Painter

A browser tool that turns any image into a build guide for **Minecraft map art** — the
"staircase" style where each block's shade depends on its height relative to its northern
neighbour (▲ raised / ■ flat / ▼ lowered).

Drop in an image and the app dithers it to the real Minecraft map palette, then gives you a
zoomable, pannable block grid with per-block material info, a chunk-by-chunk column build
list, a materials shopping list, and PNG export.

**Live app:** https://galileo81580-dotcom.github.io/mcmappainter/

## Features

- **Load any image** — drag & drop, file picker, or the built-in sample. Automatic
  Floyd–Steinberg / ordered dithering to the Minecraft palette (no external pre-processing).
  The **Sample** button loads the original pre-dithered art at the original app's exact
  settings (12×6 maps, zoom 8, world origin −26176/6208, nearest-match) — a verified
  pixel-perfect, block-aligned reproduction of the legacy Cloud Storage version.
- **Adjustable map size** — set how many maps wide × tall (1 map = 128×128 blocks) with
  contain / cover / stretch fit.
- **Modern palette** — all 61 paintable map colours of current Minecraft (Java 1.21.x),
  including cherry, bamboo, mangrove, resin, tuff/copper variants, mud, froglights, pale
  moss and the deepslate family.
- **Block preferences** — for every map colour, rank the blocks you'd use in preference
  order (preferred + backups) and exclude any you don't have. A fully-excluded colour is
  dropped from matching. Choices persist in your browser. The guide, tooltips and shopping
  list all follow your picks; the block inspector tooltip shows the alternatives.
- **Fast viewport rendering** — renders only what's visible, so it stays smooth at any zoom
  even on multi-map builds (the original redrew every pixel every frame).
- **Block inspector** — hover any block for its material, placement, alternatives, and
  in-game / chunk coordinates (set your upper-left world X,Z).
- **Column build list** — click a block to see the 16-block chunk column; step through
  neighbouring columns with the on-screen buttons or the arrow keys (← → ↑ ↓).
- **Painting progress** — click maps on the minimap to cycle **done (green ✓) → skip (red ✗)
  → clear**. Done/skip are shown on the canvas and minimap with a progress count; skipped
  maps are excluded from the shopping list.
- **Materials shopping list** — total blocks per material, stack count, across painted maps.
- **Saved workspaces** — save your settings, block preferences, and done/skip progress as
  named workspaces and reload them from a dropdown. Works locally out of the box; with a
  Firebase config it becomes shared cloud storage (see below).
- **Multi-user presence** *(with Firebase)* — when several people open the same workspace,
  each person's highlighted column shows live in their own colour, and an online list shows
  who's working where. Presence **auto-idles after 15 min** without touching the map (so a
  long-open tab stops showing a stale column), and an optional **"I'm in Minecraft" toggle**
  lets people flag that they're actually in-game (Realms can't report this automatically).
- **Export** — the dithered result at 1× (1 px per block) or the current view as PNG.

## Usage

It's a single self-contained `index.html` — open it directly, or serve the folder:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Optional: shared workspaces + multi-user (Firebase)

Saved workspaces work locally (browser storage) with no setup. To make them shared across
people and devices — and to enable live multi-user presence — add a Firebase project. This app
uses only the **Realtime Database** (both workspaces *and* presence live there) plus
**Authentication**, and avoids Firestore, Cloud Storage, and Cloud Functions — so it stays within
the **free tier**. (If your project already has billing enabled it'll be on the **Blaze** plan,
which *includes* that same free tier — this app's usage is negligible, so it's effectively $0.
Set a GCP **budget alert** for peace of mind.) Note: you do **not** need Firestore — if the
console shows Firestore in "Datastore mode" (common on old App Engine projects), ignore it.

**Access model:** users **sign in with GitHub** to create/edit shared workspaces and publish
images; that same sign-in also provides the repo token for publishing (no manual PAT). Anyone
can instead **continue as a guest** for read-only viewing — guests can open workspaces and watch
others' highlighted columns, but can't save, edit, or publish.

1. In the [Firebase console](https://console.firebase.google.com/), use your existing project
   (e.g. `mcmappainter`) or create one.
2. Enable **Realtime Database**, and two sign-in methods under **Authentication → Sign-in
   method**: **Anonymous** (for guests) and **GitHub**.
   - GitHub requires a **GitHub OAuth App**: GitHub → Settings → Developer settings → OAuth Apps
     → *New OAuth App*. Set the **Authorization callback URL** to the value Firebase shows when
     you enable the GitHub provider (e.g. `https://<project>.firebaseapp.com/__/auth/handler`).
     Copy the **Client ID** + **Client secret** into Firebase's GitHub provider config.
3. Apply the security rules: Realtime Database → **Rules** → paste [`firebase/database.rules.json`](firebase/database.rules.json)
   → Publish. They allow **read** for any signed-in user (incl. guests) and **write** only for
   GitHub users (`sign_in_provider == github.com`), for both `workspaces` and `presence`.
4. Project settings → *Your apps* → **Web app** → copy the config object, and paste it into
   `index.html` where it says:
   ```js
   const FIREBASE_CONFIG = /*FIREBASE_CONFIG*/ null;
   ```
   e.g. `const FIREBASE_CONFIG = { apiKey:"…", authDomain:"…", projectId:"…", databaseURL:"…", appId:"…" };`
   (Make sure `databaseURL` is present — it appears once Realtime Database is enabled.)
5. *(Recommended)* GCP Console → **Billing → Budgets & alerts** → create a small budget (e.g. $1)
   so you're emailed if usage ever leaves the free tier.

That's it. With a config present, the app prompts each visitor to sign in with GitHub or continue
as a guest; GitHub users read/write shared workspaces and see each other's highlighted columns in
real time. A **connection badge** in the header (and an LED in the column control-pad) always
shows the current state (Local only / Connecting / Cloud ready / Live · N online / Cloud
offline) — the fallback to local is never silent.

### Shared image library (in the repo)

So collaborators load the *identical* source image, images are stored in the repo's
[`images/`](images/) folder rather than in the database (keeps Firebase on the free tier).

- **Pick** an existing shared image from the dropdown in the *Image* section.
- **Publish** the current image with *Publish current image to repo…*. This commits it to
  `images/` via the GitHub Contents API. It needs a **fine-grained personal access token**
  with *Contents: Read & write* on this repo, entered under *GitHub upload token* — it's kept
  only in your browser (localStorage) and never committed. Only the person publishing needs a
  token; everyone else just loads the already-committed images (public, no token).
- Workspaces store the image's repo **path**, so opening a shared workspace auto-loads its
  image. Locally-loaded files stay private until you publish them.

**Collaborators can contribute their own images too.** To create a new workspace with their own
image, a collaborator just needs an authenticated GitHub account with write access to this repo:
the owner adds them as a repo **collaborator** (Settings → Collaborators), and they use their own
fine-grained token in the *GitHub upload token* field. Viewing already-published images needs no
token at all.

This is a low-volume, one-time-ish action (a handful of images), so a PAT is the simplest safe
approach without standing up an OAuth backend.

## How it works

`palette.json` lists each Minecraft base map colour as `{id, name, rgb, blocks}`, where `rgb`
is the brightest (×255) shade and `blocks` is the default preference-ordered list of blocks
that produce that colour. The app derives the three staircase shades (×180 ▼ / ×220 ■ / ×255
▲) at load. The palette is embedded directly into `index.html`, so the app needs no network
and no build step. Images are matched to the nearest enabled colour in CIELAB space with
optional error-diffusion dithering. Colour/block RGB data is sourced from the Minecraft Wiki
"Map item format" page (Java 1.21.x).

## Repo layout

| Path | Purpose |
|------|---------|
| `index.html` | The entire app (palette embedded). |
| `palette.json` | Source palette (`{id, name, rgb, blocks}` per colour), for editing / regenerating. |
| `examples/` | Sample image, the original prototype, and the legacy 1.18 palette for reference. |

To regenerate the embedded palette after editing `palette.json`, replace the array following
the `/*COLORS_PLACEHOLDER*/` marker in `index.html` (a compact JSON dump of `palette.json`).

## License

MIT — see [LICENSE](LICENSE).
