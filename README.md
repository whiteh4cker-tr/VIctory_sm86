# VIctory_sm86

A fork of [VIctory](https://github.com/Marvisak/VIctory), a compatibility layer that runs an Xbox Series title
natively on Windows from files **you** supply. VIctory_sm86 is tuned for **NVIDIA RTX** GPUs: DLSS Super Resolution and
DLSS Frame Generation in place of the title's own upscaling, the Series X profile at a native 2560x1440, and fewer
stalls at scene cuts.

> **This repository contains no game files, code or assets.** You need your own legally obtained copy of the
> title. VIctory_sm86 is not affiliated with or endorsed by the title's developer or publisher, nor by NVIDIA.
> See [docs/asset-policy.md](docs/asset-policy.md).

> [!NOTE]
> As of commit bd41424, the software is stable. The fishing mission has been completed without any crashes or
> major graphical glitches. 36 average FPS has been measured in the city using `-FrameGen`, `DLSS Quality`, and the
> Series X profile at a native 2560x1440 on an NVIDIA RTX 3090. 48 GB of RAM is recommended.

## Where it stands

Tested on one PC: Windows 11, Ryzen 7 7800X3D, 32 GB RAM, **RTX 4070 SUPER 12 GB**, NVMe SSD; the Xbox Series
build **v1.0.2.1** of the title, Series X profile, 2560x1440 output, DLSS Quality — and frame generation on an
**RTX 3090** with the Ampere add-on below (2x presented). Frame times are the layer's
own count, in the opening scenes unless noted.

| | Frame time |
|---|---|
| Opening, gameplay | 34-37 ms (27-30 fps); our CPU work per frame sets it |
| Opening, cutscenes | median 36-38 ms, 95th percentile 49-58 ms, no second over 100 ms |
| Open water (first boat ride) | about 47 ms (21 fps); the GPU sets it there |
| With `-FrameGen` (2x) | every frame the title renders is shown twice; 4x-6x on RTX 50 or with `-FrameGenX`, as the card allows |
| Memory | about 8 GB of VRAM; 32 GB of RAM is enough, 64 GB is more comfortable |

What is known to be wrong:

- **No ray tracing.** The title traces its indirect light, reflections and lamp shadows in its own compute
  shaders. They are off (`VICTORY_BVH_BUILDER=1` turns them on: about 3 fps and a broken picture today). An
  estimate stands in for indirect light and for reflections of opaque surfaces: rooms are lit, but reflections
  are blurry, and in cutscenes they can still show the sky's bright colour.
- **Left out:** the heat-haze / in-scattering pass (it smears blocks of pixels, also on water) and the decal
  heap's defragmentation (5.6 ms of GPU a frame). `VICTORY_LEAVE_OUT` names them.
- **Hitches:** some single frames of 70-190 ms in cutscenes while the title streams.
- **Frame rate:** the launcher no longer caps at 30 fps by default (`-Fps 1000`); `-Fps 30` restores the title's
  own rate. The layer's CPU work still keeps most scenes near 30, so 60 fps remains the goal.
- **The first run of every new build stutters** while the driver compiles pipelines (see *Warming up* below).

## What this port adds

This tree carries the work after `02fc18a` from
[trustno1-coder/VIctory-NVIDIA](https://github.com/trustno1-coder/VIctory-NVIDIA), ported here. The fork's
automatic NVIDIA SDK downloader is deliberately left out; install Streamline yourself (below).

- **F1 settings menu, in every build.** DLSS mode and model, frame generation, the frame-rate cap, console profile,
  picture size, the light estimates and the main speed switches. It writes `victory-settings.txt` (or the file
  `VICTORY_MENU_FILE` names), applies it at the next start, and can save and restart. While it is open the title
  gets no keyboard; a pad still works.
- **Frame generation from 2x to 6x, live.** `VICTORY_FRAMEGEN_X=2..6` chooses the starting amount; `0` makes it
  ready but off. The layer asks the driver for the card's maximum and uses no more. The F1 menu changes the amount
  while the title runs; the launcher's `-FrameGenX` and `-FrameGen` select the starting amount.
- **No frame-rate cap by default.** `-Fps` defaults to 1000 (`VICTORY_FRAME_INTERVAL_US=1000`); `-Fps 30` restores
  the title's own rate. The caption and the top-left line show rendered and presented rates when frame generation
  is on.
- **FPS line and F3 stats panel.** The FPS line is off by default (`VICTORY_FPS_LINE=1` shows it). F3 toggles the
  stats panel in every build now (`VICTORY_PANEL=1`), not only in the diagnostics build; it also shows the
  frame-generation multiplier.
- **`victory play`, or double-click.** `target\release\victory.exe play <path to game_scarlett_master.exe>` uses
  the same settings as the PowerShell launcher. Put `victory.exe` beside the title's executable and start it with
  no arguments to play. Settings and logs stay under `%LOCALAPPDATA%\victory`; nothing is downloaded.
- **Window and input, off by default.** `VICTORY_FULLSCREEN=1` (with `VICTORY_FULLSCREEN_RES=WxH`),
  `VICTORY_IME_ENGLISH=1` for an English keyboard layout, and `VICTORY_MOUSE_CAPTURE=1` to keep the pointer inside
  and read relative motion (`VICTORY_MOUSE_WARP=1` recentres it). The F1 menu can turn them on for the next start.
- **PS5 pads.** DualSense and DualSense Edge are read from HID over USB or Bluetooth. `VICTORY_DUALSENSE=0` turns
  them off and `=1` keeps them beside a remapper; by default they fill the slots XInput leaves free (a remapper's
  XInput pad wins).
- **Open water without dark blocks.** On a 32-lane-wave host the two tiled water-light programs are left out;
  `VICTORY_WATER_LIGHT=1` runs them as before (green water, dark blocks).
- **One-click launcher and extreme modes.** `PLAY_FRAMEGEN.cmd` (or `scripts\play-framegen.ps1 -Mode 1..6`)
  finds the dump, backs up and verifies the saves, tunes the worker pools by CPU count, and starts through
  `play-VIctory_sm86.ps1`. On RTX 50 frame generation starts at 4x (`TITLE_FRAMEGEN_X=2..6` overrides;
  `TITLE_NO_TUNE=1` skips the pool tuning); modes 5 and 6 raise the game to High priority. NVIDIA's files are
  checked for, never downloaded.

## What you need

- Windows 10 or 11 (x64), a CPU with AVX2, 32 GB of RAM or more, an SSD.
- An NVIDIA GeForce RTX GPU with a recent driver for DLSS (RTX 40 or later for Frame Generation, with
  hardware-accelerated GPU scheduling on; **RTX 30 and 20 too**, with the small add-on in *Set up*). Any
  Direct3D 12 GPU with shader model 6.6 and mesh shaders runs the layer without DLSS.
- **Your own dump** of the title's Xbox Series build v1.0.2.1 (the folder with `game_scarlett_master.exe`).
  Other builds may start, but the image patches (1440p output, FidelityFX path, frame rate) check every byte
  they change and leave a differing build alone.
- [Rust](https://rustup.rs) (the toolchain is pinned in `rust-toolchain.toml`; rustup fetches it), and the
  **Visual Studio Build Tools** with "Desktop development with C++" (MSVC and a Windows SDK).
- For DLSS: NVIDIA's **Streamline SDK** (below). You download it yourself; nothing of NVIDIA's is in this
  repository.

## Build

```powershell
git clone https://github.com/whiteh4cker-tr/VIctory_sm86
cd VIctory_sm86
cargo build --release --locked -p victory
```

The diagnostics build (frame timelines, captures, GPU timing) is optional:
`cargo build --locked --profile diag -p victory --features diagnostics`.

## Set up

1. **Point it at your copy.** Copy `victory.example.toml` to `victory.toml` (git-ignored) and set `game_dir` to
   your dump's folder. Keep your files **outside** this repository; the tools refuse otherwise.

   ```toml
   [paths]
   game_dir = 'D:\Games\MyDump'
   ```

2. **DLSS (optional, recommended on RTX).** Download the Streamline SDK release from
   <https://github.com/NVIDIA-RTX/Streamline/releases> (tested: `streamline-sdk-v2.14.16.zip`; it comes under
   NVIDIA's RTX SDK license, which you accept yourself), unpack it **outside** the repository, and point VIctory_sm86
   at its `bin\x64` folder:

   ```powershell
   [Environment]::SetEnvironmentVariable('VICTORY_STREAMLINE_DIR', 'D:\sdk\streamline-sdk-v2.14.16\bin\x64', 'User')
   ```

   Check it without the title: `target\release\victory.exe upscaler-check --device` (and `--present` for Frame
   Generation). Details: [docs/dlss.md](docs/dlss.md). Without Streamline the launcher runs without DLSS.

   **Frame Generation on RTX 30 and 20** (optional; RTX 40 and later need nothing): NVIDIA's DLSS-G is Ada-only,
   so take `sdli1995/dlssg_for_sm86` (MIT; binaries only, **not** in this repository) from
   <https://github.com/sdli1995/dlssg_for_sm86/releases>. Put what its release holds in the **same folder as this
   software**, next to `victory.exe`:

   ```
   <folder with victory.exe>\
     victory.exe
     version.dll        (the release's proxy)
     dlssg_sm86.ini     (its settings)
     dlssg_sm86\        (the whole folder, as it comes: backend, kernels, runtime, ...)
   ```

   Those three (`version.dll`, `dlssg_sm86.ini` and the `dlssg_sm86` folder) are what VIctory_sm86 looks for, and
   it finds them in the working directory, beside `victory.exe` and in the repository root of a `target` build;
   they are git-ignored, so nothing of the add-on is committed. To keep it somewhere else (a shared `D:\sdk`, or
   another version), name the folder that holds them:

   ```powershell
   [Environment]::SetEnvironmentVariable('VICTORY_DLSSG_SM86_DIR', 'D:\sdk\dlssg_for_sm86-0.4.2', 'User')
   ```

   It needs driver R580 or newer. VIctory_sm86 loads the add-on before Streamline, and makes the D3D12 device on
   the GPU the add-on uses (CUDA device 0) — on a PC with two NVIDIA GPUs the default adapter can be the other
   one, and frame generation then never starts.

3. **Translate the shaders ahead** (a few minutes, once per new build of VIctory_sm86):

   ```powershell
   $env:VICTORY_HOST_WAVE = '32'   # NVIDIA runs 32-lane waves; leave it out on AMD
   target\release\victory.exe shaders
   ```

## Run

The short way is the built-in launcher:

```powershell
target\release\victory.exe play 'D:\Games\MyDump\game_scarlett_master.exe'
```

or put `victory.exe` beside `game_scarlett_master.exe` and double-click it. The PowerShell launcher has the same
settings and more options:

```powershell
.\scripts\play-VIctory_sm86.ps1 -Exe 'D:\Games\MyDump\game_scarlett_master.exe'
```

| Option | What |
|---|---|
| *(none)* | Series X profile, 2560x1440, DLSS Quality (if Streamline is set up), no frame-rate cap, all fixes on |
| `-Dlss off` / `dlaa` / `balanced` / `performance` | the title's own upscaling, or another DLSS mode |
| `-FrameGen` | DLSS Frame Generation (RTX 40 or later, or RTX 30 / 20 with the add-on; about +1.4 GB of VRAM) |
| `-FrameGenX 2..6` | frames shown per rendered frame with `-FrameGen` (the card's maximum wins) |
| `-NoFrameGenReady` | do not make frame generation ready at start; only `-FrameGen` turns it on |
| `-Fps 30` / `60` / `1000` | frame-rate cap; 1000 is the default (no cap), 30 is the title's own |
| `-SeriesS` | the Series S profile: 1280x720 inside, faster, softer |
| `-Res 1080to1440` / `4k` | scene at 1080p upscaled by the title / the stock 3840x2160 output |
| `-Build diag` | the diagnostics build |
| `-More @{VICTORY_X='1'}` | add or override any switch |

In the game: **F1** is the settings menu (Up/Down pick, Left/Right change, Enter runs the line), **F3** shows and
hides the stats panel. The `victory play` options mirror the launcher: `--res`, `--dlss`, `--framegen N`,
`--no-framegen-ready`, `--fps`, `--series-s` and `--set VICTORY_X=1`.

Full screen: `VICTORY_FULLSCREEN=1` (or the F1 menu) makes the window borderless at the monitor's size;
`VICTORY_FULLSCREEN_RES=2560x1440` asks for another size. The older `.\scripts\fullscreen.ps1` still works.

Logs go to `%LOCALAPPDATA%\victory\logs`, saves to `%LOCALAPPDATA%\victory\data`. Close the game window to quit.

### Warming up (once per build or driver)

The first run of a build compiles pipelines while playing and stutters. Runs write down every pipeline the title
makes (`VICTORY_PIPELINE_BOOK=1`, set by the launcher); afterwards make them all ahead:

```powershell
$env:VICTORY_HOST_WAVE = '32'
target\release\victory.exe warm
```

The NVIDIA driver keeps its compiled pipelines **per program path**: a build run from another folder starts cold.

## Switches worth knowing

Every `VICTORY_...` switch is described in a comment where the code reads it. The launcher's full set is in
`scripts/play-VIctory_sm86.ps1`; these are the ones VIctory_sm86 added or changed:

| Switch | Launcher | What |
|---|---|---|
| `VICTORY_DLSS`, `VICTORY_DLSS_MODE` | `quality` | DLSS Super Resolution through the title's FidelityFX dispatch |
| `VICTORY_FRAMEGEN` | ready (`-FrameGen` turns on) | DLSS Frame Generation, with the picture before the HUD tagged |
| `VICTORY_FRAMEGEN_X` | `0` (ready, off) | frames shown per rendered frame: 2-6, or 0 for ready but off |
| `VICTORY_FPS_LINE` | off | FPS line at the top left; `1` shows it |
| `VICTORY_PANEL` | off | F3 stats panel in every build |
| `VICTORY_FULLSCREEN`, `VICTORY_FULLSCREEN_RES` | off | borderless fullscreen at the monitor's size, or `WxH` |
| `VICTORY_IME_ENGLISH` | off | switch to the en-US keyboard layout while the game has focus |
| `VICTORY_MOUSE_CAPTURE`, `VICTORY_MOUSE_WARP` | off | keep the pointer inside and read relative motion; recentre it |
| `VICTORY_DUALSENSE` | auto | PS5 pads from HID: `0` off, `1` beside XInput |
| `VICTORY_WATER_LIGHT` | off | run the tiled water light (green water, dark blocks) |
| `VICTORY_NATIVE_1440P` | on | Series X renders and outputs 2560x1440 (image patched) |
| `VICTORY_SOFT_VIEWS` | `copies` | the title's tiled-heap page mappings kept without Windows views: no freezes at scene cuts |
| `VICTORY_SECTION_MB` | off | physical pages in sections of at most N MiB: fewer map/unmap stalls when the camera moves (not with `VICTORY_PORT_CONSTANTS=1`) |
| `VICTORY_WC_NO_BACK`, `_MIN_KB` | `1`, `1024` | shader writes into big write-combined copies are not copied back |
| `VICTORY_AMBIENT`, `VICTORY_AMBIENT_SPECULAR` | `0.8`, `1` | estimates of indirect light and of reflections (no ray tracing) |
| `VICTORY_EARLY_DEPTH` | on | pixel programs keep the console's early depth test |
| `VICTORY_PRELUDE_UPLOADS`, `VICTORY_SHOW_NEW_LATER`, `VICTORY_TABLED_REREAD` | on | texture upload order and slot changes (flicker, device losses) |
| `VICTORY_SYNC_IN_PLACE`, `VICTORY_SPANS_QUICK`, `VICTORY_HEAP_SYNC_AT_SUBMIT` | on | cheaper descriptor-heap bookkeeping |
| `VICTORY_PORT_CONSTANTS` | off | the title's constants heap read in place: less CPU, but on a 12 GB card the GPU then waits (slower overall) |
| `VICTORY_FRAME_INTERVAL_MS`, `_US` | from `-Fps` | patch the title's own rate, or set our frame clock; the launcher sets `_US` |
| `VICTORY_BVH_BUILDER` | off | the title's ray tracing (slow, broken) |

## How it works, and the findings

- [docs/architecture.md](docs/architecture.md): the layer's parts.
- [docs/findings.md](docs/findings.md): measurements behind the choices.
- [docs/dlss.md](docs/dlss.md): DLSS and Frame Generation.
- [docs/fork-ray-tracing.md](docs/fork-ray-tracing.md): the title's ray tracing and the light estimate.
- [docs/replay.md](docs/replay.md): recording a frame and replaying it.
- [docs/asset-policy.md](docs/asset-policy.md): what may and may not be in this repository.

## Contributing

Issues and pull requests are welcome. Never attach or commit anything from the game: files, dumps, shader dumps
or extracted data. Describe faults technically. `cargo xtask policy` (also the pre-commit hook in `.githooks/`)
checks the asset policy.

## Credits

A tree of other people's work with more on top; the history is kept, and commits taken from elsewhere name the
commit and its author.

- **[VIctory](https://github.com/Marvisak/VIctory)** by Marvisak and its contributors: the loader, the runtime
  services, the Direct3D 12 back end and the shader recompiler.
- **[fxceless/victory-fork](https://github.com/fxceless/victory-fork)** and the people whose pull requests it carries.
- **[avatarmanz/VIctory-7900XTX](https://github.com/avatarmanz/VIctory-7900XTX)**: the submission line, write
  guards, relay and much of the speed work this fork starts from.
- **[codec-d/cool_fox](https://github.com/codec-d/cool_fox)**: the base this fork was taken from.
- **[DexNom/VIctory](https://github.com/DexNom/VIctory)**: native 1440p output, the FidelityFX path on Series X,
  ray tracing fixes.
- **[Belobug/VIctory-Belobug-AMD](https://github.com/Belobug/VIctory-Belobug-AMD)**: the light estimate, renderer
  ports (constants heap in place, heap records), texture upload order, the frame rate switch; ported commits name
  their source.
- **[FQY55kai/ZJOU-VICTORY](https://github.com/FQY55kai/ZJOU-VICTORY)**, **[myokex/victory-amd](https://github.com/myokex/victory-amd)**
  (with Spexivus, joroyt and tightfall), Joshua Odom, **Geckomacabre** and **cxy0072019-sketch**.
- **[XCL](https://github.com/ivall/xcl)** (GPL-2.0-only) was read for how the same title behaves under another
  layer; no code of it is here.
- **[trustno1-coder/VIctory-NVIDIA](https://github.com/trustno1-coder/VIctory-NVIDIA)** and its contributors
  (superboynick, Blitzball996, LucaruluDev): the F1 settings menu, live 2x-6x frame generation, built-in
  `victory play`, window/input controls, PS5 pads, the stats panel in every build and the open-water fix. The
  NVIDIA SDK downloader from that fork is not included.
- **[sdli1995/dlssg_for_sm86](https://github.com/sdli1995/dlssg_for_sm86)** (MIT) makes DLSS Frame Generation
  run on RTX 30 and 20; its release is loaded at run time from a folder you supply, and nothing of it is here.
- NVIDIA Streamline, DLSS and Reflex are NVIDIA's; they are loaded at run time from a folder you supply.
- Much of this fork's code was written with [Claude Code](https://claude.com/claude-code).

## License

GPL-3.0-or-later, as VIctory. See [LICENSE](LICENSE); the one file that is not covered is named in
[docs/asset-policy.md](docs/asset-policy.md).
