# Building Dot Timer

The `.msi` has to be built on Windows and the `.dmg` on macOS. The steps are the same as for World Time.

## Option A: GitHub Actions

There are two separate workflows, so a slow Mac build never holds up the Windows one:

| Workflow | Makes | File |
|---|---|---|
| **Build Windows installer** | `.msi` | `.github/workflows/build.yml` |
| **Build macOS installer** | `.dmg` (Intel + Apple Silicon) | `.github/workflows/build-macos.yml` |

1. Push this folder to its own GitHub repository, for example `dot-timer`.
2. Go to **Actions**, pick a workflow on the left, and click **Run workflow**.
3. Open the run, scroll to the bottom of its Summary page, and download the installer from **Artifacts**.

Pushing a tag such as `v1.0.0` also builds the Windows `.msi` and puts it in a draft GitHub release.

## Option B: build locally

### Windows (makes the .msi)

1. Install the tools once, from Terminal. Skip this if you've already built World Time on this PC:

   ```powershell
   winget install OpenJS.NodeJS.LTS
   winget install Rustlang.Rustup
   winget install Microsoft.VisualStudio.2022.BuildTools --override "--wait --passive --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
   ```

2. Open a terminal in this folder. In File Explorer, type `powershell` in the address bar of the folder that contains `package.json`.
3. Run:

   ```powershell
   npm install
   npm run build:win
   ```

4. The installer is written to `src-tauri\target\release\bundle\msi\Dot Timer_1.0.0_x64_en-US.msi`.

### macOS (makes a universal .dmg)

```bash
rustup target add aarch64-apple-darwin x86_64-apple-darwin
npm install
npm run build:mac
```

The `.dmg` is written to `src-tauri/target/universal-apple-darwin/release/bundle/dmg/`.

For development, `npm run dev` runs the app with live reload. `npm test` runs the logic tests in Node, which cover durations, layout, chimes and World Time matching.

## Project layout

```
src/                 UI (no build step)
  index.html         the widget window
  settings.html      the settings window
  js/timers.js       lengths (Y/M/D/H/M/S), start/pause/reset, time-left read-out
  js/chimes.js       the ten 8/16-bit chimes (Web Audio synth)
  js/dotmatrix.js    5×7 font, World Time's layout, timer rows + loading bar
  js/worldtime.js    reads World Time's settings and measures its widget
  js/time.js         time-zone helpers (needed to measure World Time's rows)
  js/store.js        settings model + bridge to the native shell
src-tauri/           native shell (Rust), config, icons, capabilities
tools/               icon generator, preview builder
tests/               Node tests
```

## Things to know

- **The link to World Time is by identifier.** "Match World Time" finds World Time's settings using its app identifier, `app.worldtime.dotmatrix`. If you change that identifier in World Time's `tauri.conf.json`, also change `WORLD_TIME_ID` at the top of `src-tauri/src/lib.rs` here.
- **Test the chimes in the installed app.** On Windows the app tells WebView2 it may play sound without a click first (`additionalBrowserArgs` in `tauri.conf.json`). On macOS, check that a timer finishing on its own makes a sound.
- **The installers aren't code-signed.** As with World Time, Windows will show a SmartScreen warning and macOS needs **Privacy & Security › Open Anyway**.
- **Replace the placeholder identifier** `app.worldtime.dottimer` in `src-tauri/tauri.conf.json` if you like.
