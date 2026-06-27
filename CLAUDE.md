# CLAUDE.md

Guidance for working in this repo. Read alongside `README.md` and `DESIGN.md` —
this file is the orientation; those are the source contracts.

## What this is

A **content repo**, not an app. It holds the **Docker** topic (a zero-to-DCA
course) that the `graphl-ux` app (sibling repo) loads **at runtime**. No content
logic, no render engine, and no scenes live here — the app fetches this repo's
notebooks (and, once wired, its `manifest.json`) over the network and renders
them.

There is **nothing to build, run, or test** in this repo. Changes are content and
JSON; correctness is verified by the `graphl-ux` app consuming them. (The one
executable is `scripts/colab_generate_audio.ipynb`, a Colab tool that turns the
`tts/` scripts into `audio/` `.wav`s — see "Narration" below.)

## The core contract (do not break)

1. **The notebook is the single source of truth** for a module's prose and code.
   Any `manifest.json` only *wires* — it must never duplicate notebook content.
2. The app splits each notebook at every `## ` heading into **sections** (= pages).
   Sections are matched to the manifest overlay by **normalized heading text**, so
   a heading edit in a notebook must be mirrored in the manifest `heading` field.
3. A section's diagram **images (`![]()`) are stripped** by the app — a **scene**
   replaces them. Don't rely on inline notebook images surviving.
4. **Scenes live in the app**, authored with the engine's pattern helpers — **not**
   in this repo. The local `scenes/` dir is reserved/empty (`.gitkeep`). Here you
   only reference a scene **by id**.

## The scene

Every Docker module points at one dense scene, **`docker`** — the full
client → daemon stack (`dockerd` → `containerd` → `runc`) → image/container
internals → registry → orchestration map (authored in the app at
`src/data/scenes/docker.ts`). The app pages across a module's sections without
swapping the diagram; each section spotlights the relevant `dk-*` nodes via its
`highlight` list. The node ids live in the scene TS.

## Layout

```
notebooks/      # the 10 teaching .ipynb (prose + code source of truth) — 01..10
tts/            # per-section narration scripts (plain spoken prose)
audio/          # generated .wav narration (per-section)
scenes/         # reserved/empty — real scenes live in the app
scripts/        # colab_generate_audio.ipynb — tts/*.tts -> audio/*.wav on a Colab GPU
DESIGN.md       # FIXED visual house style — palette, calm filled blocks, spotlight
```

## Narration (per-section TTS)

One `.tts` script **per section**, plain spoken prose — what a teacher would say at
a whiteboard. Anchor narration to what's on screen: the notebook `## ` section.
The source curriculum (`~/Projects/docker/tts/`) ships **one `.tts` per notebook**;
here narration is **authored fresh per section**, dropping the intro
("What's covered") and any outro framing — those are not slides.

### TTS guidelines

`.tts` files are read aloud by ChatterboxTTS (typically on a T4 GPU via
`scripts/colab_generate_audio.ipynb`). They must be plain spoken prose.

- **Plain prose only** — no markdown, no `#` headings, no bullets, no backticks, no
  asterisks. Write section titles as a plain sentence ending with a full stop (e.g.
  `What is a container.`).
- **No raw code or shell commands** — describe what a command does in prose.
  `docker run -d --name web -p 8080:80 nginx` becomes "run an n-ginx container in
  detached mode, name it web, and publish host port eighty-eighty to container
  port eighty."
- **Spell out symbols, paths, and shorthand:**
  - Paths: `/var/lib/docker` → "slash var slash lib slash docker",
    `/etc/docker/daemon.json` → "slash e-t-c slash docker slash daemon dot j-son",
    `~/.docker/config.json` → "tilde slash dot docker slash config dot j-son"
  - Image references: `nginx:1.25-alpine` → "n-ginx tag one point twenty-five
    alpine", `myorg/app:latest` → "my-org slash app tag latest"
  - Port and volume syntax: `-p 8080:80` → "publish host port eighty-eighty to
    container port eighty", `-v data:/app` → "mount the data volume into slash app"
  - Operators and flags: `--rm` → "remove on exit", `-d` → "detached", `-it` →
    "interactive with a t-t-y", `--name` → "name flag", `&&` → "and-and", `|` →
    "pipe"
  - Acronyms: OS → "operating system", CLI → "command-line interface", API →
    "ay-pee-eye", CPU → "see-pee-you", RAM → "ram", I/O → "input output", OCI →
    "open container initiative", CNCF → "cloud native computing foundation", CNI →
    "container network interface", DNS → "dee-en-ess", TCP → "tee-see-pee", UDP →
    "you-dee-pee", IP → "eye-pee", VM → "virtual machine", PID → "process I-D",
    UID → "user I-D", GID → "group I-D", TLS → "tee-el-ess", DCA → "dee-see-ay",
    BuildKit → "build kit"
  - Namespaces and cgroups: `PID namespace` → "process I-D namespace", `net
    namespace` → "network namespace", `mnt namespace` → "mount namespace",
    `cgroups` → "see groups", `cgroup v2` → "see group version two"
  - Commands as words: `docker` → "docker", `dockerd` → "docker daemon",
    `docker compose` → "docker compose", `containerd` → "container-d", `runc` →
    "run-see", `buildx` → "build-x"
  - Sizes: `512MB` → "five hundred and twelve megabytes", `1.5GB` → "one point
    five gigabytes"
- **Natural spoken flow** — write as a teacher explains at a whiteboard. Use
  transitional phrases: "notice that", "the key insight here is", "picture this".
- **Skip code outputs and tables** — never read aloud `docker ps` or `docker
  inspect` output. Describe the takeaway instead.
- **Pace with paragraph breaks** — each paragraph = one idea. A blank line between
  paragraphs gives the TTS engine a natural pause. Aim for 2–4 sentences per
  paragraph.

### Naming & generation

- **Naming:** `tts/<NN>-<SS>-<slug>.tts` → `audio/<NN>-<SS>-<slug>.wav`, where `NN`
  is the module number and `SS` the section order within that notebook (the `## `
  heading index, 1-based, where `01` is the dropped "What's covered" intro — so the
  first content section is `02`). The stem is shared by the `.tts`, the `.wav`, and
  the manifest `audio` field, and keeps the Colab glob (`tts/*.tts`, sorted) in
  section order.
- **Generate** with `scripts/colab_generate_audio.ipynb` (ChatterboxTTS, Colab T4):
  one `.wav` per `.tts`, committed + pushed from the Colab VM. See README.

## How content is served

The app fetches this repo at runtime over **raw GitHub**:
`https://raw.githubusercontent.com/schemabotview/docker-content/main/…`
(configurable in the app via `VITE_CONTENT_BASE_URL`). No app build bundles this
content — so a content change is live once pushed to `main`, no app rebuild needed.

## Source of notebooks

Notebooks are copied as-is from the runnable curriculum at `~/Projects/docker`
(the zero-to-DCA course). Keep them self-contained.
