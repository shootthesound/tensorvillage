# Tensor Village

**Got AI models scattered across five drives? Tensor Village turns them into one.**

No copying. No symlinks. No renaming. It reads each file's header, works out what the
model actually *is*, and serves your whole library as a drive letter — assembled live,
from wherever the files already sit.

- **Nothing moves.** Models on `S:`, `W:` and `D:` appear in one folder together, and
  nothing is ever written to your model drives. Uninstall and every file is where it was.
- **Nothing needs a tidy filename.** `final_v3_REAL.safetensors` files itself just as
  accurately as a well-named one — the classifier never looks at the name.
- **Your apps get one path.** The folders follow ComfyUI's naming, so
  `M:\models\loras\flux` just works — and one click writes the config for you.

![The organized drive in Explorer](docs/images/drive.png)

*2 TB of models across several disks, in one place. Not a copy of them — that's the
drive, assembled live.*

What each folder is for:

```
M:\
├── models\                  ← ComfyUI folder naming, ready to point apps at
│   ├── checkpoints\sdxl\
│   ├── diffusion_models\flux\
│   ├── loras\krea2\
│   ├── vae\wan21\
│   ├── diffusers\flux\        ← whole pipelines, kept intact
│   ├── upscale_models\esrgan\
│   ├── detection\yolo-detect\
│   ├── llm\qwen35\
│   └── ...
├── browse\
│   ├── by-medium\           ← image / video / audio / 3d / language / utility
│   ├── by-family\           ← the same models, arranged for humans
│   └── unsorted\            ← anything unrecognized, never lost
└── Save\                    ← universal drop zone: anything landing here files itself
```

## Install

Download and run the installer:
**https://pub-542328a810f247e88ace846e2c2a0b52.r2.dev/TensorVillage-Setup-latest.exe**

Per-user, no admin rights for the app itself. It bundles [WinFsp](https://winfsp.dev/)
(the driver behind the drive) and installs it only if it isn't already present — that
one step needs elevation. Windows 10/11, x64. The app updates itself from then on.

## How it works

Four decisions do the work. They're the reason this isn't another folder-scanning GUI.

- **It reads headers, never files.** Tensor names, shapes and `__metadata__` are enough
  to identify almost anything, and they sit in the first few kilobytes. **No tensor data
  is ever read** — so a terabyte scans in seconds, and a 40 GB checkpoint costs the same
  as a 4 MB LoRA.
- **It never unpickles a `.pth`.** Loading a pickle is what executes code — and nearly
  every upscaler, detector and face model in the ecosystem ships as one. So the pickle is
  *walked* instead: each opcode declares its own length, so the names can be read out
  while nothing is constructed and nothing runs. That's what makes that whole half of the
  ecosystem sortable at all, safely.
- **It doesn't trust what a file says it is.** GGUF quantizers routinely stamp the wrong
  architecture — Krea 2 ships as `qwen_image`, ERNIE-Image as `wan`, Chroma and Lens both
  as `flux`, and stable-diffusion.cpp writes no metadata whatsoever. Tensor names survive
  conversion intact, so those get classified on evidence instead of on the label.
- **It's a real filesystem, not a pile of links.** Built on [WinFsp](https://winfsp.dev/):
  the tree lives in memory and every file is served straight from wherever it physically
  sits. That's what lets disks share one namespace — hardlinks can't cross volumes, so
  link-based approaches force your whole library onto a single drive. It also means
  writing to the drive's Save folder works literally, from any program.

Everything else follows from those: nothing is materialized, nothing is duplicated, and
uninstalling leaves your models exactly as they were.

**Diffusers pipelines stay whole.** A diffusers model is a *folder* — `model_index.json`
plus `unet/`, `vae/`, `text_encoder/`. Catalogued file by file it would scatter across
four shelves under names every pipeline shares, so ten models would collide into one
folder of numbered `diffusion_pytorch_model.safetensors` duplicates telling you nothing.
Instead the folder is a single entry keeping its own name, appearing on the drive with its
whole subtree intact and loadable. The pipeline class names the family; where it doesn't,
the weights themselves are read.

**Nothing is chosen for you.** Tensor Village puts nothing on your disks that you didn't
nominate: the only thing in your profile is its own settings and catalog. The save and
trash folders have no defaults, and Save settings tells you what's still needed rather
than inventing somewhere.

## The app

`TensorVillage.Tray` sits in the system tray: it mounts the drive at launch, watches your
source folders, and files any new model automatically within seconds of it landing.
Double-click the icon for the window — **Status**, **Dedupe**, **Metadata**, **Views**,
**Settings**, **Tools**, **Help** and **Support**. Activity is logged to
`%LOCALAPPDATA%\TensorVillage\tray.log`; config and catalog live beside it.

It checks for a new version once a day and offers it in the tray menu; Settings also has
a **Check for updates** button for when you'd rather not wait.

![The Status page](docs/images/status.png)

## The Save and Trash folders

Both are chosen in **Settings** — there are deliberately no defaults, because both decide
where real files of yours end up. Both file their contents into `kind\family` subfolders,
so they still make sense browsed in Explorer, or after you stop using Tensor Village
entirely.

- **Save folder** — where models dropped into the drive's `Save` folder end up. That drop
  zone is the point: because the drive is a real filesystem, you can set it as your
  browser's download folder, drag models in from Explorer or a NAS, point a trainer at it
  as its output folder, or use `tv pull <url>`. Anything that lands is identified and
  filed within seconds — you never decide where a model goes again. Duplicates are handled
  (identical file discarded, same-named different file gets a ` (2)` suffix), and anything
  that isn't a model is left alone. Until a save folder is set there is no `Save` folder on
  the drive at all: a drop zone that can't file anything is worse than none.
- **Trash folder** — where Dedupe moves duplicate copies, and where a model goes when
  you delete it from the drive in Explorer (it comes off every view at once, since they
  were all the same file). Nothing is ever deleted; the space comes back only when you
  empty this folder yourself, which is what makes every removal reversible until you're
  sure. Reachable in one click from the Dedupe page, the
  tray menu, or Settings.

Files already on the same disk as the folder move instantly; ones from another disk are
copied across — slower, but with a library spread over several drives some of that is
unavoidable wherever you put them.

## Views — browse trees of your own

`browse\by-medium` and `browse\by-family` are built in. The **Views** tab adds
your own, and the point of them is what you leave *out*: a tree of everything is just
another copy of the library.

A view is a **layout** plus **filters**:

- **Layout** — the folders to build, from `{kind}` `{family}` `{medium}` `{format}`
  `{year}` `{drive}`. `{drive}\{kind}\{family}` gives you
  `browse\by disk\drive-S\loras\flux\`.
- **Include** — tick only the mediums, kinds or families the view should hold. Nothing
  ticked in a column means all of it, so an empty Include is the whole library.

![The Views tab](docs/images/views.png)

As you edit, the line under the editor keeps count — *"1,843 models in 28 folders"* — so
you see a filter working before you commit to it. Eight presets get you started, each
answering a real question rather than demonstrating syntax:

| Preset | What you get |
|---|---|
| `video work` | everything for making video — models, LoRAs, VAEs, encoders — and nothing else |
| `audio work` | the same for speech and music |
| `loras by base model` | every LoRA under the base model it was trained for |
| `models only` | just the things you generate with; no encoders, adapters or detectors |
| `by disk` | split by the drive each file physically lives on |
| `by year` | this year's models, and the ones you've stopped using |
| `upscalers` | by architecture, the only thing that distinguishes them |
| `loras by format` | kohya / PEFT / LoKR / LoHa, for tools that are fussy |

Presets never filter by family — families differ from library to library, and a preset
that lands empty on someone else's collection teaches them the feature is broken.

Views never hide or move anything: every model is always in `models\` regardless of what
any view says.

## Free — all of it

Every feature, with nothing held back: the drive, classification, auto-filing, the Save
folder, routing rules, ComfyUI wiring, Civitai previews, integrity verification, the
audit tool, and these three as well —

- **Dedupe** — find byte-identical models across every drive and consolidate them safely.
- **Views** — extra browse trees of your own: a layout built from `{kind}` `{family}`
  `{medium}` `{format}` `{year}` `{drive}`, plus filters so a view holds only what you
  want in it (just video, just LoRAs, just the things you generate with). Presets get you
  started, and a live count shows what a view will contain before you save it.
- **Metadata editor** — set title, author, tags and trigger words, and embed preview art
  directly into a model's header (with a pan/zoom crop tool).

No trial, no clock, no account, no telemetry. It never phones home.

If it saved you time and you feel like it, you can
**[buy me a coffee](https://buymeacoffee.com/lorasandlenses)** — entirely optional, and
it unlocks nothing, because there's nothing to unlock.

![The Dedupe page](docs/images/dedupe.png)

*Dedupe compares content, not names: the same model saved twice under different names in
different folders is still one file's worth of disk.*

## Wiring ComfyUI

Tools → **Wire ComfyUI…** (or `tv wire <path>`) adds a managed `tensorvillage:` section
to an install's `extra_model_paths.yaml`, resolving checkpoints, loras, vae, text
encoders, controlnet, clip vision, style models, upscalers and the newer folder keys
(`latent_upscale_models`, `model_patches`, `audio_encoders`, `detection`, `ipadapter`)
through `M:\models`. The path can be the ComfyUI folder, a portable bundle root, or the
yaml itself. The section sits between marker comments, so re-running updates it in place,
the rest of your yaml is untouched, and the first write leaves a `.tv-backup`. Restart
ComfyUI afterwards.

It works best once your models live in one place and you want ComfyUI to look there. If
an install's own models folder is also a Tensor Village source, its pickers will list
those models twice (original path + drive path).

## Choosing a drive letter

**Pick a high one.** Windows gives every drive you plug in the *lowest* free letter it
can find, so a USB stick connected while Tensor Village is closed can quietly take a low
letter and hold it. Letters near the end of the alphabet are almost never claimed that
way, which is why the Settings dropdown lists the highest first and a first run picks the
safest free letter.

If it does happen anyway, nothing is lost. Tensor Village notices on startup and asks
what to do: move itself to a free letter, or move the other drive off yours (Windows
prompts for permission for that one, and it's only offered when it's safe — never the
system drive, a network drive, or a disk holding your model folders). Choosing neither
just leaves the drive unmounted until next launch; the rest of the app carries on.

## Command line

```
tv init <folder> [...]     write config pointing at your model folders
tv scan                    scan + classify into the catalog (cached by size/mtime)
tv status                  show config, catalog and drive state
tv inspect <file>          show one file's header, metadata and classification
                           (safetensors, GGUF or torch — says which and why)
tv route                   file anything sitting in the Save folder
tv pull <url>              download a model into the Save folder
tv wire [comfyui-dir]      point a ComfyUI install at the drive
tv enrich [--limit N]      hash models, fetch Civitai previews + metadata
tv enrich <file ...>       enrich specific models (drive paths accepted)
tv dedupe [--apply]        find identical models; --apply moves copies to trash
tv audit                   flag files that trigger the Windows Explorer bug below
```

The drive itself is mounted by the tray app, so there is no mount/unmount command.

`enrich` writes `<name>.preview.jpeg` and `<name>.civitai.info` next to each model's
real file — the Civitai Helper / ComfyUI-Manager convention — so previews show up both
on the drive and for apps pointed at your original folders. Note that Civitai geo-blocks
its API in some regions (HTTP 451, the UK among them); if every lookup fails that way,
connect through a VPN and rerun. For gated downloads, set a Hugging Face token or
Civitai API key in Settings — the only service credentials the app uses, both optional.

## What it recognizes

Reference, not a promise of completeness — the classifier gains families constantly, and
anything it can't place is still visible in `browse\unsorted`.

**Formats** — safetensors, GGUF, PyTorch saves (`.pth` / `.pt` / `.ckpt`), and diffusers
pipelines (folders with a `model_index.json`).

**Image and video** — SD1.5, SD2, SDXL (+refiner), SD3, Flux and Flux.2 (incl. Klein
4B/9B), Chroma and Chroma Radiance, Qwen-Image (+Layered), Krea 2, Z-Image, Lumina2,
HiDream (+O1), Ideogram 4, Ovis, LongCat, Lens, Mage-Flow, ERNIE-Image, Boogu, PixelDiT,
NewBie, JoyImage, Anima, Cosmos, PixArt, AuraFlow, Stable Cascade, HunyuanDiT and
HunyuanImage; Wan 2.1/2.2 with its derivatives (VACE, S2V, HuMo, Animate, SCAIL,
Fun-Control, I2V, FLF2V…), LTX-Video and LTX-2, HunyuanVideo and 1.5, MiniMax H3,
CogVideoX, Mochi, SVD, SeedVR2, Kandinsky 5.

**Adapters** — LoRA in every dialect (kohya, PEFT, LoKR, LoHa, GLoRA, OFT, BOFT, DoRA,
XLabs), ControlNets and their dozen sub-types, T2I-Adapters, IP-Adapters, style models,
and the newer model-patch category (SUPIR, USO, PuLID, Uni3C, MultiTalk, VACE modules,
LLLite).

**Supporting cast** — VAEs and TAESD approximations, text encoders (37 types, from CLIP-L
to Qwen3-VL, Gemma 4, GPT-OSS and ByT5), CLIP-Vision and DINOv2/v3, embeddings, latent
upscalers.

**Beyond image gen** — upscalers and restorers by architecture (the spandrel set: ESRGAN,
SwinIR, DAT, HAT, ATD, SPAN, RealPLKSR, SCUNet, GFPGAN, CodeFormer and ~40 more), depth
and geometry (Depth Anything v2/v3, MoGe), segmentation and detection (SAM 1/2/3, YOLO,
BiRefNet, RMBG, GroundingDINO, RetinaFace, SegFormer), frame interpolation and optical
flow (RIFE, FILM, GIMM-VFI, RAFT), speech and music (Whisper, Chatterbox, VibeVoice,
F5-TTS, ACE-Step, Stable Audio, RVC, stem separation), and language models by
architecture from llama.cpp's full list.

## What if a model isn't recognized?

It goes to `browse\unsorted` — visible, usable, never lost or renamed. That's the
designed outcome for anything genuinely unidentifiable (cached conditionings, one-tensor
scraps, formats nobody has published a spec for), not a failure state. `tv inspect` on
one shows exactly what was found in it, which is usually enough to say whether it's a
model at all.

## Known issue: the "Digital Signatures" tab crashes Explorer

**A Windows bug that affects safetensors files everywhere, not just here. Your files
are fine.**

A safetensors file starts with its header length as a raw 8-byte little-endian integer.
Whenever that length happens to be ≡ `0x30` (mod 256) — a 1-in-256 chance — the file's
first byte is `0x30`, the ASN.1 `SEQUENCE` marker every certificate begins with.

Windows registers its crypto property-sheet handler (`cryptext.dll`, `CryptoSignMenu`)
for **all** file types and identifies signature files by content-sniffing those leading
bytes. So the file gets a **Digital Signatures** tab in Properties, and opening that tab
makes `crypt32` try to parse gigabytes of tensor data as a certificate chain — hanging,
then crashing Explorer (and Directory Opus, which hosts the same shell extension).

It looks random but isn't: every checkpoint from one training run tends to share a
header size, so entire epoch series are either all affected or all safe.

**What to do:** nothing — just don't open that tab on model files. `tv audit` lists
which of your files carry the unlucky byte. Every other Properties tab is safe.

(A system-wide fix exists — removing `CryptoSignMenu` from
`HKCR\*\shellex\PropertySheetHandlers` — but it also removes the tab for signed
executables, so Tensor Village doesn't touch it.)

## Requests, bugs and unrecognized models

Open an [issue](../../issues). There's a template for **"a model wasn't recognized"** —
run `tv inspect <file>` and paste the output, and I can usually add the family from that
alone. New families land in the next release.

Tensor Village is closed-source: this repo is the download, the changelog and the place
to talk to me. No source lives here.

## Roadmap

Everything-SDK search integration, code signing (to quiet SmartScreen), deeper dedupe
intelligence — precision siblings (fp16 vs fp8 of the same model) and epoch families —
and the `.onnx` and `.bin` corners of the aux-model world.


