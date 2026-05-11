# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Slay the Spire 2** mod that adds a "Retry" (重打) button to the pause menu. When clicked, it reloads the game's autosave (`current_run.save`), effectively restarting the current fight. It only works in singleplayer.

**Original Author & Source:** This mod is adapted from **freude916**'s [sts2-quickRestart](https://github.com/freude916/sts2-quickRestart). This version is a local adaptation for Slay the Spire 2 v0.105.1, based on the original author's code.

The mod is built as a **C# / .NET 9** DLL that uses **Harmony** runtime patching to inject into the game's code. It targets the Godot 4.5.1 / MegaDot engine.

## Build Commands

The project auto-detects the OS and Steam library path, then references the game's `sts2.dll` and `0Harmony.dll` directly from the game installation.

```bash
# Build and auto-deploy to game's mods folder
dotnet build quickRestart2.csproj -c Release

# Build only (no copy to mods folder; the PostBuildEvent does the copy)
dotnet build quickRestart2.csproj -c Debug
```

After a successful build, the following are copied to `$(Sts2Path)/mods/quickRestart2/`:
- `quickRestart2.dll` — the compiled mod
- `quickRestart2.json` — the mod manifest

The `.csproj` also has a `GodotPublish` target that exports a `.pck` file if `GodotPath` is configured, but this is optional and currently skipped in this adaptation.

## Architecture

### Entry Point: `[ModInitializer]` System

The game loads mods via a `[ModInitializer(nameof(MethodName))]` attribute placed on a class. `MainFile.cs` serves as the entry point:

1. `MainFile.Initialize()` is called by the game's `ModManager`
2. It creates a `Harmony(ModId)` instance
3. Calls `harmony.PatchAll(Assembly.GetExecutingAssembly())` to scan and apply all `[HarmonyPatch]` attributes

### Core Components

**`PauseMenuPatch.cs`** — Harmony postfix on `NPauseMenu._Ready`
- Duplicates the existing "Save & Quit" button to create a "Retry" button
- Inserts it before the "Give Up" button in the button container
- Connects the button's `Released` signal to `QuickSaveLoad.QuickLoad()`
- Only activates in singleplayer (`NetGameType.Singleplayer`)
- Disables the button if no autosave exists (`SaveManager.Instance.HasRunSave`)
- Rebuilds focus neighbors for controller/keyboard navigation

**`QuickSaveLoad.cs`** — Orchestrates the save/load flow
1. Reads `SaveManager.Instance.LoadRunSave()` to get the autosave
2. Tears down the current run: resets action queue, stops music, fades out
3. Calls `RunManager.Instance.SetUpSavedSinglePlayer(runState, serializableRun)` to reload
4. Re-initializes networking and fades back in

**Version compatibility note:** `SetUpSavedSinglePlayer` changed signature between game versions:
- **v0.105.1**: synchronous `void SetUpSavedSinglePlayer(RunState, SerializableRun)`
- **v0.104+**: asynchronous `Task SetUpSavedSinglePlayer(...)` (requires `await`)

The current codebase targets **v0.105.1** and uses the synchronous call.

### Localization Strategy

This adaptation ships **without a PCK file** (`has_pck: false` in the manifest). The game loads mod localization from `res://{mod_id}/localization/{lang}/` paths, which only exist when packaged into a PCK. Since no PCK is built, `PauseMenuPatch.GetRetryLabel()` reads `TranslationServer.GetLocale()` and returns a hardcoded label per language instead of using `LocString`.

### Project Structure

```
├── MainFile.cs              # Mod entry point, Harmony bootstrap
├── PauseMenuPatch.cs        # UI injection (Retry button)
├── QuickSaveLoad.cs         # Save/load orchestration
├── quickRestart2.csproj     # MSBuild project (targets net9.0)
├── quickRestart2.json       # Mod manifest (id, version, has_pck, etc.)
├── project.godot            # Godot project file
├── decompiled/              # Reference: decompiled game source (excluded from compilation)
│   └── RunManager.cs, NPauseMenu.cs, SaveManager.cs, etc.
└── quickRestart2/localization/  # JSON localization files (not compiled into DLL)
```

### Key Dependencies

- **0Harmony.dll** — The Harmony patching library (bundled with the game)
- **sts2.dll** — The game's main assembly (publicized via `BepInEx.AssemblyPublicizer.MSBuild`)
- **Godot.NET.Sdk/4.5.1** — Godot C# SDK

### Decompiled Reference Code

The `decompiled/` folder contains decompiled game source code used as a reference for API signatures. It is excluded from compilation via `<Compile Remove="decompiled\**"/>`. Do not edit these files — they are read-only references. If the game updates, decompile the new version with `ilspycmd` and replace the files.

## Mod Manifest

`quickRestart2.json` describes the mod to the game's `ModManager`:

```json
{
  "id": "quickRestart2",
  "version": "v0.1.2-v0.105.1-local",
  "has_pck": false,
  "has_dll": true,
  "affects_gameplay": false
}
```

- `affects_gameplay: false` means multiplayer clients do not need to install this mod
- `has_pck: false` means no Godot resource pack is shipped (no art assets, no LocString-based localization)

## Adapting to New Game Versions

When the game updates, verify these API signatures by decompiling the new `sts2.dll`:

1. `RunManager.SetUpSavedSinglePlayer` — check if it's sync or async
2. `NPauseMenu._Ready` — ensure the node path `%ButtonContainer` still exists
3. `SaveManager.LoadRunSave()` — verify return type `ReadSaveResult<SerializableRun>`
4. `NGame.Instance.Transition.FadeOut()` — check default parameter values
5. `RunManager.Instance.NetService.Type` — verify `NetGameType.Singleplayer` enum value

Use `ilspycmd` to decompile:
```bash
ilspycmd "path/to/sts2.dll" -o ./decompiled
```

## Troubleshooting Mod Loading

- If the mod fails to load, check game logs at `%APPDATA%/SlayTheSpire2/logs/`
- Launch with `--nomods` to disable all mods for baseline testing
- Ensure `quickRestart2.json` and `quickRestart2.dll` are in `Slay the Spire 2/mods/quickRestart2/`
- The `CheckDependencyPaths` build target errors if the game data directory or Godot path is missing
