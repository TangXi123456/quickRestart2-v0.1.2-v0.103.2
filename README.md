[English](./README_en.md) / [中文](./README.md)

# QuickRestart2 — 快速重打 Mod for 杀戮尖塔2

## 简介

在暂停菜单中新增一个**「重打」**按钮。点击后读取游戏自动存档（`current_run.save`），快速重开本场战斗或本个事件。仅在单人模式下生效。注意该mod在原作者的代码仓库下由ClaudeCode适配的新版本不保证会出现存档崩坏等bug，目前游戏频繁更新mod适配可能不及时。

> **原作者与来源**
> 本 Mod 改编自 **freude916** 的 [sts2-quickRestart](https://github.com/freude916/sts2-quickRestart)。
> 原作者仓库：<https://github.com/freude916/sts2-quickRestart>
> 当前版本为基于原作者代码适配 **Slay the Spire 2 v0.107.0** 的本地修改版。



# 🎮 玩家指南

## Mod 安装与使用

尖塔2官方已支持 Modding，并提供了大量钩子和 Harmony 支持。实际上也有点屎山。

**安装：**
1. 在游戏目录下创建 `mods` 文件夹
2. 将 `.pck` 或 `.dll` 文件放入该文件夹。每个 mod 必有一个 .json 来描述，而这两个文件是可选的。
3. 启动游戏。
4. 开启 Mod 后游戏内会弹出一次 Warning，确认会导致游戏切换为使用 Modded 模式，**并切换到另一套 MODDED 存档系统**。如果您需要恢复，请见后一节。

mod_manifest.json 例如下：(来自 Alchyr ModTemplate)

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

**平台相关**
如果您的 Mod 加载有问题（尤其是 Linux 和 MacOS 用户），可尝试安装 [BaseLib](https://www.github.com/Alchyr/BaseLib) 。

Linux 用户也可以在游戏启动选项前添加 `LD_PRELOAD=/usr/lib/libgcc_s.so.1`。

MacOS 因为 arm 似乎有另一个加载问题，但是在 0.99 中似乎已经得到修复。

> [!IMPORTANT]
> 由于 Mod 社区的疏忽， 2026年3月15日之前 ModTemplate 误将 pck 构建目标(`binary_format/architecture`)设为了 `x86_64` 而非 `msil` ，而 Godot 下 pck 元数据会影响 dll 导致无法加载，这严重破坏了 arm 兼容性，包括 M 系列 mac 和所有手持终端。
> 如果您在日志中发现  `xxx.dll The assembly architecture is not compatible with the current process atchitecture` 问题，这不是 dll 的问题 ( C# msil dll 是跨平台的 ) 而是 pck 的问题 ，您可能需要手工用 pck 工具拆解 pck 后重打包 Mod，可用工具见后。

**关 Mod**
您可以在主菜单的设置中切换每个 mod 的开关，或者在 Steam 启动选项中添加 `--nomods` 关闭所有 Mod。如果您不确定是否是 Mod 导致了游戏启动问题，建议先使用 `--nomods` 来排查。

> [!IMPORTANT]
> 请注意，如果您在游玩内容型 Mod （尤其是角色 Mod） 中出现任何问题，请不要关闭当前存档的角色 Mod ！
> 在主界面“继续当前游戏”中，游戏会尝试读取角色信息，然后角色 Mod 的缺失将直接引起闪退！
> 如果您的游戏启动时闪退，此时你必须关掉 Steam 云同步，然后按照下面的途径删除云缓存和本地两处存档的当前局内。

**联机**
杀戮尖塔 2 强制所有联机的玩家都必须使用完全相同的游戏版本，并都加载了具有相同版本号的联机 mod。

部分平台的热更新似乎是滞后的。让朋友拷贝安装目录（Steam -> 塔2 -> 列表右键/启动按钮右边的右边的设置图标 -> 管理 -> 浏览本地文件） release_info.json 给您，
就可以欺骗游戏认为您的版本和他们完全一样了。不保证安全性。

mod_manifest.json 里规定了 mod 是否会影响游戏内容 `affects_gameplay`，即 mod 是否需要多人都装。 affect_gameplay 的 mod 联机者必须都装，否则除了“我们有而房主没有的mod”这一提示之外也很有可能提示“联机超时”。

affects_gameplay 的 Mod ID 、版本号、**顺序**必须完全匹配。请让您的朋友们也安装相同的 Mod ，然后在联机之前仅保留一个 Mod 打开一次游戏清除 Mod 顺序配置，再把所有需要的 Mod 放进去让游戏发现，否则顺序问题一直得不到改善。您也可以下载 Jianbao233 的顺序调整器，方便一点。

另外，杀戮尖塔 2 的应用层网络结构是：客户端 Client 向 房主 Host 发送请求，然后 Host 再广播给所有 Client（不包括 Host 自己）。 Client 总会被再广播一次自己的请求，但是 Host 的请求只会广播 1 次而不会发送到自身。如果您的 Mod 在客户端和房主表现不同，请排查这里。

## 设置和存档

游戏有两份存档：

Steam 云同步缓存： `[Steam 目录]/userdata/[SteamID]/2868840/` (Windows / Linux / macOS 均适用) 。

本地存储：
* 见下打开控制台，输入 `open saves` 可以快速打开这一目录。
* 如果游戏不能启动，请将下面路径复制到您资源管理器的地址栏。
* **Windows**: `%APPDATA%/SlayTheSpire2/`
* **Linux**:  `~/.local/share/SlayTheSpire2/`
* **MacOS**: `~/Library/Application Support/SlayTheSpire2/`
* (这其实是 Godot 的 `user://` 自动路径，是 %APPDATA% 或 $XDG_DATA_HOME 或 Application Support 这种用户数据目录然后加上 project ID ，此处即 SlayTheSpire2。)

在这个目录下，最重要的是两个：

- 存档目录， `~~~/steam/[SteamID]/`，`open saves` 同样可以打开。
- 日志目录， `~~~/logs` ，里面存储所有的日志，`open logs` 同样可以打开。

如果您想要协助 Mod 作者调试，将日志文件按照对应时间戳发给作者。或者使用 --log-file 参数指定一个日志文件路径方便快速复制。

在存档目录下，最重要的是 profile1 profile2 profile3 modded 四个文件夹，其中 modded 即 mod 模式的存档槽，里面同样有profile1 profile2 profile3。

只要将 profile 复制进 modded 就能同步您的进度。

每个 profile 下有 saves 文件夹和 replay 文件夹， replay 可以用于重播战斗但是比较复杂，在此按下不提。

saves文件夹可能有下列文件：

- history 文件夹，是主菜单展示的游玩历史，想要什么自己打开看，在此不赘述了。
- progress.save 保存进阶、时间线和卡牌的解锁历史。
- prefs.save 保存的是设置。
- current_run.save 保存的是当前局的数据，游戏启动时会尝试读取并显示“继续当前游戏”，死档主要是删这个。


目前推测是游戏启动前 Steam 先将云缓存和云上对比，游戏启动后游戏会将 Steam 云缓存复制到本地目录；关闭后逆序。

如果您死档了，您可能需要关闭 Steam 云同步，再删除两处的 current_run.save 。

## 控制台 (DevConsole) 开启与使用

**打开方式**：按 `` ` `` (反引号) 键。这个键在横排数字 1 的左边，Tab的上面。

### 手动开启完整控制台
在未开启 Mod 的情况下，控制台默认只支持一些基本的调试命令。您可以加载一些简单的 Mod （例如本 mod) 来启用完整的控制台功能，或者在**不开启 Mod** 的情况下手动启用：
1. 在存档目录里往上几层，搜索 `settings.save` 文件
2. 用文本编辑器打开，在开头添加 `"full_console": true,`：
   ```json
   {
     "full_console": true,
     "aspect_ratio": "auto"
   }
   ```
3. 保存文件并重启游戏

### 控制台快捷键
* **Tab**: 自动补全命令或参数
* **↑ / ↓**: 快速浏览历史命令
* **F11**: 切换全屏/半屏显示
* **Ctrl + L**: 清除屏幕信息
* **Ctrl + A / E**: 光标移至行首/行尾
* **Ctrl + U / K**: 删除整行 / 删除至行尾
* **Ctrl + W**: 删除光标前的一个词
* **Escape / Ctrl + C**: 关闭或隐藏控制台
* 是的这是 Emacs 风格的快捷键，习惯了就好。

### 控制台命令列表
带 ✅ 标记的命令支持在多人联网模式下同步状态到其他玩家，带 ❌ 标记的命令则仅在本地生效

<details>
<summary>点击展开：资源与数值修改</summary>

| 命令                | 参数     | 描述           | 联网 |
|-------------------|--------|--------------|----|
| `gold`            | `<数量>` | 增加或减少金币      | ✅  |
| `stars`           | `<数量>` | 增加或减少星星      | ✅  |
| `heal` / `damage` | `<数值>` | 恢复生命值 / 受到伤害 | ✅  |
| `energy`          | `<数值>` | 获得能量         | ✅  |
| `block`           | `<数值>` | 获得格挡值        | ✅  |
</details>

<details>
<summary>点击展开：卡牌、遗物与物品</summary>

| 命令        | 参数                  | 描述              | 联网 |
|-----------|---------------------|-----------------|----|
| `card`    | `<ID> [牌堆]`         | 生成卡牌到指定位置（默认手牌） | ✅  |
| `upgrade` | `<卡牌ID>`            | 升级指定的卡牌         | ✅  |
| `remove`  | `<卡牌ID>`            | 从牌组中移除卡牌        | ✅  |
| `enchant` | `<卡牌ID>`            | 为卡牌施加附魔         | ✅  |
| `relic`   | `[add/remove] <ID>` | 添加或移除指定的遗物      | ✅  |
| `potion`  | `<药水ID>`            | 获得指定的药水         | ✅  |
| `draw`    | `<数量>`              | 立即抽取指定数量的牌      | ✅  |
</details>

<details>
<summary>点击展开：战斗与进程控制</summary>

| 命令            | 参数         | 描述            | 联网 |
|---------------|------------|---------------|----|
| `fight`       | `<战斗ID>`   | 跳转到特定的敌人战斗    | ✅  |
| `win` / `die` | -          | 立即获得胜利 / 立即死亡 | ✅  |
| `kill`        | `<目标>`     | 击杀场上指定目标      | ✅  |
| `act`         | `<Act-ID>` | 跳转到指定的章节（Act） | ✅  |
| `room`        | `<类型>`     | 进入特定类型的房间     | ✅  |
| `event`       | `<事件ID>`   | 强制触发特定事件      | ✅  |
| `god`         | `[on/off]` | 开启/关闭无敌模式     | ❌  |
| `instant`     | `[on/off]` | 开启/关闭即时动作模式   | ❌  |
</details>

<details>
<summary>点击展开：系统与调试</summary>

| 命令               | 描述               | 联网 |
|------------------|------------------|----|
| `help [cmd]`     | 显示命令的详细帮助信息      | ❌  |
| `clear` / `exit` | 清除屏幕 / 关闭控制台     | ❌  |
| `unlock <ID>`    | 解锁指定的游戏内容        | ✅  |
| `achievement`    | 解锁指定的 Steam/游戏成就 | ❌  |
| `log [level]`    | 设置日志记录级别         | ❌  |
| `dump`           | 导出当前的调试信息快照      | ❌  |
| `multiplayer`    | 进入多人游戏调试工具       | ❌  |
| `trailer`        | 开启适合拍摄预告片的演示模式   | ❌  |
</details>

## 启动参数

在 Steam 启动选项或命令行中使用：

| 参数                        | 用途说明                 |
|---------------------------|----------------------|
| `--force-steam[=on\|off]` | 强制启用或禁用 Steam 集成     |
| `--autoslay`              | **（调试用）** 启动时自动开始新游戏 |
| `--seed <seed>`           | 指定特定的游戏种子（Seed）      |
| `--log-file <path>`       | 自定义日志文件的保存路径         |
| `--bootstrap`             | 以引导模式启动游戏            |
| `--fastmp`                | **快速多人模式**           |
| `--clientId <id>`         | 指定客户端 ID             |
| `--nomods`                | 禁用所有模组加载，以原生状态启动     |
| `+connect_lobby <id>`     | 启动后自动加入指定的 Steam 大厅  |

如果 Mod 使您的游戏无法正常启动，您可以使用 `--nomods` 参数来禁用 Mod 加载，恢复到原生状态。

`--force-steam=off` + `--fastmp` + `--cliendID` 可以让您快速测试多人模式相关的 Mod，而不需要每次都找别人的账号，用法如下：

--fastmp [type]：`host` `host_standard` `host_daily` `host_custom` `load` `join`
--clientId [id]：指定客户端的 NetGameService ID。fastmp 时 host ID 必须1，client 必须 1000 ~1002，否则无法进行“多人读档”操作。

或者，如果您喜欢，您也可以使用 [gbe_fork](https://github.com/Detanup01/gbe_fork) 来模拟 Steam API 。

## 获取更多 Mod 与玩家交流群

目前已经有相当多的 Mod 出现和在路上：
* **[Slay the Spire 2 Discord](https://discord.gg/slaythespire)**: 英语官方社区，内有专门的 Modding 频道。[此帖子](https://discord.com/channels/309399445785673728/1480486428843446383) 包含了早期 Mod 的列表和链接。
* **[中文 Mod 统计表格 (腾讯文档)](https://docs.qq.com/sheet/DZnNQTnBFdXVJeGpH?tab=BB08J2)**: 中文，标注了已在制作中和已发布的 Mod（中文社区较多），欢迎补充（我会不定期与 Discord 同步）。主阵地在下面的核心开发群，开发群内讨论偏向技术，如果您无意开发请勿加入。

**玩家交流群：**
* 哔哩哔哩 UP 主 *蝴蝶是幼虫* 的泛 Mod 群：`1048696711`

---

# 🛠️ 开发者与 Mod 制作指南

## 教程与学习资源

* **[Glitched Reme 的塔2 Mod 教程](https://github.com/GlitchedReme/SlayTheSpire2ModdingTutorials)**（中文，持续更新中）
* **[BaseLib wiki](https://github.com/Alchyr/BaseLib-StS2/wiki)** （BaseLib wiki，除了围绕 baselib 本身也提供了一些额外的教程）

**语言与引擎基础：**
如果你打算把 Mod 开发作为学习 C# 的机会，建议系统学习：
* **C# 语言**: [C# Learning (En)](https://learn.microsoft.com/en-us/training/paths/csharp-first-steps/) / [C# 学习 (中)](https://learn.microsoft.com/zh-cn/training/paths/csharp-first-steps/)
* **Godot 引擎**:[Godot (En)](https://docs.godotengine.org/en-us/4.x/getting_started/step_by_step/index.html) / [Godot (中)](https://docs.godotengine.org/zh-cn/4.x/getting_started/step_by_step/index.html)
  *(注：微软和 Godot 的官方中文翻译可能不完善，推荐看英文原版并配合大模型/翻译插件)*

如果你希望长期从事简单 Mod 开发，你可能需要长期和下面几个家伙打交道，学习一下：
* **[Harmony Lib](https://harmony.pardeike.net/articles/intro.html)**：Harmony Patch，塔2 内置，必修。
* **[MonoMod](https://github.com/MonoMod/MonoMod)**。选修。
* **[BepInEx](https://github.com/BepInEx/BepInEx)**：选修。一个流行的 C# Mod 框架，通过注入 winhttp.dll 来加载 Mod。

（这三者是互有关联的，Harmony 负责补丁，MonoMod 负责修改和重写代码，BepInEx 负责在游戏不主动加载 Mod 的情况下强行注入 Mod。）

## 样板与库文件

推荐参考 Alchyr 的样板，一键构建：
* **[ModTemplate-StS2](https://github.com/Alchyr/ModTemplate-StS2)**：一个开箱即用的尖塔2 C# Mod 工程模板，内置构建 csproj 脚本和基础目录结构，克隆后改改就能上手。
* **[BaseLib-StS2](https://github.com/Alchyr/BaseLib-StS2)**：类似塔1 BaseMod+stslib 的基础库。提供一个简易的设置面板，一些基础库，反射发现钩子 和一大堆样板 buff 和 keyword 等。
* **[STS2-RitsuLib](https://github.com/BAKAOLC/STS2-RitsuLib/**：一个更重、更显式的基础库。提供了非常丰富的设置面板。大多数钩子倾向于多让您集中手动注册各种数据。也没有把 Keyword 塞进原生游戏而是分开单独判断。提供了丰富的 fmod 支持。

## 工具与反编译

**代码反编译：** 尖塔2没有进行任何混淆，可使用 [ilSpy](https://www.github.com/icsharpcode/ILSpy) 或[dnSpyEx](https://github.com/dnSpyEx/dnSpy) 反汇编出游戏源代码。
**资源解包：** 使用 [GD RE Tools 的 gdsdecomp](https://github.com/GDRETools/gdsdecomp/) 将游戏的资源解包。相较于下面的 pck explorer， gdsdecomp 会还原.import的压缩格式到原始路径，甚至拆分纹理图集 (Atlas)。



