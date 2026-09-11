# MeleeXR

An open-source, native port of **Super Smash Bros. Melee** (NTSC 1.02, `GALE01`) built from the
100%-matched [doldecomp/melee](https://github.com/doldecomp/melee) decompilation, targeting PC and
**Meta Quest** VR/MR (room-scale passthrough diorama) via OpenXR.

> **No game assets are included or downloaded.** You must own Melee and supply your own disc dump
> (ISO/RVZ). This repository contains only source code and build scripts. Not affiliated with Nintendo
> or HAL Laboratory.

## Architecture

```
doldecomp/melee C source  ──►  Aurora (GX / PAD / DVD / CARD / AX compat layer)
                                  └─► WebGPU (Dawn) ─► Vulkan / Metal / D3D12
                                        └─► OpenXR (Meta OpenXR SDK) — Quest stereo + passthrough
```

Same stack as [Dusklight](https://github.com/TwilitRealm/dusklight) (Twilight Princess), which is the
reference implementation for the Android/Quest shell.

## Status

Phase 0 — compile baseline. CI compiles every game + HSD translation unit against Aurora's Dolphin SDK
headers with gcc/clang × 32/64-bit and reports what breaks. Nothing runs yet.

| Phase | Goal |
|---|---|
| 0 | All decomp TUs compile natively against Aurora headers |
| 1 | Flat native PC build boots to CSS → Battlefield at 60 fps (macOS/Metal first) |
| 2 | Android/Quest flat build (Dusklight `platforms/android` template) |
| 3 | Stereo rendering: per-eye projection/view inside Aurora GX + Dawn↔OpenXR swapchain bridge |
| 4 | MR passthrough (`XR_FB_passthrough`), world-locked stage, floating HUD quad, controller input |
| 5 | Stretch: Slippi / rollback netplay |

## Layout

- `extern/melee` — fork of doldecomp/melee (submodule)
- `extern/aurora` — fork of encounter/aurora, branch `melee-rebase` (ribbanya's Melee header fixes)
- `CMakeLists.txt` — Phase 0 static library build of the decomp

## Building

```sh
git clone --recursive https://github.com/astelmach20/meleexr
cmake -S . -B build -G Ninja -DMELEEXR_M32=ON   # decomp assumes 32-bit pointers for now
ninja -C build -k 0
```

## Credits

doldecomp/melee contributors · [Aurora](https://github.com/encounter/aurora) (encounter, r-burns,
ribbanya) · Dusklight team · the GC/Wii decompilation community.

## Phase 0 status — compile check

`compile-check.yml` builds all 986 decomp TUs (`melee/`, `sysdolphin/`) against Aurora's headers on gcc and clang.

| target | failing TUs | notes |
|---|---|---|
| `-m32` | **0** | the decomp assumes ILP32; this is the real target |
| `-m64` | 33 | `offsetof` static asserts on pointer-bearing structs (`ToyED8Data`, …) — inherent to 32-bit layouts |

Each matrix cell is gated by `ci/baseline/<compiler>-m<bits>`: the job fails if the count rises and warns when the baseline can be lowered. Header fixes live in the forks (`astelmach20/melee`, `astelmach20/aurora` branch `melee-rebase`) and are pulled in by bumping the submodules.
