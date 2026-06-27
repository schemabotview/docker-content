# docker-content

Content repo for the **Docker** topic in graphl-ux. Designed to load at runtime;
no content logic lives in the app code. Follows the **manifest +
notebook-as-source-of-truth** contract.

## Layout

```
docker-content/
  notebooks/        # the 10 teaching .ipynb (the prose + code source of truth)
  tts/              # per-section narration scripts (plain spoken prose)
  audio/            # generated .wav narration
  scenes/           # reserved — scenes live in the app (src/data/scenes), not here
  scripts/          # colab_generate_audio.ipynb — TTS .tts -> .wav on a Colab GPU
  DESIGN.md         # the visual house style (calm filled blocks, palette, reel chrome)
```

## The course

A zero-to-DCA Docker curriculum in **10 notebooks**, each strictly assuming only
what came before:

| # | Topic |
|---|---|
| 01 | Getting Started with Docker |
| 02 | Images, Layers & the Dockerfile |
| 03 | Running & Inspecting Containers |
| 04 | Storage, Volumes & Bind Mounts |
| 05 | Networking & Port Publishing |
| 06 | Docker Compose & Multi-Container Apps |
| 07 | Registries, Tags & Distribution |
| 08 | Security, Secrets & Hardening |
| 09 | Orchestration with Swarm |
| 10 | Install, Configure, Troubleshoot & DCA Prep |

## Generating audio

Narration `.wav`s are produced from the `tts/*.tts` scripts by
`scripts/colab_generate_audio.ipynb` (ChatterboxTTS on a Colab T4 GPU). It clones
this repo, generates one `audio/<stem>.wav` per `tts/<stem>.tts`, and commits +
pushes each clip straight from the Colab VM. Needs a Colab secret `GITHUB_TOKEN`
(a PAT with **Contents: Read/Write** on `schemabotview/docker-content`). Set
`FORCE=True` to regenerate existing clips, or `ONLY_STEM="01-02-what-is-a-container"`
to do a single section. This is a convenience tool — there is otherwise nothing to
build or run in this repo.

## Contract

- The **notebook is the single source of truth** for a module's prose and code.
  The manifest only *wires* — it never duplicates notebook content.
- The app splits each notebook at every `## ` heading into **sections** (= pages).
  A section's diagram **images (`![]()`) are stripped** — the scene replaces them.
- The manifest overlay attaches, per section: a `scene` id (the diagram), a
  `spine` flag (drives feed-mode flow), an optional `role` (e.g. `hook`), an
  optional `highlight` (scene node ids that light AMBER), and an `audio` stem.
  Sections are matched to the overlay by normalized heading.
- A **scene** is a reusable spec (portrait 800×1200) referenced by id from many
  sections. Scenes are authored and bundled in the **app** (`src/data/scenes`,
  with the engine's pattern helpers) — they are **not** served from this repo.
  Styled per `DESIGN.md`.

## The scene

Every Docker module points at one dense scene, **`docker`** — the full
client → daemon stack → image/container internals → registry → orchestration map
(authored in the app at `src/data/scenes/docker.ts`). The app pages across a
module's sections without swapping the diagram; each section spotlights the
relevant `dk-*` nodes via its `highlight` list.

## Source

Notebooks are copied as-is from the runnable curriculum at `~/Projects/docker`
(the zero-to-DCA course). Keep them self-contained. The source curriculum ships
one `.tts` per notebook; here narration is **authored fresh per `## ` section** to
align with the app's pages.

## Serving

graphl-ux fetches this repo at runtime over raw GitHub
(`https://raw.githubusercontent.com/schemabotview/docker-content/main/…`,
configurable via the app's `VITE_CONTENT_BASE_URL`). No app build bundles this
content; the app ships only the render engine + scenes.
