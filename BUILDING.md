# Building Dot Timer

The `.msi` has to be built on Windows and the `.dmg` on macOS. The steps are the same as for World Time.

## build locally

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

4. The installer is written to `src-tauri\target\release\bundle\msi\Dot Timer_1.1.0_x64_en-US.msi`.

### macOS (makes a universal .dmg)
 
1. Install the tools once, from Terminal. Skip any you already have.
 
   Xcode command line tools:
   ```
   xcode-select --install
   ```

   Rust. If you previously installed Rust through Homebrew, remove it first, because its older Cargo can't build this project and it conflicts with rustup:
   ```
   brew uninstall rust
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
   Choose the standard installation, then load Rust into your current shell (or just open a new Terminal window):
   ```
   . "$HOME/.cargo/env"
   ```
 
   Node.js (with [Homebrew](https://brew.sh)):
   ```
   brew install node
   ```
   Or download the LTS installer from https://nodejs.org and open a new Terminal window afterwards.
 
   Check everything is in place. Each command should print a version, and `cargo` should be 1.85 or newer:
   ```
   node -v
   npm -v
   cargo --version
   rustup --version
   ```
 
2. Open Terminal in the project folder, the one that contains `package.json`. Type `cd ` (with a trailing space), drag the folder from Finder into the Terminal window and press Enter. For example:
   ```
   cd /Users/yourname/Downloads/dot-[widghetname]-main
   ```
 
3. Add the two Mac build targets and install the dependencies:
   ```
   rustup target add aarch64-apple-darwin x86_64-apple-darwin
   npm install
   ```
 
4. Build. The first run compiles a lot of Rust and can take several minutes:
   ```
   CI=true npm run build:mac
   ```
   `CI=true` skips the step where the DMG script uses AppleScript to arrange the window in Finder. That step can hang or fail depending on Automation permissions. The only effect is that the DMG window has no custom icon layout.
 
5. Open the output folder:
   ```
   open src-tauri/target/universal-apple-darwin/release/bundle/dmg/
   ```
   The installer is `Dot WIDGETNAME_X.X.X_universal.dmg`. If you only see `bundle_dmg.sh` and `icon.icns`, the build didn't finish; see Troubleshooting below.
 
### Troubleshooting (macOS)
 
- **`rustup: command not found`**: run `. "$HOME/.cargo/env"` or open a new Terminal window.
- **`npm: command not found`**: Node.js isn't installed; see step 1.
- **`feature edition2024 is required` / Cargo 1.84**: an old Homebrew Rust is being used instead of rustup's. Run `brew uninstall rust`, open a new Terminal window and check that `cargo --version` is 1.85 or newer.
- **No `.dmg` in the folder, and a "Dot WIDGET" disk is mounted**: a previous build was interrupted. Eject it with `hdiutil detach "/Volumes/Dot WIDGET"` (add `-force` if it's busy), then rebuild with `CI=true npm run build:mac`.
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
