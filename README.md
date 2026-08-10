# OpenRow

RELEASE DATE FOR SOURCE CODE IS TBD ONCE CODE IS STABLE AND WE REACH BETA, WHICH IS SOON.

**An open-source, from-scratch reimplementation of the Saints Row (2006) game engine.**

OpenRow is a native reimplementation of the engine that shipped with the original
*Saints Row* on Xbox 360. It is not an emulator and not a mod loader  it is new C++
that reads the retail game's data files directly and runs the game on modern hardware,
in a modern renderer, at framerates and resolutions the original never targeted.
See https://ko-fi.com/itismrwonderful if you want to buy a coffee :)

> **Status: pre-alpha.** OpenRow boots, streams Stilwater, plays the intro cutscene,
> runs player creation, and executes the shipped mission scripts. It is not a finished
> game, and it is not yet a replacement for playing the original on hardware. See
> [Status](#status) for an honest breakdown.

---

## Table of contents

- [What OpenRow is](#what-openrow-is)
- [What OpenRow is not](#what-openrow-is-not)
- [Status](#status)
- [Requirements](#requirements)
- [Building](#building)
- [Running](#running)
- [Repository layout](#repository-layout)
- [How this is built](#how-this-is-built)
- [Contributing](#contributing)
- [Legal](#legal)
- [License](#license)

---

## What OpenRow is

- **A native engine.** No PowerPC interpretation, no Xenia, no binary translation at
  runtime. Every subsystem  streaming, renderer, physics, animation, audio, UI,
  mission scripting  is reimplemented in portable C++.
- **A data-compatible engine.** OpenRow reads the retail `.vpp_xbox2` archives and the
  formats inside them: BBChunk world geometry, `.csc` cutscenes, `.anim` clips, XTBL
  tuning tables, PEG/texture archives, XACT/XMA2 audio banks, Havok collision, and the
  shipped Lua 5.0 mission scripts.
- **A modern renderer.** Direct3D 12 is the primary Windows backend; D3D11, D3D9,
  Vulkan, OpenGL, OpenGL ES, and Metal backends live alongside it behind a common
  `Sr1RenderHost` interface.
- **Measurement-first.** Behaviour is reproduced against captured reference frames and
  the original binary's logic, then verified by automated gates that fail on
  regression  not by eyeballing it. See [How this is built](#how-this-is-built).

## What OpenRow is not

- **Not an emulator.** It will not run arbitrary Xbox 360 titles, and it does not
  execute the original executable.
- **Not a source port.** No original source code was used. See [Legal](#legal).
- **Not a redistribution of the game.** OpenRow ships **zero** game assets. You must
  supply your own legally-obtained copy of Saints Row.
- **Not affiliated** with Volition, Deep Silver, THQ, Plaion, or Embracer Group.

---

## Status

Subsystem coverage as it currently stands. "Implemented" means real code paths exist
and are exercised by tests  not that they are bug-free or feature-complete.

| Area | State | Notes |
| --- | --- | --- |
| Boot & streaming | Working | `.vpp_xbox2` mount, resource residency, Stilwater world load |
| Renderer (DX12) | Working | Primary backend; DX11/DX9/Vulkan/GL/GLES/Metal hosts also present |
| World geometry | Working, defects open | BBChunk decode, material/LOD resolution, PVS culling |
| Intro cutscene | Working, defects open | `.csc` playback, per-shot cameras, actor placement, subtitles |
| Player creation | Working | Morph targets, presets, wardrobe, lighting |
| Character animation | Partial | Locomotion states online; some pose/resolver defects remain |
| Vehicles & physics | Partial | Havok-derived vehicle kit, wheel integration, collision |
| Mission scripting | Working | Stock Lua 5.0 host runs the 39 shipped mission scripts |
| Console | Broadly implemented | ~967 of the retail engine's 979 commands are bound |
| Audio | Partial | XACT/XMA2 decode, radio, dialog, cue banks |
| AI, traffic, notoriety | Partial | Pathing and police response present; wiring gaps remain |
| UI / HUD | Partial | Frontend, HUD, menus |
| Online / multiplayer | Experimental | Clean-room backend, dedicated server, relay transport |



### Known open defects

- Character silhouettes render pale under some lighting paths.
- Duplicate/phantom pillar geometry in certain city blocks.
- Occasional pedestrian T-pose when an animation fails to resolve.
- Memory exhaustion on full-city load in some configurations.
- Held-prop orientation (briefcase) is wrong in the intro cutscene.

Tracked in `docs/saintsrow/` alongside the eliminated hypotheses for each  please read
the existing investigation before re-opening one of these.

---

## Requirements

**You must own a copy of Saints Row (Xbox 360, 2006).** OpenRow reads the game's data
files from a directory you point it at. Nothing is bundled and nothing is downloaded.

| | |
| --- | --- |
| Game data | An extracted Saints Row (Xbox 360) install containing the `.vpp_xbox2` archives |
| Compiler | MSVC 2022 / clang-cl (Windows), or Clang 15+ / GCC 12+ elsewhere |
| Build system | CMake 3.20 or newer |
| Language | C++20 |
| GPU | Direct3D 12 capable (Windows), or Vulkan 1.2 / OpenGL 4.3 / Metal |
| Dependencies | zlib; optional FFmpeg, EOS SDK, Discord, Odin voice |

---

## Building

Configure and build from the repository root:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
```

```bash
cmake --build build --config Release --target saints_row
```

The produced binary is `build/Release/saints_row.exe` (Windows) with its shader
packages and authored assets staged beside it by post-build steps.

> **Check the link step, not just the exit code.** `cmake --build` can exit `0` while an
> individual target fails to link. If a gate or smoke target misbehaves, confirm its
> binary actually exists on disk before trusting the build.

---

## Running

OpenRow locates game data through the `SR1_GAME_ROOT` environment variable:

```bash
SR1_GAME_ROOT=/path/to/saints_row_install ./build/Release/saints_row.exe
```

On Windows PowerShell:

```bash
$env:SR1_GAME_ROOT = 'D:\games\saints_row'; .\build\Release\saints_row.exe
```

The runtime defaults to a 1280x720 window. Press the console key to reach the command
console; nearly the whole retail command surface is bound, including the debug and
metrics commands.

---

## Repository layout

```
saintsrow/
├── runtime/      Core runtime, resource residency, frame loop        (97 units)
├── render/       Renderer, materials, shaders, PVS, post-processing  (66 units)
├── gameplay/     Gameplay systems, entities, missions, activities    (34 units)
├── mechanics/    Discrete game mechanics recovered from the original (34 units)
├── ui/           Frontend, HUD, menus, player creation               (32 units)
├── world/        City streaming, zones, traffic, time of day         (32 units)
├── online/       Multiplayer, sessions, matchmaking, transport       (28 units)
├── data/         File formats: VPP, BBChunk, XTBL, PEG, CSC          (27 units)
├── audio/        XACT/XMA2 decode, radio, dialog, cues               (24 units)
├── server/       Dedicated server and admin control                  (22 units)
├── physics/      Havok-compatible collision and vehicle dynamics     (18 units)
├── script/       Lua 5.0 host and the mission verb surface           (15 units)
├── host/         Per-backend render hosts (DX9/11/12, VK, GL, Metal) (15 units)
├── console/      Command table and dispatch                          (8 units)
├── effects/      Particles, emitters, triggers, explosions           (8 units)
├── animation/    Skeletons, clips, morphs, retargeting                (5 units)
├── ai/           Pedestrian and police behaviour                      (4 units)
├── tools/        Offline probes, audits, and the radio editor         (9 units)
└── tests/        Smoke tests and capture gates
```

Design notes, format specifications, and per-subsystem parity audits live in
`docs/saintsrow/` and `docs/saintsrow/recomp_specs/`.

---

## How this is built

OpenRow's working method is worth understanding before contributing, because it is
stricter than most reimplementation projects.

1. **Specify from observed behaviour.** Each subsystem gets a written specification in
   `docs/saintsrow/recomp_specs/` derived from the retail data files and observed
   runtime behaviour, before any implementation lands.
2. **Implement in portable C++.** No scripting languages in shipped code. Python is
   permitted only for throwaway analysis scratch work.
3. **Prove it on a frame or a receipt.** A change is not done because it compiles or
   because a counter says `applied=1`. It is done when a capture gate, a frame
   comparison against reference footage, or a smoke test says so.
4. **Record the negative results.** Every ruled-out hypothesis gets written down. A
   large fraction of the documentation in this repository exists to stop the next
   person from re-litigating a dead lead.


---


## Contributing

Contributions are welcome. Before opening a pull request:

- **Read the relevant spec** in `docs/saintsrow/recomp_specs/` and the closed-defect
  index. Many "obvious" bugs have been investigated and ruled out already.
- **Match the house style.** The codebase follows an id Tech-derived C++ standard 
  see `CODING_STANDARD.md`. Write code that reads like the code around it.
- **All engine code is C++.** No Python, no scripting glue in shipped code.
- **Bring evidence.** For any behavioural change, include the gate output, capture
  diff, or smoke test that demonstrates it. "Looks right" is not a result.
- **Never commit game assets**, extracted archives, or decrypted executables.

Good first areas: format decoders with documented gaps, unreached translation units
that need wiring to a live root, and the open defects listed above.

---

## Legal

OpenRow is an independent reimplementation. It contains no code, assets, audio, text,
or artwork from Saints Row. The engine was written by observing the behaviour and data
formats of software the authors legally own  the same basis on which projects like
ScummVM, OpenMW, and OpenRCT2 operate.

**You must provide your own legally-obtained copy of the game.** OpenRow will not run
without it, and this project does not distribute, link to, or assist in obtaining game
data.

*Saints Row* is a trademark of its respective owners. This project is not affiliated
with, endorsed by, or sponsored by Volition, Deep Silver, THQ, Plaion, or Embracer
Group.

---

## License

OpenRow is released under the **GNU General Public License v3.0**. See `LICENSE`.



---

## Acknowledgements

To Volition, for a game worth this much effort  and to the reverse-engineering and
game-preservation communities whose tooling and documentation made the work tractable.
