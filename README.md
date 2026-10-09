<p align="center">
  <img src="docs/logo.png" width="128" alt="Source2PortTools">
</p>

<h1 align="center">Source2PortTools</h1>

<p align="center">
  Ports Source 1 models, textures, materials, particles, sounds, maps and SFM sessions to <b>Source 2</b> (Half-Life: Alyx / S2FM) in one click.
</p>

<p align="center">
  <a href="https://github.com/izzetyarali-arch/Source2PortTools/releases/latest"><b>⬇ Download</b></a> ·
  <a href="https://discord.gg/TnfQdnaF7g"><b>💬 Discord</b></a> ·
  <a href="https://github.com/izzetyarali-arch/Source2PortTools/issues/new/choose"><b>🐞 Report a Bug</b></a>
</p>

---

## What does it do?

Point it at a Source 1 game folder (HL2, Black Mesa, L4D2, SFM, GMod content...), pick your Half-Life: Alyx addon as the target, and the program does the rest:

| Source 1 | Source 2 output |
|---|---|
| `.mdl` models | `.vmdl` / `.vmdl_c` + binary DMX meshes, animations, physics and ragdolls, bodygroups, flexes/morphs |
| `.vtf` textures | `.png` / `.tga` + `.vtex` / `.vtex_c`, generated PBR maps (normal, roughness, metalness, AO, height) |
| `.vmt` materials | `.vmat` / `.vmat_c` (Valve's HLA shaders, eye shader and wrinkle maps included) |
| `.pcf` particles | `.vpcf` / `.vpcf_c` |
| Sounds (`.wav`, `.mp3`) | `.vsnd_c` + sound events (`.vsndevts`) |
| Skyboxes | HDR panorama |
| SFM sessions (`.dmx`) | Source 2 Filmmaker sessions *(new)* |
| `.bsp` maps | `.vmap` *(experimental)* |

No external tools needed: no Crowbar, Blender or VTFEdit. Everything runs inside the program, compiling included: the program has its own compiler for models, materials, textures, particles and sounds. Valve's `resourcecompiler.exe` from the Workshop Tools is used for maps and for the few things the program's own compiler does not cover yet.

---

## Highlights

- **Native C++17:** a single multi-threaded application with a Qt5 interface. No Python, no external converters.
- **Built-in model decompiler:** reads `.mdl` (v44–v49), `.vvd`, `.vtx`, `.phy` and `.ani` directly and writes binary DMX meshes and animations plus `.vmdl`, keeping skeleton, animations, flexes and physics.
- **Its own compiler:** models, materials, textures, particle systems, sounds and sound events are written straight to their compiled Source 2 files, without resourcecompiler. It comes with the material types and particle rules it has learned from resourcecompiler, so a fresh install compiles most content directly. In Preferences you choose the program's compiler, resourcecompiler, or both (the default).
- **SFM sessions to S2FM:** animation, cameras, lights and the timeline carry over; bones are matched to the ported skeletons, and the models, materials, sounds and particles the session uses are found and ported with it.
- **GPU work:** BC7 texture compression, AI upscaling, PBR maps and image filters run on the graphics card through Direct3D 12 + DirectML (tensor cores), Direct3D 12 or Direct3D 11.
- **PBR material generation:** Source 1 shaders (`VertexLitGeneric`, `LightmappedGeneric`, `EyeRefract` and more) become Source 2 PBR materials, on the same shaders Valve picks in Half-Life: Alyx. Normal, roughness, metalness and AO maps are generated, optionally with a trained AI model, and textures can be upscaled by an AI upscaler.
- **Built-in AI:** a small neural network (about 5.8 million parameters) trained from scratch for this program helps the port along: texture upscaling and cleanup, PBR maps, surface types and wrinkle maps. It runs locally on the CPU or GPU, with no internet connection and no third-party model.
- **Missing file recovery:** models, materials, particles and sounds a port needs but the source folder lacks are looked up in installed games (loose files and `.vpk` archives) and ported with it.
- **Parallel by design:** conversion runs on every core; models compile in parallel while materials are still compiling.
- **12 languages and themes:** English, Turkish, French, German, Spanish, Portuguese (Brazil), Russian, Polish, Ukrainian, Chinese (Simplified), Japanese and Korean. Make your own theme from any built-in one and share it as a single `.s2theme` file.

---

## Measured Performance

**The full Half-Life 2 content folder**, conversion only (no compiling), textures upscaled to 2048×2048, 16 worker threads, source on an HDD (measured with v1.0.0):

| | |
| :--- | :--- |
| Source | 43,791 files, 8.77 GB |
| Assets converted | 27,779 |
| Output | 110,911 files, 47.1 GB |
| Total time | **49 min 26 s** |
| Peak memory | 7.45 GB |

| Stage | Files | Time |
| :--- | ---: | ---: |
| Models | 3,312 | 65 s |
| Textures | 7,243 | 23 min 12 s |
| Materials | 7,052 | 22 min 36 s |
| Particles | 61 | 1 s |
| Sounds / maps / other | 10,111 | 75 s |

Over 90% of the time goes to texture upscaling and PBR map generation. With the texture size set to **Original**, both the time and the output size drop sharply. Since v1.1.0 the AI runs on the GPU's tensor cores, and upscaled textures are kept, so a texture ported again is not upscaled a second time.

**Half-Life: Alyx character port** (models, materials and full compiling): **17.6 s**.

Times depend on the drive (HDD/SSD), core count, GPU and texture sizes.

---

## Details

### Models & Animations
- Embedded and external animation packages (`*_animations.mdl`, `$includemodel`) are extracted and linked to the model.
- Meshes, physics and animations are written as binary DMX only.
- `.phy` physics models become Source 2 collision shapes; ragdolls get one body per bone with their Source 1 joint limits.
- Flex sliders keep their Source 1 controllers (`jaw_drop`, `blink`, right / left pairs...) and the rules that drive them, like in SFM. Optionally each right / left pair can be merged into one slider.
- Large models that studiomdl split into extra bodyparts (`clamped1`…`clamped8`) are kept whole.
- Damaged `.vtx` files, or ones written by a different compiler version, are read safely: an unreadable part is skipped instead of crashing.
- Eye reflections and eye sockets are set up for human models.

### Textures & Materials
- About 35 VTF formats are decoded in-process: DXT1/3/5, ATI1N/ATI2N, 8-bit RGB/RGBA variants and HDR (16/32-bit float).
- Animated textures and sprite sheets become `.mks` sheets, downscaled when needed to fit.
- Material paths follow the model's `$cdmaterials`. When a model built from parts of several packs has a material in another folder, it is found by file name.
- Optional parallax: wall and floor materials get a height map and HLA's parallax shader, so bricks and stones gain depth.

### Maps *(experimental)*
- `.bsp` brushes, faces, displacements, entities, static props and I/O connections are rebuilt as a Hammer 5 `.vmap`, without an intermediate `.vmf`.
- Displacement blend painting becomes Hammer 5 vertex paint; overlays and decals are placed on their surfaces; grass sprites and detail models are carried over.
- Files embedded in the `.bsp` (community maps' own textures, models and sounds) are extracted, and the materials, models, sounds and particles the map uses are found and ported with it.
- Each kind of entity (NPCs, sounds, props, lights, triggers, particles, decals, grass...) can be left out of the map.
- `light_environment`, `light` and `light_spot` become Source 2 lights; skybox names become `env_sky`; a light probe volume is added automatically.

### Resource Use
- Cores, the RAM limit, process priority and the texture cache size are set in Preferences. The RAM limit is enforced: at the limit no new file is started until running ones finish, so the port slows down instead of stopping.
- The status bar shows the program's memory, CPU use and free space on the destination drive; when the drive fills up, the port stops with one clear message.
- When the source folder is on an HDD, file scanning uses fewer threads so the disk head does not keep seeking.
- Folder and file names that would exceed Windows' 260-character path limit are shortened automatically.

---

## Requirements

- Windows 10 / 11 (64-bit)
- DirectX 11 (feature level 11_0) capable GPU; DirectX 12 recommended
- Half-Life: Alyx + **Half-Life: Alyx Workshop Tools** (needed for maps; recommended for everything else, the program finds `resourcecompiler.exe` by itself)

## Install

Download the latest `.zip` from [Releases](https://github.com/izzetyarali-arch/Source2PortTools/releases/latest), extract it anywhere and run `Source2PortTools.exe`. No installer; the Visual C++ runtime comes with the program in its `runtime` folder.

## Command Line

`Source2PortTools.exe` is the interface. Everything can also be done from a terminal with **`Source2PortTools-cli.exe`**, next to it:

```bash
# Full port (no compiling):
Source2PortTools-cli port "E:/hl2" "C:/Steam/steamapps/common/Half-Life Alyx/content/hlvr_addons/my_addon"

# Port and compile, looking for missing models and textures in installed games:
Source2PortTools-cli port "E:/hl2" "C:/.../hlvr_addons/my_addon" --compile --rescue

# Selected files only:
Source2PortTools-cli port "E:/hl2" "C:/.../hlvr_addons/my_addon" models/alyx.mdl models/props_c17/oildrum001.mdl

# Only models and materials:
Source2PortTools-cli port "E:/hl2" "C:/.../hlvr_addons/my_addon" --types models,materials

# Convert an SFM session for Source 2 Filmmaker:
Source2PortTools-cli session "E:/sfm/elements/sessions/scene.dmx" "C:/.../my_addon"

# Convert a BSP map to a Hammer 5 VMAP:
Source2PortTools-cli map "E:/hl2/maps/d1_trainstation_01.bsp" "maps/d1_trainstation_01.vmap"

# Decompile a single model to SMD/DMX + QC:
Source2PortTools-cli decompile "models/alyx.mdl" "output/alyx"
```

| Command | What it does |
|---|---|
| `port <source> <addon> [file ...]` | Port a Source 1 folder, or only the listed files, into an HLA addon |
| `session <in.dmx> <addon> [out.dmx]` | Convert an SFM session for Source 2 Filmmaker |
| `decompile <model.mdl> [folder]` | Decompile a model (`--transform` turns it to Source 2 axes) |
| `map <map.bsp> <out.vmap>` | Convert a BSP map to a VMAP |
| `texture <file.vtf> [out prefix]` | Decode a VTF to TGA (`--mips` for every mip level) |
| `wrinkle <color> [normal] [ao]` | Make squish / stretch wrinkle maps for a face texture |
| `compile-model <addon .vmdl>...` | Compile models without resourcecompiler (`--out <folder>` to write elsewhere) |
| `compile-material <addon .vmat>...` | Compile materials (or every `.vmat` in a folder) without resourcecompiler |
| `compile-texture <addon .vtex>` | Compile a `.vtex` without resourcecompiler |
| `compile-particle <addon .vpcf>...` | Compile particle systems without resourcecompiler |
| `compile-sound <addon .wav/.mp3>...` | Compile sounds without resourcecompiler |
| `check` | Check that every program module works |
| `version` | Show the version |
| `help` | List every command and option (`<command> --help` works too) |

`port` options: `--types` (models, textures, materials, particles, maps, sounds, sessions), `--compile`, `--compiler both|native|rc`, `--rescue`, `--merge-flexes`, `--no-pbr`, `--pbr-ai`, `--max-size <N|original>`, `--format png|tga`, `--flat`, `--skip-existing`, `--threads <N>`, `--gpu-api d3d11|d3d12|directml`. The `compile-*` commands take `--threads <N>` to compile several files at once. Messages follow the interface language, or `--lang tr|en`; `--verbose` shows the full technical log. A mistyped option is an error, not ignored. Exit code: `0` done, `1` failed or finished with errors, `2` wrong usage.

---

## Found a bug?

[Open an issue](https://github.com/izzetyarali-arch/Source2PortTools/issues/new/choose) or tell us on [Discord](https://discord.gg/TnfQdnaF7g). Please attach:
- the latest run log from the `logs` folder (`Run_Log_....txt`) and `source2porttools.log` (next to the executable),
- the `.dmp` file from the `crash` folder if it crashed,
- the name/path of the file that failed and a screenshot.

The program sends no data anywhere. The only exception is the optional feedback panel, which sends what you type there when you choose to send it.

---

<p align="center">
  Developer: <b>İzzet Yaralı</b><br>
  <sub>Closed source — this repository hosts releases and bug reports only.</sub>
</p>
