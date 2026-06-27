# Visual design — tuned for retention on mobile + Udemy video

Two delivery targets: a **mobile reel feed** and **recorded Udemy lessons**. Both
are served by the same dark, high-contrast, semantic-color system below. This is
the Docker topic's house style; it mirrors the shared graphl-ux look.

## Style: calm filled blocks (NOT neon)

The diagram reads like a clean reference figure, not a glowing dashboard. Color
lives in a **muted filled block**, not a neon outline. Rules:

- **No glow** (no `box-shadow`), **no backdrop-blur**, **no dotted grid**.
- Nodes are **filled** with the role hue mixed into a dark base (`#14161c`) at
  ~30% — a desaturated block, with a thin solid border one step brighter. White
  bold title; sub-caption a soft tint of the role color.
- **Edges are plain gray (`#5b6270`), thin**, with a **small slow gray flow dot**
  (2.4s) as a gentle directional cue — calm, not neon. Brighter color/motion is
  *reserved* for the narration spotlight, so the spotlight still stands apart.
  Keep edge count low — prefer 2 clean arrows over many crossing ones.

## Background

- Scene canvas + app: **`#16181d`** (flat neutral dark — no blue tint, no grid;
  avoids OLED smear / video banding; portrait pillarbox bars vanish into it).
- Code panel: **`#0d1117`** (github-dark, matches highlight.js theme).

## Semantic palette (FIXED roles — do not reuse a color for an unrelated concept)

The Docker scene is a topology of layers — the user-facing CLI, the daemon stack,
image and container internals, the registry, and orchestration. Each band has a
role color. The map below is what the `docker` scene encodes
(`src/data/scenes/docker.ts`).

| Token  | Hex       | Always means…                                                       |
| ------ | --------- | ------------------------------------------------------------------- |
| BLUE   | `#5b8cff` | **Client surface** — the `docker` CLI verbs (run, build, pull, ps…) |
| ORANGE | `#ff7a59` | **Docker Host** — the daemon stack (`dockerd`/`containerd`/`runc`), images, layer model |
| YELLOW | `#f5c542` | **Authoring & build** — the Dockerfile instructions, BuildKit, build cache |
| GREEN  | `#37d39a` | **Container lifecycle** — create, start, stop, rm                   |
| RED    | `#ff5d6c` | **Isolation boundaries** — namespaces, cgroups, capabilities, security hardening |
| TEAL   | `#3fd0d6` | **Resources attached to a container** — volumes, networks, ports    |
| PURPLE | `#b98bff` | **Off-box & cluster** — registry (push/pull/identity), Compose, Swarm |
| GRAY   | `#9aa3b2` | **Inert / context** — the spine wrappers, structural grouping       |

Color is a *memory cue*: a learner who sees the daemon=orange and isolation=red
every time encodes it faster. Keep ≤4–5 colors live on screen at once; push
everything supporting to GRAY so the colored nodes actually *mean* something.

## Spotlight (the video attention lever)

- **AMBER `#ffc24b`** is reserved for the **node(s) the narration is on right now**
  — brighter glow + the animated flow-in-path edge converging on them. A section's
  manifest `highlight` list names those `dk-*` node ids; the rest of the scene dims
  back. Spotlighting a container also lights its children. Omit `highlight` to show
  the scene full-strength (e.g. the module hook).
- Nothing else uses amber. Motion + a reserved highlight color drives the eye to
  exactly the concept being spoken — the biggest retention win in video/feed.

## Typography (legibility on small screens + after compression)

- Labels **bold, large** (≥ ~18px equiv at the 800×1200 scale); avoid thin weights
  — compression eats them.
- `term` chips (the label *is* the concept) use mono + a filled tint of the role
  color.

## Motion

- Edges animate a dot along the path (`flow-in-path`) to show direction/sequence.
- Reveal spine nodes in narration order; depth nodes appear with the panel.
- Keep it calm — one thing moving at a time, paced to the narration's 300 ms
  pauses (blank lines in the `.tts`).
