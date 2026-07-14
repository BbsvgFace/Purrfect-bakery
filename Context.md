Project: Purrfect Bakery — a cozy pixel-art idle bakery game that doubles as a Pomodoro focus timer. Baking cats. Stardew-inspired.
Stack: Electron desktop app wrapping a single-file HTML/CSS/JS game. No frameworks, no build step for the game itself — vanilla JS with a <canvas> scene. electron-builder for packaging.
File structure:

index.html — the entire game (all HTML, CSS, and JS inline). This is where ~all game logic lives.
main.js — Electron main process: creates the always-on-top window, handles the pin toggle over IPC, sets the dock/taskbar icon.
preload.js — secure bridge exposing window.desktop.togglePin() / getPin() to the game.
package.json — scripts (start, dist) and electron-builder config (win .exe/nsis, mac .dmg, linux AppImage).
build/icon.ico + build/icon.png — app icons (1024px master PNG, multi-size ICO). Placeholder pixel cat-chef; swappable.
INSTALL.txt / README.md — user + dev docs.
.gitignore — excludes node_modules/, dist/, package-lock.json, .DS_Store.

How the game is architected (all inside index.html):

State: one global S object (coins, day, happiness, fedCount, chocolates, recipes[], rewards[], chocoAnchor, bake, muted, musicOn). Saved to localStorage under key pb2_save. A draft object holds the in-progress bake setup.
Bake engine: uses absolute timestamps (phaseEnd = Date.now() + …), not countdown counters — so it survives app restarts and tab throttling. advanceBake(now) walks through chained phases (focus → break → grace) and can catch up on time elapsed while closed. Phases: focus (work block), break (rest — returning anytime counts as perfect), grace (short window after break ends; return late = keep bake but lose the discipline star; miss it = burned).
Star rating (0–3): time star (focus block ≥25 min), care star (≥2 breaks), discipline star (came back during every break, never in grace). Sell price = baseValue × (0.6 + 0.7 × stars).
Recipes: 20 total (RECIPES array). 3 owned at start, rest unlockable with coins. Each has a shape string that drives a procedural pixel sprite drawn on canvas (paintSprite).
Idle system: while NOT baking, cats make 1 chocolate per 30 min (= 2/hr), worth 2 coins each, capped at 48 per collection. chocoTick computes this from chocoAnchor timestamp; offline earnings shown in a welcome-back modal.
Rewards: buy real-life luxuries (gaming time, meal, movie…) with coins, bank them, claim anytime.
Scene: <canvas> render loop (drawScene) draws the Stardew-style bakery — animated cats that roam/mix/bake/frost depending on bake phase, oven, stations, string lights, steam/sprinkle particles, a live timer plaque.
Music: fully procedural chiptune via Web Audio API (startMusic/stopMusic) — upbeat major C→F→G→C loop at 100bpm, kick + soft tap + bell chirps, no audio files. Toggle via 🎵 button. Also small SFX (beep, sfxCoin, etc.).
UI: 5 screens (home/setup/recipes/rewards/cats) swapped via bottom nav; each has its own render*() function. Fonts: Silkscreen (headers) + VT323 (body).

Pin feature: main.js opens the window alwaysOnTop: true at 'floating' level; the in-game 📌 button calls setAlwaysOnTop through IPC.
Dev workflow: edit index.html → npm start to test → npm run dist to build installer → bump version in package.json each release. appId (com.purrfect.bakery) must stay constant so saves persist across updates.

Known gotchas:
Windows npm run dist may need Developer Mode or admin (symlink perms in electron-builder's winCodeSign cache).
Unsigned app → SmartScreen (Windows) / Gatekeeper (Mac) warnings on first run; normal.
Web Audio needs a user gesture before it plays — first click anywhere unlocks it.
