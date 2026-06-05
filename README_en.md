> **Original Author & Source**
> This mod is adapted from **freude916**'s [sts2-quickRestart](https://github.com/freude916/sts2-quickRestart).
> Original repository: <https://github.com/freude916/sts2-quickRestart>
> This version is a local adaptation for **Slay the Spire 2 v0.107.0**, based on the original author's code.

[English](./README_en.md) / [中文](./README.md)

*This mod has quite a few stars, so I'll leave some modding guides here.*

---

# 🎮 Player Guide

## 1. Mod Installation & Usage

Slay the Spire 2 officially supports modding with extensive hooks and Harmony integration.

**Installation:**
1. Create a `mods` folder in the game's root directory
2. Drop the `.pck` or `.dll` files into it. Each mod must have a mod_manifest.json to describe it, while these two files are optional
3. Launch the game

mod_manifest.json example: (Alchyr ModTemplate)

```json
{
  "id": "ModTemplate",
  "name": "ModTemplate",
  "author": "{ModAuthor}",
  "description": "Slay the Spire 2 mod created from a template for use with BaseLib",
  "version": "v0.0.0",
  "has_pck": true,
  "has_dll": true,
  "dependencies": ["BaseLib"],
  "affects_gameplay": true
}
```

**Note:**
If your mods fail to load (especially for Linux and MacOS users), install [BaseLib](https://www.github.com/Alchyr/BaseLib) to resolve compiler differences.

MacOS seems to have another loading issue due to arm, but it appears to have been fixed in 0.99.

**Disabling Mod Mode:**
You can toggle each mod on/off in the main menu settings, or add `--nomods` to Steam launch options to disable all mods. If you're unsure whether mods are causing launch issues, it's recommended to use `--nomods` first to troubleshoot.

**Multiplayer:**
Slay the Spire 2 requires all players in multiplayer to have the exact same game version, and all must load multiplayer mods with matching version numbers.

Some platforms have delayed hot updates. You can ask your friend to copy the release_info.json from their installation directory (Steam -> Slay the Spire 2 -> Right-click on the game in the list/Settings icon to the right of the launch button -> Manage -> Browse local files) to yours,
which will trick the game into thinking your version is exactly the same as theirs. Security not guaranteed.

The mod_manifest.json specifies whether a mod affects gameplay content via `affect_gameplay`, i.e. whether the mod needs to be installed by all players. Mods with affect_gameplay must be installed by all multiplayer participants, otherwise besides the "we have mods that the host doesn't have" warning, you may also get "connection timeout" errors.

Mod IDs and version numbers must match exactly. Make sure your friends also have the same mods installed.

Additionally, Slay the Spire 2's application-layer network architecture is: Client sends requests to Host, then Host broadcasts. Client receives broadcasted requests once, but Host's own requests are only broadcast once without replay. If your mod behaves differently on client vs host, please investigate this.

## 2. Save System & Data Recovery

⚠️ Enabling mods will trigger a Warning prompt. Confirming will switch the game to Modded mode with a **separate save system**. Here's how to restore your saves:

Press `` ` `` (the key left of the number 1, above Tab) to open the console, then type `open saves` to quickly open the save directory. Navigate up a few folders until you see `profile1`, `2`, `3` and `modded` folders.

Copy `profile1` into `modded`, overwriting the existing folder, to restore your saves.

Alternatively, you can use Steam Cloud sync:

`[Steam Directory]/userdata/[SteamID]/2868840/` (works for Windows / Linux / macOS) contains another save directory.

Since this is a Steam Cloud sync directory, when you return to Steam, it will prompt **"Cloud Conflict"**. Choose **"Keep Local Files"**.

If Steam Cloud or Slay the Spire 2 persistently overwrites your saves, manually copy to these paths:
* **Windows**: `%APPDATA%/SlayTheSpire2/steam/[SteamID]/2868840/`
* **Linux**:  `~/.local/share/SlayTheSpire2/steam/[SteamID]/2868840/`
* **MacOS**: `~/Library/Application Support/SlayTheSpire2/steam/<SteamID>/`
  *(Note: `~/` is your home directory, i.e. `/home/[yourname]` or `/Users/[yourname]` )*
  *(Basically, all three platforms use `%APPDATA%` or `$XDG_DATA_HOME` or `Application Support` as the user data directory, then add `SlayTheSpire2/steam/[SteamID]/2868840/` )*

## 3. DevConsole - Enabling & Usage

**How to Open**: Press `` ` `` (backtick) key. It's located left of the number 1 key, above Tab.

### Manually Enable Full Console
Without mods enabled, the console only supports basic debug commands. You can load a simple mod to enable full console functionality, or manually enable it **without mods**:
1. Open the save directory as described above
2. Navigate up two levels, find the `settings.save` file
3. Open it in a text editor and add `"full_console": true,` at the beginning:
   ```json
   {
     "full_console": true,
     "aspect_ratio": "auto"
   }
   ```
4. Save the file and restart the game

### Console Shortcuts
* **Tab**: Auto-complete commands or parameters
* **↑ / ↓**: Browse command history
* **F11**: Toggle fullscreen/half-screen display
* **Ctrl + L**: Clear screen
* **Ctrl + A / E**: Move cursor to beginning/end of line
* **Ctrl + U / K**: Delete entire line / Delete to end of line
* **Ctrl + W**: Delete word before cursor
* **Escape / Ctrl + C**: Close or hide console
* Yes, these are Emacs-style shortcuts. You'll get used to them.

### Console Commands List
Commands marked with ✅ support syncing to other players in multiplayer mode. Commands marked with ❌ only work locally.

<details>
<summary>Click to expand: Resources & Values</summary>

| Command           | Argument   | Description         | MP |
|-------------------|------------|---------------------|----|
| `gold`            | `<amount>` | Add or remove gold  | ✅  |
| `stars`           | `<amount>` | Add or remove stars | ✅  |
| `heal` / `damage` | `<value>`  | Heal / Take damage  | ✅  |
| `energy`          | `<value>`  | Gain energy         | ✅  |
| `block`           | `<value>`  | Gain block          | ✅  |
</details>

<details>
<summary>Click to expand: Cards, Relics & Items</summary>

| Command   | Argument            | Description                                     | MP |
|-----------|---------------------|-------------------------------------------------|----|
| `card`    | `<ID> [pile]`       | Generate card to specified pile (default: hand) | ✅  |
| `upgrade` | `<cardID>`          | Upgrade specified card                          | ✅  |
| `remove`  | `<cardID>`          | Remove card from deck                           | ✅  |
| `enchant` | `<cardID>`          | Apply enchantment to card                       | ✅  |
| `relic`   | `[add/remove] <ID>` | Add or remove specified relic                   | ✅  |
| `potion`  | `<potionID>`        | Gain specified potion                           | ✅  |
| `draw`    | `<amount>`          | Draw specified number of cards                  | ✅  |
</details>

<details>
<summary>Click to expand: Combat & Progress Control</summary>

| Command       | Argument        | Description                    | MP |
|---------------|-----------------|--------------------------------|----|
| `fight`       | `<encounterID>` | Jump to specific enemy fight   | ✅  |
| `win` / `die` | -               | Instantly win / die            | ✅  |
| `kill`        | `<target>`      | Kill specified target on field | ✅  |
| `act`         | `<Act-ID>`      | Jump to specified Act          | ✅  |
| `room`        | `<type>`        | Enter specific room type       | ✅  |
| `event`       | `<eventID>`     | Force trigger specific event   | ✅  |
| `god`         | `[on/off]`      | Toggle god mode                | ❌  |
| `instant`     | `[on/off]`      | Toggle instant action mode     | ❌  |
</details>

<details>
<summary>Click to expand: System & Debug</summary>

| Command          | Description                        | MP |
|------------------|------------------------------------|----|
| `help [cmd]`     | Show detailed help for command     | ❌  |
| `clear` / `exit` | Clear screen / Close console       | ❌  |
| `unlock <ID>`    | Unlock specified game content      | ✅  |
| `achievement`    | Unlock specified Steam/achievement | ❌  |
| `log [level]`    | Set log level                      | ❌  |
| `dump`           | Export current debug info snapshot | ❌  |
| `multiplayer`    | Enter multiplayer debug tools      | ❌  |
| `trailer`        | Enable trailer demo mode           | ❌  |
</details>

## 4. Launch Options

Use these in Steam launch options or command line:

| Parameter                 | Description                               |
|---------------------------|-------------------------------------------|
| `--force-steam[=on\|off]` | Force enable or disable Steam integration |
| `--autoslay`              | **(Debug)** Auto-start new game on launch |
| `--seed <seed>`           | Specify game seed                         |
| `--log-file <path>`       | Custom log file path                      |
| `--bootstrap`             | Launch in bootstrap mode                  |
| `--fastmp`                | **Fast Multiplayer**                      |
| `--clientId <id>`         | Specify client ID                         |
| `--nomods`                | Disable all mod loading                   |
| `+connect_lobby <id>`     | Auto-join specified Steam lobby on launch |

If mods cause your game to fail launching, use `--nomods` to disable mod loading and restore the game to vanilla state.

`--force-steam=off` + `--fastmp` + `--cliendID` can let you quickly test multiplayer-related mods without needing to find other accounts every time. Usage is as follows:

--fastmp [type]: `host` `host_standard` `host_daily` `host_custom` `load` `join`
--clientId [id]: Specify client NetGameService ID. For fastmp, host ID must be 1, clients must be 1000 ~ 1002, otherwise "multiplayer save/load" won't work.

Or, if you prefer, you can use [gbe_fork](https://github.com/Detanup01/gbe_fork) to simulate the Steam API.

If you want to assist mod authors with debugging, find the logs folder in the local save directory mentioned above, and send the log files with corresponding timestamps to the author. Or use the --log-file parameter to specify a log file path.


## 5. Get More Mods & Community

Many mods are already available or in development:
* **[Slay the Spire 2 Discord](https://discord.gg/slaythespire)**: Official English community with dedicated Modding channel. [This post](https://discord.com/channels/309399445785673728/1480486428843446383) contains early mod lists and links.
* **[Chinese Mod Spreadsheet (Tencent Docs)](https://docs.qq.com/sheet/DZnNQTnBFdXVJeGpH?tab=BB08J2)**: Chinese community mod tracker (in-progress and released). I periodically sync with Discord.

* Deleted. Tencent QQ need Chinese phone number to register.

---

# 🛠️ Developer & Modding Guide

## 1. Tutorials & Learning Resources

* **[Glitched Reme's StS2 Modding Tutorials](https://github.com/GlitchedReme/SlayTheSpire2ModdingTutorials)** (Chinese, continuously updated)
* **[EarlyStS2ModdingGuides](https://github.com/Cany0udance/EarlyStS2ModdingGuides/wiki/Getting-Started-With-Modding)**: Mod development beginner's guide (English).

**Language & Engine Basics:**
If you're using modding as an opportunity to learn C#, systematic learning is recommended:
* **C# Language**: [C# Learning (En)](https://learn.microsoft.com/en-us/training/paths/csharp-first-steps/) / [C# Learning (中)](https://learn.microsoft.com/zh-cn/training/paths/csharp-first-steps/)
* **Godot Engine**: [Godot (En)](https://docs.godotengine.org/en-us/4.x/getting_started/step_by_step/index.html) / [Godot (中)](https://docs.godotengine.org/zh-cn/4.x/getting_started/step_by_step/index.html)
  *(Note: Microsoft and Godot official Chinese translations may be incomplete. Recommend reading English original with LLM/translation plugin)*

If you plan on long-term mod development, you'll work closely with these tools:
* **[Harmony Lib](https://harmony.pardeike.net/articles/intro.html)**: Harmony Patch, built into StS2. Essential reading.
* **[MonoMod](https://github.com/MonoMod/MonoMod)**
* **[BepInEx](https://github.com/BepInEx/BepInEx)**: Popular C# mod framework that injects mods via winhttp.dll.

(These three are related: Harmony handles patching, MonoMod handles code modification/rewriting, BepInEx handles injection when the game doesn't natively load mods.)

## 2. Templates & Libraries

Recommended: Alchyr's templates for quick setup:
* **[ModTemplate-StS2](https://github.com/Alchyr/ModTemplate-StS2)**: Ready-to-use StS2 C# mod project template with build scripts and standard folder structure. Clone and customize.
* **[BaseLib-StS2](https://github.com/Alchyr/BaseLib-StS2)**: BaseMod equivalent for StS2 (work in progress).

## 3. Tools & Decompilation

**Code Decompilation:** StS2 has no obfuscation. Use [ilSpy](https://www.github.com/icsharpcode/ILSpy) or [dnSpyEx](https://github.com/dnSpyEx/dnSpy) to decompile game source code.

**Asset Unpacking:** Use [GD RE Tools' gdsdecomp](https://github.com/GDRETools/gdsdecomp/) to unpack game resources.

**Asset Packing:**
* You need to pack resources into `.pck` files for the game to recognize them.
* StS2 uses a modified Godot 4.5.1 called [MegaDot](https://megadot.megacrit.com/). You can also use standard [Godot 4.5.1](https://godotengine.org/download/archive/) release version. *(Newer Godot versions produce pck files that will be rejected by the game)*.
* *Spine Note*: Godot uses [Spine Runtime for Godot](https://esotericsoftware.com/spine-godot) GDExtension to pack Spine assets. You need to install it in your Godot project. (GDExtensions are project-level, not global, so you need to install it in each project, i'm not sure if any better way exists)
* **No Art Assets**: If you don't need any art assets, use [PCK Explorer](https://github.com/DmitriySalnikov/GodotPCKExplorer) to create a minimal pck containing only `mod_manifest.json`. Set command line version parameter to `3.4.5.1`.

## 4. Migrating from Slay the Spire 1

* **Engine Change**: StS1 is Java-based; StS2 migrated to C# (Godot). Mod scripts need complete rewriting.
* **Spine Animation Change**: StS1 uses Spine 3.4 with JSON skeletons; StS2 uses Spine 4.2 with binary (`.skel`) skeletons.
* **Conversion Tool**: Recommend [SpineSkeletonDataConverter](https://github.com/wang606/SpineSkeletonDataConverter) for binary migration.
  (However, this tool only supports Spine 3.5 minimum. You need to force-change the version string in StS1's JSON to 3.5 for it to work. Compatibility not guaranteed.)
* **Important Notes**: Animation key names and scale ratios have changed. Testing shows StS1 "Defect" scales approximately 10x after migration (applied to root bone), but different mod characters may vary.
* **Animations**: StS1 skeleton files only have idle and hit animation tracks; most attack animations are separate; death uses static portrait. StS2 has 6 built-in tracks.

## 5. Developer Community Groups

Discord.

* Deleted. Tencent QQ need Chinese phone number to register.

## 6. Other

I occasionally distribute test builds of mods. You might get lucky in various groups.
If you're interested in our work, check out the GitHub organization **[ModinMobileSTS](https://github.com/ModinMobileSTS)** for StS1/2 mobile mod info (some content may be private due to copyright reasons).