<div align="center">

# 📸 Packshot-Prompt

**Studio-Grade E-Commerce & Zara-Style Fashion Editorial Prompt Engine for AI Agents**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-Antigravity%20%7C%20Claude%20%7C%20Codex-brightgreen)](SKILL.md)
[![Compatible](https://img.shields.io/badge/Compatible-Flux%20%7C%20Midjourney%20%7C%20Firefly%20%7C%20Imagen-orange)](#target-image-models)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-purple.svg)](https://github.com/sharooz007/packshot-prompt/pulls)

*Turn any raw product or garment photo into surgical, multi-angle commercial packshot and high-fashion editorial prompts.*

[Quickstart](#installation--setup) • [The 4-Part Anatomy](#-the-4-part-prompt-architecture) • [Composition Registry](#-the-composition-registry) • [Zara Editorial](#-the-zara-editorial-collection) • [Sample JSON](examples/sample-prompt-set.json) • [Launch Kit](LAUNCH_KIT.md)

---

</div>

## 💥 The Problem with AI Product Photography

Generating consistent, e-commerce-ready product photography with AI is notoriously difficult:

- ❌ **Angle Drift:** Generating different angles (front, back, 3/4 side) usually produces completely different garments, colors, or fabrics.
- ❌ **Hallucinated Branding:** AI image models mangle text, invent fake chest logos, or drop real typography.
- ❌ **Unrealistic Lighting & Floating Artifacts:** Inconsistent studio lighting, harsh shadows, weird mannequins, and studio equipment cluttering the frame.
- ❌ **Poor Fit Translation:** AI either renders clothes looking like cardboard or drapes them with distorted proportions.

---

## ⚡ The Solution: Packshot-Prompt

**Packshot-Prompt** is an advanced prompt-engineering skill for AI coding and design agents. 

When you upload photos of a garment, the skill performs a **multimodal visual audit** of the piece and outputs a **comprehensive JSON dictionary** containing one hyper-engineered prompt per camera angle and staging style. 

You can immediately paste these prompts into **Flux**, **Midjourney**, **Nano Banana**, **Adobe Firefly**, or **Google Imagen** for flawless, catalog-ready assets.

```
       ┌────────────────────────┐
       │ Upload Product Photos  │ (1-5 reference shots + optional worn photos)
       └───────────┬────────────┘
                   │
                   ▼
       ┌────────────────────────┐
       │ Multimodal Visual Audit│ ──► Extracts: Cut, Stitch, Hardware, 
       │   (Separation Engine)  │      Materials & Exact Typography
       └───────────┬────────────┘
                   │
                   ▼
       ┌────────────────────────┐
       │  Composition Selection │ ──► Interactive menu: Choose Packshots, 
       │   (Registry Dispatch)  │      Model 3/4s, Zara Editorials, or Hoodie Pack
       └───────────┬────────────┘
                   │
                   ▼
       ┌────────────────────────┐
       │ Structured JSON Export │ ──► Ready-to-render prompts with 
       │    (Ready for Flux)    │      1:1 material fidelity & 5600K lighting
       └────────────────────────┘
```

---

## 🧩 The 4-Part Prompt Architecture

Every generated prompt is mathematically assembled from four coordinated sections concatenated into a single cohesive string:

```
┌─────────────────┐   ┌───────────────────────────┐   ┌───────────────────────┐   ┌──────────────────┐
│ 1. Staging Lead │ + │ 2. Structural & Material  │ + │ 3. Typography Line    │ + │ 4. Render-Spec   │
│                 │   │    Narrative (View-Bound) │   │    (Exact Placement)  │   │    Tail          │
└─────────────────┘   └───────────────────────────┘   └───────────────────────┘   └──────────────────┘
```

1. **Staging Lead:** Decisive composition-defining instruction (ghost-mannequin, flat-lay, 3/4 fashion model, or Zara beauty crop).
2. **Structural & Material Narrative:** Jargon-free description of the garment’s weave, drape, hardware, seams, and silhouette—specifically tailored to what is visible from that exact camera angle.
3. **Exact Typography Line:** `Product Typography and Alignment Syntax: ...` reproducing visible inscriptions and logos with exact spelling, capitalization, and coordinates.
4. **Render-Spec Tail:** Calibrated optical instruction block enforcing 5600K daylight, #f9f9f9 shadow-lifting studio diffusion, 4K microscopic texture capture, and strict negative constraints (no stands, no cords, no human artifacts for packshots).

---

## 🗂️ The Composition Registry

The skill features a built-in modular composition registry:

| Key | Type | Framing / View | Purpose |
| :--- | :--- | :--- | :--- |
| `front_packshot_prompt` | Product-only | Ghost Mannequin Front | Hollow-body floating 3D e-commerce hero shot |
| `back_packshot_prompt` | Product-only | Ghost Mannequin Back | Pure back view highlighting yoke, rear seams, or prints |
| `flatlay_prompt` | Product-only | Surgical Top-Down | Clean flat-lay for web stores and lookbooks |
| `folded_prompt` | Product-only | Relaxed Streetwear Fold | Folded presentation highlighting fabric weight and depth |
| `side_angle_prompt` | Model-editorial | 3/4 Front-Side on Body | Dynamic drape, shoulder drop, and silhouette |
| `rear_side_angle_prompt`| Model-editorial | 3/4 Rear-Side on Body | Back silhouette, rear drape, and arm proportions |

---

## 🧥 The Zara Editorial Collection

For elevated brand campaigns, the skill includes specialized Zara-style high-fashion editorial prompts:

* **`zara_front_prompt`**: 3/4 length mid-thigh crop with naturalistic daylight, off-white editorial wall, and intelligent sleeve handling (auto-rolls long sleeves to mid-forearm).
* **`zara_face_studio_prompt` & `zara_face_beach_prompt`**: Tight beauty crop focusing on the model's face and neckline collar construction (available in soft studio neutral or coastal high-key daylight).
* **`zara_full_studio_prompt` & `zara_full_beach_prompt`**: Head-to-toe full-body editorial with an auto-styled tonal monochrome companion outfit and accessories.
* **`zara_logo_studio_prompt` & `zara_logo_beach_prompt`**: Tight macro chest detail isolating the brand mark and placket texture (only generated if a logo is present).

### ⚡ Conditional "Hoodie Pack"
When a hoodie or hooded sweatshirt is detected, the skill automatically unlocks a 6-composition specialized pack:
- `hoodie_front_model`: Three-quarter model shot with hood down
- `hoodie_front_detail_model`: Chest detail with hands gripping hood opening in natural lookbook gesture
- `hoodie_back_model`: Dominant back-print model shot
- `hoodie_front_flat`: Squared flat-lay with hood and kangaroo pocket
- `hoodie_back_detail_flat`: Macro close-up on full back graphic
- `hoodie_front_detail_flat`: Upper chest close-up with inner neck label

---

## 🔬 Multimodal Separation Engine: Product vs. Worn

One of Packshot-Prompt's most powerful capabilities is its **hard separation of references**:

- **Product-Reference Photos:** Drives color, material texture, hardware, stitch lines, and logo typography.
- **Worn (On-Body) Photos:** Strictly drives the **Fit Profile** (shoulder seam drop, body length, boxiness, arm break). 
- *The rules never cross:* Lighting, skin tones, or wearer details from worn photos are completely excluded from the product's material identity.

---

## 🎯 Target Image Models

The generated prompts are optimized for and tested across:
- **Black Forest Labs Flux (Flux.1 Pro / Dev / Schnell)**
- **Midjourney v6 & v6.1**
- **Adobe Firefly Image 3**
- **Google Imagen 3 (Gemini)**
- **Stable Diffusion 3.5 & SDXL**
- **Nano Banana**

---

## 🚀 Installation & Setup

### 1. Google Antigravity / Gemini CLI
Clone or copy the skill into your skills directory:

```bash
# Global configuration (available in all workspaces)
git clone https://github.com/sharooz007/packshot-prompt.git ~/.gemini/config/skills/packshot-prompt

# Project-specific configuration
mkdir -p .gemini/skills
git clone https://github.com/sharooz007/packshot-prompt.git .gemini/skills/packshot-prompt
```

### 2. Claude Code
Install to your Claude Code skills or rules directory:

```bash
mkdir -p ~/.claude/skills/packshot-prompt
cp /path/to/packshot-prompt/SKILL.md ~/.claude/skills/packshot-prompt/
```

### 3. Cursor / Windsurf / Codex
Copy the instructions in [`SKILL.md`](SKILL.md) directly into your workspace instructions or agent system prompt.

---

## 💡 Usage

Simply attach an image of your clothing item or product and run:

```text
/packshot-prompt Generate prompts for all angles of this polo shirt.
```

Or naturally in prompt:
```text
I want packshot prompts for this uploaded hoodie. Give me ghost mannequin, flat-lay, and Zara model shots.
```

The agent will audit the images, present an interactive composition selection menu, and deliver the final JSON prompt suite.

---

## 📄 Output Schema Example

```json
{
  "output_name": "vintage-washed-oversized-hoodie",
  "brand": "Acme Studios",
  "product_description": "Heavyweight 450gsm loopback cotton French terry hoodie in washed vintage charcoal with dropped shoulders and distressed chest embroidery.",
  "aspect_ratio": "3:4",
  "front_packshot_prompt": "Produce a high-end commercial ghost-mannequin packshot...",
  "back_packshot_prompt": "Produce a high-end commercial ghost-mannequin packshot...",
  "flatlay_prompt": "Produce a clean, professional top-down flat-lay packshot...",
  "side_angle_prompt": "Produce ONLY a premium 3/4 FRONT-SIDE editorial fashion image...",
  "zara_front_prompt": "Produce ONLY a premium Zara-style front-facing editorial fashion image..."
}
```

Check out [`examples/sample-prompt-set.json`](examples/sample-prompt-set.json) for a complete sample output.

---

## 🤝 Contributing

Contributions, new composition styles (e.g. footwear, eyewear, jewelry packs), and model tweaks are warmly welcomed!
1. Fork the repo
2. Create your feature branch (`git checkout -b feature/sneaker-pack`)
3. Commit your changes (`git commit -m 'Add footwear packshot composition templates'`)
4. Push to origin (`git push origin feature/sneaker-pack`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

Crafted with care by [Sharooz](https://github.com/sharooz007).
