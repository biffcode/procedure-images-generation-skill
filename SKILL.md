---
name: procedure-images-generation
description: Generate medically-accurate procedure cover images for Metaesthetics clinic (aesthetic medicine). Use when creating cover photos/images for a specific aesthetic/medical procedure (injectables, laser, body contouring, LED, PRP, gyneco-aesthetics, etc). Ensures clinical accuracy (correct device/tool per procedure, correct body area, correct visual metaphor) and consistent 1920x1080 studio photography style.
---

# Procedure Images Generation (Metaesthetics)

## Overview
This skill generates **medically-accurate, photorealistic cover images** for aesthetic/medical procedures offered by a Metaesthetics clinic. The #1 rule of this skill: **the image must be clinically correct for the specific procedure** — the right tool, the right body area, the right visual metaphor. A wrong visual (e.g. a laser handpiece for an injectable, or a generic "spa ball" for Botox) is a hard failure even if the image looks beautiful.

This skill works together with:
- `metaesthetics-openrouter` skill — actual image generation engine (fixed studio base + Aggressive Realism prompting), OR
- `nano-banana-imagegen` / `openrouter-image-gen` — fallback generators

Use THIS skill to decide **what the image should contain** (medical research + do/don't rules) — then hand the assembled prompt to the generation skill/script to actually render it.

## Workflow

### Step 1 — Identify the procedure
Get the exact procedure name from the clinic's procedure list (e.g. `Dr_Gueret_Procedures_List.xlsx`). Note its **Category** (Injectables, Skin/Face Treatments, Body Contouring, Minor Procedures/LED, Hair and Scalp, Intimate/Gyneco Aesthetics, Consultation).

### Step 2 — Check existing assets FIRST
Before generating anything, check the clinic's existing photo folder (e.g. `Photos_Metaesthetics_bysophie`) for a usable image matching the procedure. Match by keyword (French/English), not just filename similarity — e.g. "Traitement des masséters" ≈ `saggy.png`/jaw related asset, "Acide hyaluronique" ≈ `injection*.png` or `19_acide_hyaluronique_logomark.png`. Only generate a new image if nothing suitable exists or the existing asset is a logomark/icon rather than a photographic cover.

### Step 3 — Look up the procedure's clinical reference in this skill
Consult `references/procedure_visual_guide.md` (in this skill folder) for the exact DO / DON'T rules for that procedure. If the procedure isn't listed, research it briefly (what tool is used, what body area, is it a needle/laser/device/topical) before writing the prompt — never guess.

### Step 4 — Assemble the Aggressive Realism prompt
Combine:
1. The **fixed studio base** (Phase One IQ4, warm taupe #C9B99A backdrop, butterfly lighting — auto-applied by `metaesthetics-openrouter` skill's script).
2. The **procedure-specific subject/tool/area** from the reference guide.
3. Explicit **exclusions** for that procedure (what must NOT appear).

### Step 5 — Generate at 1920x1080 (NO LOGO)
Always request 16:9 landscape at 1920x1080. **MANDATORY**: You must use the `--no-logo` flag in the generation script. These covers are clean clinical assets. If the generation script defaults to a different resolution, explicitly pass/request 1920x1080 (or upscale/pad to it without distortion — never stretch).

### Step 6 — Organize output
Save each final image inside its own folder named after the procedure (sanitized, no special characters), e.g.:
```
aimis_clinic_procedure_cover/
  generated_covers/
    Toxine_botulique/
      Toxine_botulique_cover.png
    Cryolipolyse/
      Cryolipolyse_cover.png
```
**Note**: Use the original output file from the script, NOT the `_watermarked` version.

### Step 7 — Self-check before delivering
Ask: "If a nurse/doctor looked at this, would they say this is really what this procedure looks like?" If the tool, area, or context is wrong, regenerate — do not deliver.

## Golden Rule Examples (the "Botox ≠ Ball" principle)
- **Toxine botulique (Botox)** → MUST show a small/short fine-gauge needle syringe near the face (forehead/glabella/crow's feet). NEVER a ball, sphere, generic wellness object, or unrelated prop.
- **Acide hyaluronique (filler)** → syringe with clear/viscous gel, injection near lips/cheeks/nasolabial folds, NOT the same needle look as Botox (fillers use slightly larger bore, often shown with cannula option).
- **Cryolipolyse** → a cooling/vacuum applicator paddle on skin (visible frost/condensation optional), NEVER a laser handpiece or syringe.
- **BBL / laser treatments** → a laser/IPL handpiece emitting light on skin, protective eyewear on patient, NEVER a needle.
- **PRP** → a centrifuge tube with separated blood layers (buffy coat) and/or a small syringe with reddish/amber plasma, drawn from arm — NEVER a generic clear serum.
- **LED treatments** → an LED mask or panel emitting colored light (red/blue) on the face or scalp, NO needles, NO lasers.
- **Cryolipolyse/BBL/Mesolift etc. body area MUST match** the named zone (visage=face, cou=neck, décolleté=chest, bras=arms, cuisses=thighs, mains=hands).

See `references/procedure_visual_guide.md` for the full per-procedure breakdown.

## Known Issues & Fixes (learned from sample review — Aug 2026)
- **No logo/watermark**: These procedure cover images must NOT have the Metaesthetics logo applied. Always pass `--no-logo` to `generate_image.py`. Use the plain (non-`_watermarked`) output file.
- **"Meat-like" skin tone**: Over-saturated red/pink/raw-looking skin (looked like meat) appeared on a Cryolipolyse sample. The generation script's base prompt has been updated to require "realistic healthy natural skin color and tone" and explicitly exclude over-saturated red/pink/raw appearance or glossy/waxy sheen. If a result still looks off-color, regenerate and/or add to the prompt: "natural Caucasian/olive/tan skin tone as appropriate, NOT flushed, NOT sunburnt, NOT glossy."
- **Black bars top/bottom ("black square" artifacts)**: The model sometimes bakes in letterbox bars instead of true full-bleed 16:9. The script now auto-detects and crops near-black letterbox rows, then resizes to true 1920x1080. Always verify the delivered image has no black bars before organizing into the final folder — if bars remain, regenerate.

## Resolution & Output Requirements
- Final image: **1920x1080** (16:9), PNG or JPG, full-bleed (no letterbox bars), no logo/watermark.
- One dedicated folder per procedure, named after the sanitized procedure name (spaces→underscores, no accents/special chars if the filesystem is picky, but original folder names with accents are OK on Windows/NTFS).
- File naming inside the folder: `<procedure_name>_cover.png` (add `_v2`, `_v3` for regenerations).

## When a Procedure Isn't in the Reference Guide
1. Identify its **Category** from the procedure list (Injectables / Skin-Face / Body Contouring / Minor-LED / Hair-Scalp / Intimate / Consultation).
2. Identify the **modality** by category convention (Injectables→syringe+needle, Laser/BBL/Vasculaire/Pigmentaire→laser handpiece, Cryolipolyse/Cellulite→body-contouring device, LED→light panel/mask, Consultation→no device).
3. Research briefly (ask the user or reason from medical knowledge) what tool is actually used for that named procedure before writing the prompt.
4. Add the new procedure's DO/DON'T entry to `references/procedure_visual_guide.md` so future generations reuse it (this skill should grow over time).

## Integration with metaesthetics-openrouter
When ready to render, call the `metaesthetics-openrouter` skill's generation script with the assembled variable prompt AND the `--no-logo` flag:
```bash
python C:/Users/bifar/.aios/skills/openrouter-image-gen/scripts/generate_image.py "[VARIABLE PROMPT FROM THIS SKILL'S RESEARCH]" --no-logo
```
The fixed studio base (camera/lighting/backdrop) is auto-applied by that script — this skill only supplies the clinically-correct SUBJECT/TOOL/AREA portion and the exclusions. The script also auto-crops any letterbox bars and forces true 1920x1080 output. Use the plain output filename (not `_watermarked`) since logo is disabled.