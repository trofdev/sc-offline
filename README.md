# sc-offline: 0.7.0-patch1

Independent security patch fork, unaffiliated with ChrisWareOffline and sc-offline.
Given contributors are affiliated with Griefernet, this repository is created only for future audits and patches to ensure nothing malicious happens.

**An offline, single-player mod menu for Star Citizen.** Spawn ships, NPCs and buildings, travel the star systems and wear Squadron 42 outfits from one in-game menu.

> [!WARNING]
> This mod may get your account banned. Use it at your own risk, and **only offline, in single player**.

> [!IMPORTANT]
> **This repository publishes no releases and no prebuilt binaries.** Clone it, read it, and [build it yourself](#build-it-yourself). Don't run a `dinput8.dll` or `sc-offline.exe` unless built locally. 
> Repo analysis encouraged, don't run or build code that was not verified to be safe.

## Disclosure

- **AI-assisted, under strict supervision.** The patches in this repository are made with Claude (Anthropic's AI model), under the maintainer's strict supervision. The maintainer reviews the changes before they are committed.
- **Purpose only:** making security patches, and checking that the original contributors did not introduce vulnerabilities or malicious code. This fork adds no features.
- **No guarantee.** This is an ongoing audit, not a certification. A clean result so far does not prove the code is safe; read it before you build it.
- **Found something?** Please [open an issue](https://github.com/trofdev/sc-offline/issues). For a vulnerability or anything that could harm users, use private reporting instead; see [SECURITY.md](SECURITY.md).
- **Updates:** this fork is updated every few weeks. There are no releases; you pull and [build](#build-it-yourself) the new source yourself.
- **Cloud Imperium Games:** this repository will comply with any request from Cloud Imperium Games.

## Patches in this fork

Version **0.7.0-patch1**, on top of upstream sc-offline 0.7.0. Full list in [CHANGELOG.md](CHANGELOG.md).

| # | Patch | Result |
| --- | --- | --- |
| 1 | **Self-update removed** | The launcher has no network code: no WinHTTP, no GitHub release check, no download, no staged file swap, no administrator update helper. `sc-offline.exe update`, the **Update** button and the `check_updates` / `update_channel` ini keys are gone. To update, `git pull` and rebuild. |
| 2 | **Discord presence removed** | No "Playing sc-offline" status, no Discord pipe code, no **Show on Discord** checkbox, no `discord_presence` key. The CI workflow that posted to Discord is gone too. |
| 3 | **No releases** | The release job, tag trigger, artifact upload, `tools/release-manifest.py` and the Wine update tests are removed. CI only checks and compiles. |
| 4 | **Bug-report links** | The launcher's bug-report page, the menu hint and `mod.log` point at this fork's issues, not the original's. |
| 5 | **Docs and config** | README, `docs/`, `sc-offline.ini`, `SECURITY.md`, `CONTRIBUTING.md` and the issue/PR templates match the above. |

Checked on the patched launcher: it builds with no warnings and imports no `winhttp`, `wininet` or `ws2_32`. The DLL in `src/` has no network code and was not changed.

### What it still does to your PC

Unchanged from upstream, and needed for the mod to run. Each is a switch in `sc-offline.ini` and is undone when the game closes:

- **Firewall:** `block_network` adds rules that block `StarCitizen.exe`, the RSI Launcher and `CrashHandler.exe` (inbound and outbound) while you play.
- **Hosts file:** `eac_hosts` appends `127.0.0.1 modules-cdn.eac-prod.on.epicgames.com`.
- **Easy Anti-Cheat:** `eac_rename` renames `EasyAntiCheat_EOS.exe` to `.bak`.
- **Administrator:** a small helper (this exe again, `--helper`) runs elevated to copy the mod in and out and make those changes. The game never runs as administrator.
- **Browser:** the bug-report button opens this fork's issue page in your browser. Nothing is sent by the launcher itself.

Details: [docs/launcher.md](docs/launcher.md#pc-changes).

## Build it yourself

You need Windows 10/11 x64, Git, and **Visual Studio 2026** (or its Build Tools) with the **Desktop development with C++** workload. The projects use the `v145` toolset. On VS 2022, open `sc-offline.slnx` and retarget both projects to `v143` first (Project → Retarget).

1. **Get the source** and read what you are about to build:

   ```powershell
   git clone https://github.com/trofdev/sc-offline.git
   cd sc-offline
   ```

2. **Build** in a *Developer PowerShell for VS* (so `msbuild` is on the path):

   ```powershell
   msbuild sc-offline.slnx /p:Configuration=Release /p:Platform=x64 /m
   ```

   Or open `sc-offline.slnx` in Visual Studio, pick **Release | x64**, and **Build Solution**. Output, in `x64\Release\`:
   - `dinput8.dll`: the mod (from `src/`)
   - `sc-offline.exe`: the launcher (from `launcher/`)

   The C runtime is linked statically, so no Visual C++ redistributable is needed.

3. **Assemble a play folder** anywhere outside the game folder, for example on your Desktop:

   ```powershell
   $dst = "$HOME\Desktop\sc-offline"
   New-Item -ItemType Directory -Force $dst | Out-Null
   Copy-Item x64\Release\dinput8.dll, x64\Release\sc-offline.exe, launcher\sc-offline.ini $dst
   Copy-Item data $dst -Recurse
   ```

   The folder must hold `sc-offline.exe`, `dinput8.dll`, `sc-offline.ini` and `data\` together. Don't copy anything into the game folder yourself.

4. **Optional check** that the launcher has no network libraries (`dumpbin` is in the same Developer PowerShell):

   ```powershell
   dumpbin /dependents x64\Release\sc-offline.exe
   ```

   Expect only system libraries such as `bcrypt`, `shell32`, `user32`, `advapi32`, `kernel32`. There should be no `winhttp.dll`, `wininet.dll` or `ws2_32.dll`.

**Updating:** `git pull`, rebuild, then copy `dinput8.dll` and `sc-offline.exe` over the play folder's. Leave your `sc-offline.ini` and your `data` files (`wallet.txt`, `spawn.txt`, `bookmarks.txt`, `locations_found.txt`) alone, and only copy a changed `data` file if you mean to.

**No Visual Studio, or on Linux/macOS:** `tools/check.sh` parses every source file with clang as a quick syntax check; it is not a build. See [docs/build.md](docs/build.md). Running under Wine is experimental: [docs/linux.md](docs/linux.md).

## Play

1. Close the RSI Launcher and the game.
2. Double-click `sc-offline.exe` in your play folder and click **Play**. The window shows the game folder it found. Windows asks for administrator rights; say yes (that prompt is for the helper only).
3. Once the game has loaded you in, press <kbd>M</kbd> to open the menu.

The window has **Play**, **Status**, **Install**, **Uninstall**, **Open settings** and **Open logs**, and a light that shows whether it is safe to go online. Every command also works from a terminal: `sc-offline.exe [play|install|uninstall|status|help]`.

| Key | Action |
| --- | --- |
| <kbd>M</kbd> | Open or close the menu |
| <kbd>F6</kbd> | Turn build mode on or off |
| <kbd>F7</kbd> | Save your position |
| <kbd>F8</kbd> | Teleport back to the saved position |

The launcher finds your install by itself (RSI Launcher records, usual folders, a drive search, then a folder picker) and remembers it. To force a folder, set `game =` in `sc-offline.ini`. Menu tabs: [docs/features.md](docs/features.md).

<p align="center">
  <img src="images/screenshot.webp" alt="A Vanduul holding a gun on a desert planet, with a line of Vanduul and a large ship behind it" width="85%">
</p>

## Troubleshooting

- **The menu doesn't open:** start the game with `sc-offline.exe`, not the RSI Launcher.
- **Wrong or missing game folder:** set `game =` in `sc-offline.ini` (for example `game = E:\Game\Star Citizen\StarCitizen`), or delete `data\game-path.txt` to search again.
- **Is everything set up?** Run `sc-offline.exe status`. It changes nothing.
- **"Easy Anti-Cheat is active":** set `eac_rename = on` in `sc-offline.ini`, or rename the file yourself.
- **Firewall or hosts step failed:** say yes to the administrator prompt; some antivirus tools lock the hosts file.
- **The game crashed or the PC shut down mid-game:** run `sc-offline.exe uninstall` before playing online.
- **A game update broke the mod:** expected until the mod is patched for that game version.

To report a bug, open an [issue](https://github.com/trofdev/sc-offline/issues) with `data/launcher.log`, `data/mod.log` and the game's `Game.log`.

## Go back online

1. Close the game.
2. Check the light in the sc-offline window. **Green** means the mod is out of the game folder and every PC change is undone. **Red** lists what is left: click **Uninstall**. (From a terminal: `sc-offline.exe status`, then `sc-offline.exe uninstall`.)
3. If you turned `eac_hosts` or `eac_rename` off and made those changes by hand, undo them by hand: rename `EasyAntiCheat_EOS.exe.bak` back, delete the `modules-cdn.eac-prod.on.epicgames.com` line from your hosts file, and run `ipconfig /flushdns`.

## More docs

| Doc | For |
| --- | --- |
| [Features](docs/features.md) | Every menu tab, including Squadron 42 |
| [Launcher](docs/launcher.md) | `sc-offline.exe`, `sc-offline.ini`, and what the launcher changes on disk |
| [Data files](docs/data-files.md) | The text files in `data/` and the environment variables |
| [Linux](docs/linux.md) | Running under Wine (experimental) |
| [Build](docs/build.md) | Build details, checks and CI |
| [Changelog](CHANGELOG.md) | What changed |

## Credits and license

sc-offline is **based on ChrisWareOffline 0.9.0-rc1** by Chris Ware and cloudyyrust (GPL-3.0). This repository is maintained independently and is not endorsed by its authors. The launcher, Squadron 42 tab and later fixes come from upstream sc-offline by scubamount; the patches above are this fork's.

Licensed under GPL-3.0; see [LICENSE](LICENSE). If you give anyone a binary built from this repository, you must offer them the matching source. Contributing: [CONTRIBUTING.md](CONTRIBUTING.md). Security problems: [SECURITY.md](SECURITY.md).

<sub>AI-assisted under supervision; see [Disclosure](#disclosure). It's a fan project, not made by or affiliated with Cloud Imperium Games or Roberts Space Industries. Star Citizen is a trademark of Cloud Imperium Games.</sub>
