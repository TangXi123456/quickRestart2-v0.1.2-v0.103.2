# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Slay the Spire 2** mod that adds a "Retry" (重打) button to the pause menu. When clicked, it reloads the game's autosave (`current_run.save`), effectively restarting the current fight. It only works in singleplayer.

**Original Author & Source:** Adapted from **freude916**'s [sts2-quickRestart](https://github.com/freude916/sts2-quickRestart). This version is a local adaptation for Slay the Spire 2, based on the original author's code.

The mod is built as a **C# / .NET 9** DLL that uses **Harmony** runtime patching. It targets the Godot 4.5.1 / MegaDot engine.

## Build Commands

The project auto-detects the OS and Steam library path, then references `sts2.dll` and `0Harmony.dll` from the game installation.

```bash
# Build and auto-deploy to game's mods folder
dotnet build quickRestart2.csproj -c Release

# Build only (Debug config also copies to mods folder via PostBuildEvent)
dotnet build quickRestart2.csproj -c Debug
```

After build, `quickRestart2.dll` and `quickRestart2.json` are copied to `$(Sts2Path)/mods/quickRestart2/`.

### Build Prerequisites & Common Failures

- **Slay the Spire 2 must be installed** via Steam. The `.csproj` reads the registry on Windows to find the Steam library path.
- **`CheckDependencyPaths` target** validates that `Sts2DataDir` and `GodotPath` exist. On Windows, `GodotPath` defaults to `C:/megadot/MegaDot_v4.5.1-stable_mono_win64.exe`. If this path does not exist, the build fails. Temporarily adjust the `<GodotPath>` property in `.csproj` to your actual Godot executable path, or install MegaDot there.
- The `GodotPublish` target exports a `.pck` on `dotnet publish`, but this mod ships with `has_pck: false` and does not require it.

### Release Creation

This mod does not use CI/CD. Releases are created manually:

1. Update `quickRestart2.json` version string
2. `dotnet build quickRestart2.csproj -c Release`
3. Zip the two files from the game's mods folder: `quickRestart2.dll` + `quickRestart2.json`
4. Create a GitHub Release and upload the zip

## Architecture

### Entry Point

The game loads mods via `[ModInitializer(nameof(MethodName))]` on a class. `MainFile.cs` is the entry point:

1. `MainFile.Initialize()` is called by the game's `ModManager`
2. Creates a `Harmony("quickRestart2")` instance
3. `harmony.PatchAll(Assembly.GetExecutingAssembly())` scans and applies all `[HarmonyPatch]` attributes

### Core Components

**`PauseMenuPatch.cs`** — Harmony postfix on `NPauseMenu._Ready`
- Duplicates the existing "Save & Quit" button to create a "Retry" button
- Inserts it before the "Give Up" button
- Connects `Released` signal to `QuickSaveLoad.QuickLoad()`
- Only in singleplayer (`NetGameType.Singleplayer`)
- Disables if no autosave (`SaveManager.Instance.HasRunSave`)
- Rebuilds focus neighbors for controller/keyboard navigation
- **Localization**: ships without a PCK (`has_pck: false`), so `GetRetryLabel()` uses `TranslationServer.GetLocale()` with hardcoded labels per language instead of `LocString`

**`QuickSaveLoad.cs`** — Orchestrates the save/load flow
1. `SaveManager.Instance.LoadRunSave()` reads the autosave
2. Tears down current run: resets action queue, stops music, fades out
3. `RunManager.Instance.SetUpSavedSinglePlayer(runState, serializableRun)` reloads
4. Re-initializes networking and fades back in

**`SaveAndQuitDisablePatch`** — Harmony prefix on `OnSaveAndQuitButtonPressed`
- Disables all pause menu buttons (including Retry) before Save & Quit executes

### Version Compatibility Notes

`SetUpSavedSinglePlayer` changed signature between game versions:
- **older versions** (approx. v0.103-v0.105): synchronous `void SetUpSavedSinglePlayer(RunState, SerializableRun)`
- **v0.104+ onward**: asynchronous `Task SetUpSavedSinglePlayer(...)` (requires `await`)

The current codebase uses the synchronous call. If adapting to a newer game version where this method is async, add `await` before the call.

## Version Adaptation Workflow

When the game updates and the mod needs re-targeting:

1. **Update `quickRestart2.json`** — bump the `version` field (format: `v{mod_version}-v{game_version}-local`)
2. **Update code comments** — any comments referencing the old game version in `QuickSaveLoad.cs`, `PauseMenuPatch.cs`
3. **Update README.md and README_en.md** — version references in the intro sections
4. **Update CLAUDE.md** — version references
5. **Decompile new `sts2.dll`** to verify API signatures:
   ```bash
   ilspycmd "path/to/sts2.dll" -o ./decompiled
   ```
   Check:
   - `RunManager.SetUpSavedSinglePlayer` — sync or async?
   - `NPauseMenu._Ready` — `%ButtonContainer` path still valid?
   - `SaveManager.LoadRunSave()` — return type unchanged?
   - `RunManager.Instance.NetService.Type` — `NetGameType.Singleplayer` enum value unchanged?
6. **Rebuild and test in-game**
7. **Update GitHub repository name** to match the new game version
8. **Create a new Release** with the compiled `dll` + `json`

## Key Files

| File | Purpose |
|------|---------|
| `MainFile.cs` | Mod entry point, Harmony bootstrap |
| `PauseMenuPatch.cs` | UI injection (Retry button) |
| `QuickSaveLoad.cs` | Save/load orchestration |
| `quickRestart2.csproj` | MSBuild project. Defines game path auto-detection, post-build copy to mods folder, and Godot publish target |
| `quickRestart2.json` | Mod manifest read by the game's `ModManager` |
| `quickRestart2/localization/` | JSON localization files (shipped as content, not compiled into DLL) |

## Troubleshooting

- Mod fails to load: check game logs at `%APPDATA%/SlayTheSpire2/logs/`
- Baseline test: launch with `--nomods`
- `CheckDependencyPaths` build error: verify `Sts2DataDir` (game installed) and `GodotPath` (Godot/MegaDot executable exists)
- If `BepInEx.AssemblyPublicizer` causes compile errors with new `sts2.dll` versions, try removing `<Publicize>PublicizeSts</Publicize>` from the `sts2` reference in `.csproj`
