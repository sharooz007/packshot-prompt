---
name: packshot-prompt
description: Run when the user uploads a product image and wants packshot / product-photography PROMPTS (not rendered images). Visually audits the uploaded product, then writes one prompt per composition in a defined composition registry (front packshot, back packshot, side-angle model shot, rear-side model shot, plus any the user adds), and outputs them as a single JSON object. Use whenever the user shares a product photo and asks for packshot prompts, a prompt set, model-shot prompts, or "all the angles" — even without the word "packshot." This skill ONLY writes prompts; it never renders, saves images, or calls an image API. It does not produce video prompts.
---

# Packshot Prompt Set

Prompt-only tool. The user uploads a product image; the job is to produce **one prompt per composition** in the registry below and return them as a single JSON object the user can copy into their own image tools (Nano Banana, Firefly, etc.).

Hard rules:
- **Do not** render images, write files, create folders, or call any API.
- **Never** produce a video prompt.
- **Ask first, then generate.** Before writing any prompts, ask the user which compositions they want via a selectable dialog (multi-select). Generate only the selected ones. Always include an "All" option; if the user picks "All", says "all", or declines to choose, generate the full registry.

---

## The method (philosophy carried over from the original packshot workflow)

Every prompt is built from the same four parts, in this order, concatenated into one continuous string:

1. **Staging lead** — the composition-defining sentence (from the registry).
2. **View-specific structural & material narrative** — a jargon-free description of the garment's materials, hardware, cut, and silhouette, *pointed at the view this composition shows* (front shows the chest logo; back states "no back graphics" if none; 3/4 views emphasise drape and silhouette and add a "preserve exact … from uploaded references" fidelity clause).
3. **Typography line** — `Product Typography and Alignment Syntax: …` listing on-product text exactly, *only if that text is actually visible from this composition's view*. Omit entirely for views where no text shows.
4. **Render-spec tail** — the fixed block, picked by composition type (see Step 4).

This is the same Phase A (staging) / Phase B (text) / Phase C (material narrative) logic as the original — just applied once per composition instead of choosing a single one.

---

## Step 1 — Confirm product, references, and model persona

Look at the uploaded image(s) directly. Then confirm in one short message:

- **Product description** — propose a one-phrase + one-sentence description (e.g. "Short-sleeve heather grey polo with a faded blue denim collar, two-button placket, small embroidered 'FH' chest logo"). Use the user's wording if they gave any.
- **Model persona** (needed for model-editorial compositions) — **default casting: a fair-skinned, European/Western contemporary fashion model in the Zara / H&M campaign mould** (light complexion, clean editorial features, well-groomed, natural styling). Infer gender from the product (menswear → male, womenswear → female) and state the cast inline so the user can change it: e.g. "I'll cast a fair-skinned male model in the Zara/H&M editorial look for the model angles — change that if you want." Specify the look as a positive descriptor in the prompts; do not write negative exclusions (image models follow "make it look like X" far more reliably than "don't make it Y").
- **Brand / output name / aspect ratio** — default `brand: "unknown"`, derive an `output_name` slug from the description, default `aspect_ratio: "3:4"`. Mention the defaults; let the user override.

If no image is uploaded, ask for one — the audit needs the real product. One clean product image works; three to five angles give better fidelity. Worn (on-body) photos are optional — include them only if you want the model shots to match the real as-worn fit; they are used for fit only (see Step 2).

---

## Step 2 — Audit the product (build reusable blocks)

Audit the uploaded image(s) in-context (no API call).

**First, classify every uploaded image into one of two buckets:**
- **Product-reference images** — the garment shown clean and standalone: flat, on a hanger, ghost-mannequin, folded, or any shot where no person is wearing it.
- **Worn images** — the garment actually being worn by a person (on-body, any angle). There may be several; more worn shots = a better read on the fit.

State the split to the user, e.g. "Detected 3 product-reference images and 2 worn images — fit will come from the worn shots, everything else from the clean ones."

Then produce the reusable pieces:

- **Garment type:** identify the product category — tee, polo, shirt, hoodie / hooded sweatshirt, crew sweatshirt, knit, jacket, trousers, etc. This drives **conditional compositions**: comps with a trigger only generate when the trigger is met. The **hoodie pack** (`hoodie_front_model`, `hoodie_front_detail_model`, `hoodie_back_model`, `hoodie_front_flat`, `hoodie_back_detail_flat`, `hoodie_front_detail_flat` — six comps) activates only when the product is a hoodie / hooded sweatshirt.
- **Typography (Phase B):** transcribe every visible inscription/logo/branding string exactly — original capitalisation and spelling — and note its placement. Note which view each string is visible from. **Source: product-reference images only.**
- **Material narrative (Phase C):** an intricate, jargon-free read of materials, hardware, and architectural cut/silhouette. Never mention background, environment, or any human figure in this block. **Source: product-reference images only.**
- **Fit profile (worn images only):** if one or more worn images are present, extract **only** how the garment fits and sits in wear — overall fit class (slim / regular / relaxed / oversized / boxy / cropped), shoulder-seam placement (on-shoulder vs dropped), sleeve length and break on the arm, body length and hem position on the torso/hip, ease and width through chest/waist, how it drapes in motion (clings, structured, billows), neckline behaviour on the body, and for bottoms: rise, leg width, break over footwear, length. Synthesise one coherent profile across multiple worn shots.

**Hard separation — do not cross the streams:**
- The fit profile carries fitting information **only**. Never take colour, material, texture, logo/text, hardware, or construction from a worn image — those always come from the product-reference images, even if the worn shot looks clearer. Worn-photo lighting, the wearer's body, face, pose, and background are all ignored.
- **If no worn image is detected, there is no fit profile — behave exactly as the skill did before** (cut/silhouette read from the clean product photos as usual). This feature only adds behaviour when a worn photo is present; it never changes the default path.

You will re-point the material narrative per view, and inject the fit profile into model-editorial compositions, when assembling each prompt.

---

## Step 3 — Composition registry (this is the editable list)

Generate a prompt for **each** row. To add compositions in future, append rows here (see "Adding a composition").

| key | type | view | staging lead (template) | typography? |
|---|---|---|---|---|
| `front_packshot_prompt` | product-only | front | "Produce a high-end commercial ghost-mannequin packshot, showing the item in a three-dimensional, hollow-body floating state. Front-facing …" | yes, if text visible from front |
| `back_packshot_prompt` | product-only | back | "Produce a high-end commercial ghost-mannequin packshot, showing the item in a three-dimensional, hollow-body floating state. Back-facing …" | only if text visible from back; else state "No back graphics or embroidery present." |
| `flatlay_prompt` | product-only | top-down | "Produce a clean, professional top-down flat-lay packshot, oriented with surgical precision for e-commerce. Item laid flat and squared to frame …" | yes, if text visible from the laid-flat front |
| `folded_prompt` | product-only | folded front | "Produce a stylized streetwear-inspired folded packshot, emphasizing the item's texture and silhouette in a relaxed studio setting. Neatly folded with the front face uppermost …" | only if text remains visible after folding; else omit |
| `side_angle_prompt` | model-editorial | 3/4 front-side | "Produce ONLY a premium 3/4 FRONT-SIDE editorial fashion image featuring a realistic [model persona] wearing [product]. Viewed from a slight front 3/4 angle with the front of the garment clearly visible, subtle natural body turn, relaxed silhouette emphasis, realistic fabric drape, natural folds, premium editorial fashion aesthetic, cinematic premium lighting, realistic cloth physics, and a clean premium minimal environment. Preserve the exact [logo/text], exact proportions, [material] texture, collar/trim variation, and garment dimensions from uploaded references." | yes, if text visible from front |
| `rear_side_angle_prompt` | model-editorial | 3/4 rear-side | "Produce ONLY a premium 3/4 REAR-SIDE editorial fashion image featuring a realistic [model persona] wearing [product]. Viewed from a slight rear 3/4 angle with the back of the garment dominant and clearly visible, slight natural body turn showing partial side profile, emphasis on relaxed silhouette and rear drape, realistic fabric folds, premium editorial fashion aesthetic, cinematic premium lighting, realistic cloth physics, and a clean premium minimal environment. Preserve exact back proportions, [trim] texture, [material] texture, sleeve proportions, and garment fidelity from uploaded references." | only if text visible from back |
| `zara_front_prompt` | model-editorial · **soft-editorial tail** | front | "Produce ONLY a premium Zara-style front-facing editorial fashion image featuring a realistic [model persona] wearing [product]. Three-quarter-length framing from mid-thigh up, model standing in a relaxed natural posture with a calm understated expression and subtle off-centre stance, soft naturalistic daylight, airy minimal editorial mood, realistic fabric drape and cloth physics, premium contemporary fashion-retail aesthetic. **Sleeve rule — resolve at build time:** if the product is a long-sleeve shirt, the sleeves are neatly rolled to mid-forearm with natural creasing (as in the reference); if it is a tee, short-sleeve, or sleeveless garment, leave the sleeves exactly as designed and do not fold or alter them. Preserve the exact [logo/text], exact proportions, [material] texture, collar/placket/trim detail, button finish, and garment dimensions from uploaded references." | yes, if text visible from front |
| `zara_face` → **two prompts** | model-editorial · two tails | tight face crop | "Produce ONLY a premium Zara-style editorial beauty-crop image featuring a realistic [model persona] wearing [product]. Tight head-and-shoulders framing with the model's face dominant, calm understated gaze toward camera or slightly off-axis, natural skin texture and grooming, the garment visible only at the neckline / collar / shoulder. Realistic fabric drape and cloth physics on the visible portion. Preserve the exact neckline/collar detail, [material] texture, and any visible [logo/text] from uploaded references." **Keep all environment/lighting OUT of the lead** — it is supplied entirely by the tail so the two variants differ only in background. Selecting this composition emits **two** JSON keys: `zara_face_studio_prompt` (soft-editorial tail) and `zara_face_beach_prompt` (coastal-daylight tail). | rarely; only if a neck label or text actually shows in the crop |
| `zara_full` → **two prompts** | model-editorial · two tails | full-length head-to-toe | "Produce ONLY a premium Zara-style full-length editorial fashion image featuring a realistic [model persona] wearing [product]. Full-body head-to-toe framing, model standing in a relaxed confident editorial posture (natural weight shift, one hand relaxed or resting on the hip), calm understated expression, realistic fabric drape and cloth physics, premium contemporary fashion-retail aesthetic. **Styling rule — resolve at build time:** build a coordinated tonal outfit around the product's colour — the complementary garment (trousers if the product is a top; a top if the product is bottoms) AND a single carried accessory (a simple bag) should match or closely tonally match the product's colour for a monochrome look. Keep the product itself the unaltered hero piece; the styled pieces are secondary and must not compete with it. **Sleeve rule:** if the product is a long-sleeve shirt, sleeves neatly rolled to mid-forearm; if a tee / short-sleeve / sleeveless garment, leave as designed. Preserve the exact [logo/text], proportions, [material] texture, collar/neckline/trim detail, button finish, and garment dimensions from uploaded references." **Keep all environment/lighting OUT of the lead** — supplied by the tail. Selecting this composition emits **two** JSON keys: `zara_full_studio_prompt` (soft-editorial tail) and `zara_full_beach_prompt` (coastal-daylight tail). | yes, if text visible from front |
| `zara_logo` → **two prompts** | model-editorial · two tails | tight chest / logo crop | "Produce ONLY a premium Zara-style editorial chest-detail image featuring a realistic [model persona] wearing [product], framed as a tight crop on the chest and upper torso so the on-product logo / branding mark is the clear focal point of the composition. The brand mark is rendered crisply and prominently, with the surrounding collar, placket, and fabric texture clearly visible; calm editorial mood, natural relaxed posture, realistic fabric drape and cloth physics. Reproduce the exact logo/emblem — its form, colours, embroidery or print character, scale, and placement — and the [material] texture and collar/placket detail, 1:1 from the product-reference images. **Build-time rule:** only generate this composition if the product has a visible logo, emblem, or branding mark (from the Step 2 audit); if the product has none, SKIP this composition entirely and do not invent a mark — that is an acceptable outcome." **Keep all environment/lighting OUT of the lead** — supplied by the tail. Selecting this composition emits **two** JSON keys: `zara_logo_studio_prompt` (soft-editorial tail) and `zara_logo_beach_prompt` (coastal-daylight tail). | yes — the logo/branding mark is the subject; reproduce it exactly |
| `hoodie_front_model` · *trigger: hoodie* | model-editorial · soft-editorial tail | front, hood down | "Produce ONLY a premium clean e-commerce editorial fashion image featuring a realistic [model persona] wearing [product] with the hood down. Front-facing three-quarter-length framing from mid-thigh up, relaxed natural posture, arms relaxed at the sides, calm understated expression, realistic fabric drape and cloth physics, premium contemporary fashion-retail aesthetic. Reproduce the exact front print/graphic, [material] texture, hood, drawstrings, kangaroo pocket, and cuff/hem ribbing 1:1 from the product-reference images." **Environment/lighting from the tail.** | yes, if front print visible |
| `hoodie_front_detail_model` · *trigger: hoodie* | model-editorial · soft-editorial tail | front-print crop, hands on hood | "Produce ONLY a premium clean editorial detail image featuring a realistic [model persona] wearing [product], cropped tight on the chest and lower face so the front print/graphic is the focal point, both hands raised to grip the hood opening near the chin in a natural styled-lookbook gesture, hood down. Calm editorial mood, realistic cloth physics. Reproduce the exact front print/graphic — imagery, text, colours, scale, placement — and the [material] texture, hood edge, and drawstrings 1:1 from the product-reference images." **Environment/lighting from the tail.** | yes — the front print is the subject |
| `hoodie_back_model` · *trigger: hoodie* | model-editorial · soft-editorial tail | straight-on back | "Produce ONLY a premium clean editorial fashion image featuring a realistic [model persona] wearing [product], viewed straight-on from directly behind, three-quarter-length from mid-thigh up, so the full back graphic/print is dominant and clearly legible, hood resting down at the nape, relaxed natural stance, realistic fabric drape. Reproduce the exact back print/graphic — imagery, text, colours, scale, placement — with the [material] texture, hood, and cuff/hem ribbing 1:1 from the product-reference images." **Environment/lighting from the tail.** | yes — reproduce the back print exactly |
| `hoodie_front_flat` · *trigger: hoodie* | product-only | flat front | "Produce a clean, professional flat product packshot of [product] laid flat and squared to frame, front face up, hood arranged neatly at the top, sleeves laid straight, kangaroo pocket and ribbing visible, e-commerce oriented. Reproduce the exact front print/graphic, [material] texture, and construction 1:1 from the product-reference images." | yes, if front print visible |
| `hoodie_back_detail_flat` · *trigger: hoodie* | product-only | back-print close-up flat | "Produce a clean close-up product image of the back of [product] laid flat, framed tight on the full back print/graphic so it fills the frame with even fabric tension and every element sharp and legible. Reproduce the exact back print/graphic — imagery, text, colours, scale, placement — and the [material] weave 1:1 from the product-reference images." | yes — the back print is the subject |
| `hoodie_front_detail_flat` · *trigger: hoodie* | product-only | front-print close-up flat | "Produce a clean close-up product image of the front of [product] laid flat, framed on the upper chest so the front print/graphic is dominant with the hood and inner neck label visible above it, sharp and legible. Reproduce the exact front print/graphic and the inner neck label exactly as they appear on the product, with the [material] texture 1:1 from the product-reference images." | yes — front print and neck label |

For product-only views the narrative is woven into the lead (after the staging sentence). For model-editorial views the lead already carries view + environment, and the fidelity is locked by the "Preserve exact … from uploaded references" clause.

---

## Step 3b — Ask which compositions are required

Before generating anything, present the registry as a **selectable multi-select dialog** and let the user choose which prompts they want. Use the surface's interactive selection UI (a dialog / checkboxes) where available; otherwise fall back to a numbered list the user replies to.

**Hoodie pre-question (only when a hoodie is detected):** before showing the full menu, ask one yes/no first — "This looks like a hoodie. Show only the hoodie compositions?"
- **Yes** → present only the six hoodie-pack comps for selection.
- **No** → present the full registry (all comps, with the hoodie pack included).
- For any non-hoodie product, skip this question and go straight to the full menu.

- List every current registry composition by its human label (e.g. "Front packshot", "Back packshot", "Flat-lay", "Folded", "3/4 front-side model", "3/4 rear-side model", "Zara front", "Zara face (2 prompts: studio + beach)", "Zara full (2 prompts: studio + beach)", "Zara logo (2 prompts: studio + beach — only if the product has a logo)") with a one-line description each. **Conditional comps appear only when their trigger is met** — list the six-comp hoodie pack ("Hoodie front (model)", "Hoodie front detail (model, hands on hood)", "Hoodie back (model)", "Hoodie front flat", "Hoodie back-print close-up", "Hoodie front-print close-up") only when the product is a hoodie.
- Always offer an **"All"** option at the top.
- If the user picks "All", says "all", or skips the choice, generate the full set on offer (the hoodie pack if they chose hoodie-only, otherwise the full registry).
- Generate **only** the selected compositions — do not add unselected ones, and do not drop selected ones.

This selection step is required every run. The list rebuilds automatically from the registry, so any composition added later appears here as a new option with no extra work.

---

## Step 4 — Render-spec tails (pick by composition type)

**Product-only tail** — append verbatim:

```
Material & Identity Fidelity: Every aspect of the item—its morphology, dimensions, material weight, color saturation, and surface pattern—must be a 1:1 match with the provided source image. Any graphics, hallmarks, or text must be replicated with extreme precision in font, spacing, and positional coordinates. Illumination Profile: Primary lighting is provided by a high-output, evenly diffused softbox array at a 5600k daylight temperature, specifically tuned to lift shadows without losing depth. The environment should be illuminated by a secondary neutral #f9f9f9 light source to ensure a consistent, non-vignetted, clinical grey backdrop without gradients. Render Specs: High-resolution 4k master. Capture every individual fiber, stitch line, and microscopic material texture with edge-to-edge optical clarity. Strict Negative Constraints: No creative interpretation of the original product. Do not render studio hardware, light stands, or power cords. Exclude all human elements, including hands, skin, or mannequins. No props, no dust, and no digital artifacts.
```

**Model-editorial tail** — identical, but the human-exclusion sentence is removed (a model is required, so excluding humans would contradict the shot). Append verbatim:

```
Material & Identity Fidelity: Every aspect of the item—its morphology, dimensions, material weight, color saturation, and surface pattern—must be a 1:1 match with the provided source image. Any graphics, hallmarks, or text must be replicated with extreme precision in font, spacing, and positional coordinates. Illumination Profile: Primary lighting is provided by a high-output, evenly diffused softbox array at a 5600k daylight temperature, specifically tuned to lift shadows without losing depth. The environment should be illuminated by a secondary neutral #f9f9f9 light source to ensure a consistent, non-vignetted, clinical grey backdrop without gradients. Render Specs: High-resolution 4k master. Capture every individual fiber, stitch line, and microscopic material texture with edge-to-edge optical clarity. Strict Negative Constraints: No creative interpretation of the original product. Do not render studio hardware, light stands, or power cords. No props, no dust, and no digital artifacts.
```

**Soft-editorial tail (zara-front only)** — used so this composition does NOT inherit the clinical grey backdrop the other six use. Keeps fidelity and resolution, swaps in soft natural light and a pale neutral editorial wall, no human-exclusion. Append verbatim:

```
Material & Identity Fidelity: Every aspect of the item—its morphology, dimensions, material weight, color saturation, and surface pattern—must be a 1:1 match with the provided source image. Any graphics, hallmarks, or text must be replicated with extreme precision in font, spacing, and positional coordinates. Illumination Profile: Soft, naturalistic diffused daylight at roughly 5600k with a gentle directional quality, producing soft low-contrast shadows. The setting is a pale neutral editorial backdrop—an airy off-white to cool pale-grey wall with subtle natural tonal falloff and faint surface texture, not a flat clinical sweep. Render Specs: High-resolution 4k master. Capture every individual fiber, stitch line, and microscopic material texture with edge-to-edge optical clarity. Strict Negative Constraints: No creative interpretation of the original product. Do not render studio hardware, light stands, or power cords. No added props, no dust, and no digital artifacts.
```

The six original compositions are unchanged — they keep their product-only or clinical-grey model-editorial tails. Only `zara_front_prompt` uses the soft-editorial tail.

**Coastal-daylight tail (zara-face beach variant only)** — outdoor location look matching the reference: bright natural sunlight and a defocused ocean horizon. Append verbatim:

```
Material & Identity Fidelity: Every aspect of the item—its morphology, dimensions, material weight, color saturation, and surface pattern—must be a 1:1 match with the provided source image. Any graphics, hallmarks, or text must be replicated with extreme precision in font, spacing, and positional coordinates. Illumination Profile: Bright natural outdoor daylight in a coastal setting—soft warm sunlight at roughly 5600k, a gently defocused ocean-and-sky horizon behind the subject, light airy high-key atmosphere with natural soft highlights on skin and fabric and subtle ambient bounce. Render Specs: High-resolution 4k master. Capture every individual fiber, stitch line, and microscopic material texture with edge-to-edge optical clarity. Strict Negative Constraints: No creative interpretation of the original product. Do not render studio hardware, light stands, or power cords. No added props, no dust, and no digital artifacts.
```

---

## Step 5 — Assemble each prompt

For every **selected** registry row (from Step 3b), build one continuous string:

`[staging lead with product + view narrative woven in] [Fit clause, if a fit profile exists] [Typography line, if applicable] [render-spec tail for this type]`

Tail selection: product-only rows use the product-only tail; standard model-editorial rows use the model-editorial tail; `zara_front_prompt` uses the soft-editorial tail. A composition may name its own tail in the `type` column (as zara-front does) — honour that over the default. Resolve any build-time rules in the lead (e.g. the zara-front sleeve rule) before assembling.

**Fit clause (only when a fit profile was extracted in Step 2):**
- Inject it into **model-editorial** compositions only (side, rear, zara-front, zara-face, zara-full) — these put the garment on a body, so the as-worn fit matters. Phrase it positively, e.g. "Garment fit as worn: relaxed boxy body, slightly dropped shoulder, sleeves ending mid-bicep, hem sitting at high hip." Product-only compositions (ghost-mannequin, flat-lay, folded) ignore the fit profile and keep their prior behaviour.
- If no fit profile exists (no worn image), omit this clause entirely — every prompt is built exactly as before.

**Fidelity source — important:** the "1:1 match with the provided source image" / "preserve … from uploaded references" language in every lead and tail refers to the **product-reference images** for colour, material, logo, hardware, and construction. The **only** thing sourced from worn images is the fit profile. Never let a worn shot drive colour or material in the output.

**Multi-variant compositions:** `zara_face`, `zara_full`, and `zara_logo` each build the same lead twice and emit two keys — a `…_studio_prompt` (soft-editorial tail) and a `…_beach_prompt` (coastal-daylight tail). The only difference between the two strings is the tail; the lead, styling rules, fit clause, typography, and fidelity clause are identical across the pair. (The hoodie pack is single-variant clean studio — no beach.)

**Conditional compositions:** some comps declare a trigger and generate only when it is met. They are offered in the Step 3b menu only when their trigger is satisfied, and skipped (with a one-line note) if selected without it. Never fabricate content to satisfy a comp.
- `zara_logo` — trigger: the product has a visible logo/emblem/branding mark.
- The six-comp **hoodie pack** (`hoodie_front_model`, `hoodie_front_detail_model`, `hoodie_back_model`, `hoodie_front_flat`, `hoodie_back_detail_flat`, `hoodie_front_detail_flat`) — trigger: the product is a hoodie / hooded sweatshirt.

Keep each value as a single string (no internal line breaks), matching the reference output style.

---

## Step 6 — Output the JSON

Emit one JSON object in a fenced code block. Keys: `output_name`, `brand`, `product_description`, `aspect_ratio`, then **one key per *selected* composition** in registry order. Unselected compositions are omitted entirely (no empty keys). **No `video_prompt`.** Shape (showing all keys; include only the ones selected):

```json
{
  "output_name": "[slug]",
  "brand": "[brand or 'unknown']",
  "product_description": "[full one-sentence description]",
  "aspect_ratio": "3:4",
  "front_packshot_prompt": "…",
  "back_packshot_prompt": "…",
  "flatlay_prompt": "…",
  "folded_prompt": "…",
  "side_angle_prompt": "…",
  "rear_side_angle_prompt": "…",
  "zara_front_prompt": "…",
  "zara_face_studio_prompt": "…",
  "zara_face_beach_prompt": "…",
  "zara_full_studio_prompt": "…",
  "zara_full_beach_prompt": "…",
  "zara_logo_studio_prompt": "…",
  "zara_logo_beach_prompt": "…",
  "hoodie_front_model_prompt": "…",
  "hoodie_front_detail_model_prompt": "…",
  "hoodie_back_model_prompt": "…",
  "hoodie_front_flat_prompt": "…",
  "hoodie_back_detail_flat_prompt": "…",
  "hoodie_front_detail_flat_prompt": "…"
}
```

(Additional composition keys appear here automatically as they're added to the registry.)

After the block, in plain text: list which compositions you generated, and offer to add a new one or adjust the model persona / aspect ratio.

---

## Adding a composition

When the user wants a new shot type (e.g. "add a top-down flat-lay", "add a macro collar-stitch detail", "add it hanging on a rail"):

1. Decide its **type**: `product-only` (no human) or `model-editorial` (a model wears it).
2. Write a **staging lead** in the same voice as the existing rows — a single decisive sentence naming the framing and view, followed by the view-pointed material/silhouette emphasis. For product-only shots, open with the appropriate archetype verb ("Produce a … packshot"). For model shots, open with "Produce ONLY a premium … editorial fashion image featuring a realistic [model persona] wearing [product]…" and end with a "Preserve exact … from uploaded references" clause.
3. Set **typography?**: include the typography line only if the product's text is visible from that view.
4. Append a row to the registry table and generate its prompt with the matching render-spec tail.

Examples of valid additions (each becomes its own JSON key, e.g. `macro_detail_prompt`, `hanger_prompt`):
- **Macro detail** — product-only, view = a named region; narrate the stitch/weave/hardware of that region only.
- **Hanger / rail** — product-only: "Produce a clean commercial hanger packshot, the item hung on a minimal studio rail, natural vertical drape …"
- **Lifestyle model, full-length** — model-editorial: front-on full-body editorial, same "Preserve exact … from uploaded references" lock.

Keep the same philosophy: faithful 1:1 reproduction of the real product, the render-spec tail always appended, human-exclusion present for product-only and absent for model shots.

---

## Notes

- All prompt writing is in-context — no external API, no file writes, no rendering.
- More reference angles = better material/text fidelity. One is enough; three to five is ideal.
- Ask which compositions are wanted (Step 3b) before generating; generate exactly the selected set, no more, no less.
- Worn (on-body) photos are a **fit-only** source: they shape the as-worn fit clause on model shots and nothing else. No worn photo → default behaviour, unchanged. Never source colour/material/logo from a worn shot.
- If on-product text matters, remind the user that text rendering varies between image models and may need a couple of passes in their own tool.
- This skill stops at the prompts. Rendering and saving under a brand folder is the separate automated `packshot` skill.
