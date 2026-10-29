# Architectural Relighting Experiments

**Changing sun direction / time of day in architectural images, keeping the geometry intact.**

Test task for a Junior AI Engineer role — 4 tools, 3 photos, 18 runs, free options only.
Author: Dmytro Trutenko · October 2026

📄 [Full technical report (summary.pdf)](summary.pdf) · [All runs, prompts & settings (tests.pdf)](tests.pdf)

---

## Goal

Change the **direction of lighting** (sun position, time of day) on architectural images while
**preserving the geometry** — roof lines, windows, chimneys, blind slats, objects and the camera
viewpoint must stay exactly where they were. The target use case is a real-estate / architectural
visualization product where a client wants the same building shot under different light.

Success means two things at once, and they turn out to be in tension:

1. the light actually **moves** (shadows change direction, not just colour and brightness);
2. the scene **stays intact** (no invented objects, no cropped frames, no downscaled output).

---

## Tools tested

| Tool | Type | Where it ran | License / cost |
|---|---|---|---|
| **Sun Direction LoRA** (FLUX.2 klein 9B) | LoRA for image-to-image sun control | Hugging Face Space | LoRA open-source w/ commercial use; **base FLUX.2 klein 9B is non-commercial** (klein 4B is not) |
| **Qwen-Image-Edit-2509 Relight LoRA** (dx8152) | Prompt-driven relighting LoRA | ComfyUI Cloud | **Open-source, commercial use supported** (base + LoRA) |
| **IC-Light** | SD1.5 relighting model, text + foreground condition | Hugging Face Space | Repo open-source w/ commercial use; bundled BRIA RMBG 1.4 is non-commercial (swap for BiRefNet) |
| **Relight Spaces** | Commercial web app, 14 fixed lighting presets | Browser | $0.40–0.75 per photo (3 free relights), no API confirmed |

Also surveyed but **not testable**: *Relighting Kontext Dev LoRA v3* (Space paused, copies errored) —
and, for pricing comparison, hosted *fal.ai IC-Light V2* ($0.10/ megapixel) and
*Magnific Relight API* (€0.10 per operation).

Test images ([`images/original/`](images/original)), all Unsplash, chosen to cover different failure modes:

- **Image 1 — old barn exterior**: thatched roof, wooden details, hard shadows from a low autumn sun.
- **Image 2 — modern timber building at sunset**: near-silhouette, strong orange side light; the hardest case.
- **Image 3 — interior with window blinds**: thin repeating blind lines and light patterns on the floor.

Every result was scored 1–5 by eye on **Geometry**, **Material**, **Light change** and **Shadows**,
backed by objective numbers against the original (Canny edge overlap at 5 px tolerance, mean
brightness/saturation shift, output resolution). Edge metrics alone missed regenerated scenes, so
every output was also verified visually.

---

## Results

**Main finding: no tool both moved the sun direction and kept the scene intact.** The only clear
move of the light was Sun Direction LoRA on the interior. Every other result changed colour, sky or
brightness while the shadows stayed exactly where they were.

| Tool | Sun direction moved? | Scene kept? | Time | Output size | Cost |
|---|---|---|---|---|---|
| Sun Direction LoRA | Interior **yes** (6°); weak on barn, building | Mostly; extra chimney, lamps on | 13–17 s | 57–71% of input width | Free |
| Qwen Relight LoRA | **No** (colour & contrast only) | Barn & interior yes; building depends on prompt | 5–44 s | 57–73% of input width | Free tier |
| IC-Light | **No** | Building yes; barn wall greyed | 5–7 s | 512×640 (4:5 crop) | Free |
| Relight Spaces | **No** (time of day via presets) | **Yes, all three photos** | 30–39 s | Same size or larger | $0.40–0.75/photo |

### What worked

- **Qwen Relight LoRA kept geometry and objects best of the open tools.** On the barn and interior:
  no extra chimney (unlike Sun Direction LoRA), lamps stayed off, colours kept (Canny edge recall
  91–92%, precision 99%), and it was the only tool with no licence blocker.
- **Relight Spaces gave the best time-of-day change and preservation.** The sunset building became a
  clear blue-sky day with every object in place, at full size and 1.25× larger output. Geometry 5/5,
  material 4/5 on all three photos.
- **Sun Direction LoRA is the only tool that actually moved light.** On the interior at 6° elevation
  the sun patches moved from the left wall and left floor to the right side of the floor **while
  keeping the thin blind-slat stripe pattern** — the only successful direction change in the whole
  test set. On Image 2 it removed the sunset entirely (mean brightness 78 → 188), reading as midday.
- **IC-Light was the fastest** (5–7 s) and left the building's geometry essentially untouched
  (edge recall 98%, precision 99%).

### What failed, and why

- **Sun Direction LoRA hallucinates and degrades.** On the barn it added a **second white chimney**
  and a roof-like shape behind the ridge in *both* runs with the same seed — so it comes from the
  model, not the sun setting. It also **switched lamps on** in the interior, greyed the colours
  (warm sepia → neutral 115/115/115), and returned 57–71% of the input width. At 36° elevation the
  blind-slat pattern smeared into one soft blob; at 6° it was preserved — **elevation is the key
  parameter**. An unnatural direction ("sun from below") collapsed to flat, dim, diffuse light.
- **Qwen Relight LoRA graded the image instead of moving the sun.** Cast shadows stayed in the
  original places in all runs (99% edge precision = almost no new edges appeared), even when the
  prompt explicitly asked for reversed shadows and the right-hand sun. What changed was saturation
  (95 → 127–137) and brightness. Likely cause: the LoRA is driven by its trigger word **"重新照明"**
  or by its own dataset's prompt style, and the trigger-word node was left empty — so it acted as a
  general colour grade. Prompt strength decided the outcome: **mild prompt kept the golden hour,
  strong midday prompt regenerated the scene** (a whole second house with windows and a snowy roof
  appeared on the left, replacing the distant town, hills and crane; edge precision fell to 42%),
  and a strict "preserve everything" prompt changed almost nothing. No middle ground was found.
- **IC-Light only repainted the background.** On the building it left the orange sunset light on the
  left face untouched and replaced only the sky with a smooth gradient (light on the left, almost
  black on the right, visible vertical banding) — no blue sky, sunset glow still at the horizon, and
  the result looks like a cut-out on a new background. The Space also **centre-crops to 4:5 and
  returns 512×640**, losing ~47% of the barn image width and half the original pixel size. On the
  barn it greyed the white plaster wall, added a crack-like texture and shrank the window.
- **Relight Spaces has no sun-direction control at all** — only fixed presets. The light follows the
  original shadow layout, so it is a time-of-day change, never a direction change. It also
  **reconstructs detail in dark areas** (the silhouetted person became a person in a jacket and
  trousers; snow and neighbour-roof reflections appeared where the original was almost black) —
  plausible, but detail the original did not contain. Edge precision dropped to 17% for that reason.
- **Free demos were unreliable.** Kontext Relight LoRA could not be tested at all, and the Sun
  Direction Space ran out of GPU quota, so the "sun from the right / from behind" cases were never run.

### Selected before/after comparisons

| Original | Result | Verdict |
|---|---|---|
| [Image 1 — barn](images/original/image1-old-barn.jpg) | [Sun Direction, top-left](images/results/sun-direction-lora-flux2/image1-test1-sun-top-left.webp) | Extra chimney, colour drift, weak light change |
| | [Qwen, sun from right](images/results/qwen-relight-lora/image1-test2-sun-right.png) | Geometry 5/5, but shadows unchanged, more saturated |
| | [IC-Light](images/results/ic-light/image1-test1-sun-direction.webp) | Wall greyed, image darker, 4:5 crop at 512×640 |
| | [Relight Spaces, Golden Hour](images/results/relight-spaces/image1-golden-hour.jpg) | Everything kept at full size, warm grade only |
| [Image 2 — building at sunset](images/original/image2-building-sunset.jpg) | [Sun Direction, el. 51°/rot. 114°](images/results/sun-direction-lora-flux2/image2-test1-sun-top-right-el51rot114.webp) | Best of the set: sunset → flat midday, all geometry kept |
| | [Qwen, sun right + midday](images/results/qwen-relight-lora/image2-test2-sun-right-midday.png) | **Scene regenerated** — second house invented, crane/tree/person removed |
| | [Relight Spaces, Wide Awake](images/results/relight-spaces/image2-wide-awake.jpg) | Sunset → clear blue day, all objects kept, 1920×2560 |
| [Image 3 — interior](images/original/image3-interior-blinds.jpg) | [Sun Direction, far left el. 6°](images/results/sun-direction-lora-flux2/image3-test2-sun-far-left-el6.webp) | **Only real direction change** — light moved right, blind stripes kept |
| | [Qwen, beams through blinds](images/results/qwen-relight-lora/image3-test1-sun-left-beams.png) | Scene, colours and lamps preserved, but light not moved |

---

## Conclusion — what to put in production

**Do not ship any tested option as a "sun direction" feature yet.** For a product, split the
problem in two:

**1. Time-of-day presets (solvable now).**
Run **Qwen-Image-Edit-2509 + Relight LoRA on our own GPU** — it is the only open model here with
**no licence blocker** (Apache 2.0-style commercial use; the FLUX bases are non-commercial), it kept
geometry and lamps best on the barn and interior, and it runs in ~15 s per image. Use
**Relight Spaces as the quality benchmark and hosted fallback**, not as a core dependency — it is a
real-estate web tool with no confirmed API. Gate every output automatically: output-size match,
edge overlap against the original, colour consistency and object integrity, with a human review
queue for anything ambiguous.

Before shipping Qwen, run the two untested steps: **start the prompt with the trigger word "重新照明"**
and use the model card's short prompt style, then **pin a fixed output size** (current runs returned
57–73% of the input width and slightly stretched the aspect ratio).

**2. Sun direction itself (R&D, not a feature yet).**
Train a **LoRA on an Apache-licensed base** (FLUX.2 klein 4B or Qwen) using before/after pairs
rendered with known sun positions. The Sun Direction LoRA proves the concept is learnable — it moved
the light exactly once, at low elevation — but the open demos hallucinate structure and degrade
materials. Treat elevation as a first-class parameter: low elevation preserved the blind-slat
structure, high elevation smeared it.

**Cost.** Own GPU wins above roughly **8% utilization (~20 images/hour)**: at ~15 s per image and an
assumed $2/hour GPU that is ~$0.008 per image at full utilization, ~$0.08 at 10% — versus
$0.10–0.75 hosted. Below that, a hosted API is cheaper and simpler.

**Next steps.** (a) Qwen with trigger word + fixed output size; (b) sweep Sun Direction elevation
between 6° and 36°; (c) build the automatic quality gate before any user-facing release.

---

## Repo layout

```
├── README.md
├── summary.pdf                    # 1-page summary of the test task
├── tests.pdf                      # full report: every run, prompt, settings, failure analysis
└── images/
    ├── comfyui-workflow.png       # "Image Relight" ComfyUI workflow used for Qwen
    ├── original/                  # the 3 Unsplash test photos
    └── results/
        ├── sun-direction-lora-flux2/
        ├── qwen-relight-lora/
        ├── ic-light/
        └── relight-spaces/
```

### Reproducing a run

**Qwen Relight LoRA** (ComfyUI Cloud, workflow "Image Relight"):

| Node | Value |
|---|---|
| Base model | `qwen_image_edit_2509_fp8_e4m3fn.safetensors` |
| LoRA | `dx8152/Qwen-Image-Edit-2509-Relight` (strength 1.00) |
| Companion LoRA | `Qwen-Image-Edit-2509-Lightning-4steps-V1.0-bf16` |
| LoRA trigger word | `重新照明` ← **leave empty to reproduce the failures in this report** |
| Seed | random (run counter showed "5 / 5 runs left" on the free tier) |

**Sun Direction LoRA** (HF Space): sun position control (top-left / bottom-left / top-right) plus
optional elevation and rotation; seed 162 for all runs in this report.

---

*Images from Unsplash: @onegrandtrip, @itsandreaswav, @isaax_the_artist.*
