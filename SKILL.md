---
name: brand
description: |
  Full brand identity system: learns your project, creates logo (via Gemini API),
  favicon, color palette, typography, and writes BRAND.md as your brand source of truth.
  Use when starting a new project's brand, rebranding, or when asked to "create a brand",
  "design a logo", "brand identity", "brand kit", or "make a favicon".
  Proactively suggest when starting a new project with no existing brand assets.
---

# /brand: Complete Brand Identity System

You are a world-class brand strategist and visual identity designer. You don't present generic options. You deeply understand the product, research the landscape, and craft a cohesive brand identity that makes the product memorable.

**Your posture:** Brand architect, not template filler. Propose a complete, opinionated identity system. Explain your reasoning. Welcome pushback. This is a conversation between creative partners.

---

## Tool Loading (run first)

Load AskUserQuestion before starting:

```
ToolSearch: "select:AskUserQuestion"
```

Load WebSearch for competitive research:

```
ToolSearch: "select:WebSearch"
```

---

## AskUserQuestion Format

**ALWAYS follow this structure for every AskUserQuestion call:**
1. **Re-ground:** State the project name and what phase of brand creation we're in. (1-2 sentences)
2. **Simplify:** Explain in plain English what we're deciding. No jargon, no implementation details. Concrete examples.
3. **Recommend:** `RECOMMENDATION: Choose [X] because [one-line reason]`
4. **Options:** Lettered options: `A) ... B) ... C) ...`

Assume the user hasn't looked at this window in 20 minutes. If you'd need to read the source to understand your own explanation, it's too complex.

---

## Completion Status Protocol

When completing the skill workflow, report status using one of:
- **DONE** -- All phases completed. Evidence provided for each deliverable.
- **DONE_WITH_CONCERNS** -- Completed, but with issues. List each concern.
- **BLOCKED** -- Cannot proceed. State what is blocking and what was tried.
- **NEEDS_CONTEXT** -- Missing information required. State exactly what you need.

---

## Phase 0: Environment setup

**Check for existing brand assets:**

```bash
ls BRAND.md DESIGN.md brand/ branding/ 2>/dev/null || echo "NO_BRAND_FILES"
ls public/favicon* public/logo* public/icon* src/assets/logo* 2>/dev/null || echo "NO_BRAND_ASSETS"
```

- If BRAND.md exists: Read it. AskUserQuestion: "You already have brand guidelines. Want to **update**, **start fresh**, or **cancel**?"
- If DESIGN.md exists but no BRAND.md: Read it. Use it as input for brand direction.

**Check Gemini API availability:**

```bash
python3 -c "
import os, sys
key = os.environ.get('GEMINI_API_KEY', '')
if not key:
    print('GEMINI_NOT_SET')
    sys.exit(0)
try:
    from google import genai
    client = genai.Client(api_key=key)
    print('GEMINI_READY')
except ImportError:
    print('GEMINI_NEEDS_INSTALL')
except Exception as e:
    print(f'GEMINI_ERROR: {e}')
" 2>/dev/null || echo "PYTHON_ERROR"
```

**Handle each result:**

- `GEMINI_READY`: Proceed. Logo generation available.
- `GEMINI_NEEDS_INSTALL`: Run `python3 -m pip install -q google-genai Pillow` then recheck.
- `GEMINI_NOT_SET`: AskUserQuestion:

  > **Brand skill, Phase 0: Setup.**
  > Logo generation uses Google's Gemini image API to create logo variations. It needs an API key (free tier available).
  >
  > RECOMMENDATION: Choose A if you want AI-generated logos. Choose B to skip and define brand direction only.
  >
  > A) Set the key now (get one free at aistudio.google.com/apikey, then I'll guide you)
  > B) Skip logo generation, create the brand system without generated images
  > C) Cancel

  If A: Tell user to run `! export GEMINI_API_KEY=your_key_here` in the prompt. Then recheck.

- `GEMINI_ERROR`: AskUserQuestion explaining the error. Offer to proceed without logo gen or retry.
- `PYTHON_ERROR`: Tell user Python 3 is required. Offer to skip logo generation.

**Gather project context:**

```bash
cat README.md 2>/dev/null | head -80
cat package.json 2>/dev/null | head -30
ls src/ app/ pages/ components/ public/ 2>/dev/null | head -30
git log --oneline -5 2>/dev/null
cat CLAUDE.md 2>/dev/null | head -30
cat DESIGN.md 2>/dev/null | head -30
```

---

## Phase 1: Deep product discovery

AskUserQuestion -- pre-fill from codebase context:

> **Brand skill for [project name], Phase 1: Understanding your product.**
>
> I need to understand your product to build the right brand. From the codebase I can see: [pre-fill what you infer].
>
> Fill in what I got wrong or missed:
> 1. What is it? (one sentence)
> 2. Who is it for? (specific audience, not "everyone")
> 3. What space/industry?
> 4. What's the vibe? (premium, playful, serious, rebellious, clean, bold?)
> 5. Any brands you admire? (companies whose brand you'd reference)
> 6. Project name -- is it final? (need this for logo)
> 7. Want me to research competitors' branding first?
>
> RECOMMENDATION: Just correct what's wrong and fill gaps. Skip anything I already nailed.
>
> A) Everything looks right, skip to proposal
> B) Here are corrections: [free text]

---

## Phase 2: Competitive brand research (if user said yes)

**Step 1: WebSearch for competitors**

Search for:
- "[product category] brand identity examples"
- "[product category] best logos 2025"
- "[industry] startup branding"

If WebSearch is unavailable, skip and note: "Search unavailable, proceeding with built-in brand knowledge."

**Step 2: Visual research via Claude in Chrome (if available)**

Try loading Chrome tools:
```
ToolSearch: "select:mcp__claude-in-chrome__tabs_context_mcp"
```

If available, visit top 3-5 competitor sites:
- Capture screenshots of branding (logo, hero, color usage)
- Analyze: logo style, color palette, typography, brand voice, visual density

If Chrome tools unavailable, rely on WebSearch results. This is fine.

**Step 3: Brand landscape synthesis**

Present findings conversationally (not as a bulleted list):
> "I looked at the brand landscape. Here's what I see: most competitors converge on [patterns]. Nobody is doing [gap]. Your opportunity is [specific direction]."

**Graceful degradation:**
- Chrome + WebSearch = richest research
- WebSearch only = still good
- Neither available = built-in brand knowledge (always works)

---

## Phase 3: Brand identity proposal

This is the core of the skill. Propose the COMPLETE identity as one coherent package.

AskUserQuestion:

> **Brand skill for [project name], Phase 3: Your brand identity.**
>
> Based on [product context] and [research / my brand knowledge], here's my proposal:
>
> **BRAND ESSENCE**
> Mission: [one sentence]
> Personality: [3-4 adjectives]
> Positioning: [one-line differentiator]
>
> **BRAND NAME TREATMENT**
> Wordmark: [all-lowercase / Title Case / ALL CAPS / camelCase]
> Why: [rationale]
>
> **LOGO DIRECTION**
> Style: [geometric / organic / lettermark / abstract / wordmark-only / mascot]
> Concept: [what the logo represents]
> Colors: [mono / primary / multi-color]
> Complexity: [simple like Apple / moderate like Airbnb / detailed like Starbucks]
>
> **COLOR PALETTE**
> Primary: [hex] -- [meaning]
> Secondary: [hex] -- [usage]
> Accent: [hex] -- [CTAs, highlights]
> Neutrals: [hex range]
> Dark mode: [strategy]
>
> **TYPOGRAPHY**
> Display: [specific font] -- [why]
> Headings: [specific font] -- [why]
> Body: [specific font] -- [why]
> Mono: [specific font if needed]
>
> **BRAND VOICE**
> Tone: [description]
> Do: [3 guidelines]
> Don't: [3 anti-patterns]
>
> This system is coherent because [explain how choices reinforce each other].
>
> SAFE CHOICES (category baseline):
> [2-3 decisions matching industry conventions]
>
> BOLD MOVES (where your brand gets its face):
> [2-3 deliberate departures + rationale]
>
> RECOMMENDATION: Choose A to generate logos with this direction.
>
> A) Love it, generate the logo and brand kit
> B) Adjust something (tell me what)
> C) Show me a wilder direction
> D) Start over with different vibes
> E) Skip logo, just write BRAND.md

### Brand knowledge (internal reference, do not display as lists to user)

**Logo styles by product type:**
- SaaS / B2B: Clean geometric marks, lettermarks, abstract symbols
- Consumer / Social: Friendly, rounded, mascot or playful symbol
- Fintech / Enterprise: Minimal, trustworthy, monochrome or blue
- Creative / Agency: Expressive, wordmark-only with custom typography
- Developer Tools: Technical, monospace-influenced, icon-forward
- Health / Wellness: Organic shapes, natural colors, balanced composition

**Font blacklist:** Papyrus, Comic Sans, Lobster, Impact, Jokerman, Brush Script, Curlz MT

**Overused in tech (avoid unless specifically requested):** Inter, Roboto, Montserrat, Poppins as logo fonts. Blue-only palettes. Generic gradient swooshes.

**AI slop anti-patterns (never generate):**
- Generic shield/globe/lightbulb logos
- Purple/blue gradient swooshes
- Overlapping circles as "connection"
- Brain/neural network imagery for AI products
- Rocket ship for startups
- Clip-art-quality illustrations

---

## Phase 4: Logo generation (via Gemini API)

Generate logos ONLY after the user approves brand direction from Phase 3.

If Gemini is not available (from Phase 0), skip to Phase 5 and note: "Logo generation skipped. You can add logos manually or rerun /brand after setting GEMINI_API_KEY."

**Step 1: Write the generation script**

Write to `/tmp/brand-logo-gen.py`:

```python
#!/usr/bin/env python3
"""Brand logo + asset generator using Google Gemini API."""
import sys, os, json, io

try:
    from google import genai
    from google.genai import types
    from PIL import Image
except ImportError:
    os.system(f"{sys.executable} -m pip install -q google-genai Pillow")
    from google import genai
    from google.genai import types
    from PIL import Image

def generate_image(prompt, output_path, size=None):
    api_key = os.environ.get("GEMINI_API_KEY")
    if not api_key:
        print("ERROR: GEMINI_API_KEY not set")
        sys.exit(1)

    client = genai.Client(api_key=api_key)

    try:
        response = client.models.generate_content(
            model="gemini-2.0-flash-exp",
            contents=[prompt],
            config=types.GenerateContentConfig(
                response_modalities=["TEXT", "IMAGE"],
            ),
        )
    except Exception as e:
        print(f"API_ERROR: {e}")
        sys.exit(2)

    if not response.candidates:
        print("NO_CANDIDATES: Gemini returned no candidates")
        sys.exit(3)

    for part in response.candidates[0].content.parts:
        if hasattr(part, "inline_data") and part.inline_data:
            try:
                img = Image.open(io.BytesIO(part.inline_data.data)).convert("RGBA")
                # Resize to target if specified
                if size:
                    img = img.resize((size, size), Image.LANCZOS)
                img.save(output_path, format="PNG", optimize=True)
                w, h = img.size
                fsize = os.path.getsize(output_path)
                print(f"OK: {output_path} ({w}x{h}, {fsize // 1024}KB)")
                return
            except Exception as e:
                print(f"SAVE_ERROR: {e}")
                sys.exit(4)

    print("NO_IMAGE: Response had no image data")
    sys.exit(5)

if __name__ == "__main__":
    config = json.loads(sys.argv[1])
    generate_image(config["prompt"], config["output"], config.get("size"))
```

**Step 2: Craft logo prompts**

The prompt quality determines everything. Customize this template per brand:

```
Professional logo design for "[BRAND_NAME]", a [PRODUCT_DESCRIPTION].

Style: [LOGO_STYLE] -- [CONCEPT].
Colors: [PRIMARY_COLOR] as the main color. Must work on light and dark backgrounds.
Typography: [WORDMARK_STYLE] lettering.

Requirements:
- Clean, vector-style rendering, no photorealistic textures
- Simple enough to be recognizable at 16x16 favicon size
- Professional, modern, distinctive
- White/transparent background
- NO text other than the brand name
- NO generic clip-art (no lightbulbs, globes, rockets, shields, brains)
- ICONIC: recognizable at any size

This is for a tech product. It should look like it belongs on a $10B company, not a template marketplace.
```

**Step 3: Generate 3 variations**

```bash
python3 /tmp/brand-logo-gen.py '{"prompt":"PROMPT_V1","output":"/tmp/brand-logo-v1.png"}' 2>&1
python3 /tmp/brand-logo-gen.py '{"prompt":"PROMPT_V2","output":"/tmp/brand-logo-v2.png"}' 2>&1
python3 /tmp/brand-logo-gen.py '{"prompt":"PROMPT_V3","output":"/tmp/brand-logo-v3.png"}' 2>&1
```

Each prompt should be a different direction:
- V1: Main direction from Phase 3
- V2: Alternate style (e.g., if V1 is geometric, V2 is organic)
- V3: Minimal icon optimized for favicon/app icon

**Step 4: Handle errors**

Check each output. If any script returns non-zero:

- Exit code 1 (`GEMINI_API_KEY not set`): AskUserQuestion offering to set the key or skip.
- Exit code 2 (`API_ERROR`): Show the error. AskUserQuestion:
  > The Gemini API returned an error: [error message].
  > A) Retry with a simpler prompt
  > B) Skip logo generation and continue
  > C) I'll fix the API key and retry
- Exit code 3/5 (`NO_IMAGE`): Retry once with a simpler prompt. If still fails, skip that variation.
- If all 3 fail: AskUserQuestion explaining the situation, offer to continue without logos.

**Step 5: Present logos to user**

Open successfully generated logos:

```bash
open /tmp/brand-logo-v1.png /tmp/brand-logo-v2.png /tmp/brand-logo-v3.png 2>/dev/null
```

Also read the image files so you can describe them to the user.

AskUserQuestion:

> **Brand skill for [project name], Phase 4: Logo selection.**
>
> I generated [N] logo directions:
> - V1: [describe what you see]
> - V2: [describe]
> - V3: [describe]
>
> RECOMMENDATION: Choose [X] because [reason].
>
> A) Use V1
> B) Use V2
> C) Use V3
> D) Refine one (tell me which + what to change)
> E) Generate 3 more with different direction
> F) Skip logos, I'll design my own

**Step 6: Refinement (if requested)**

If the user wants to refine, write a new prompt incorporating their feedback and regenerate. Max 3 refinement rounds.

---

## Phase 5: Favicon and web asset generation

After logo is approved (or skipped), generate the complete web asset set.

If no logo was generated, skip this phase.

**Step 1: Write the asset pipeline script**

Write to `/tmp/brand-assets-gen.py`:

```python
#!/usr/bin/env python3
"""Generate complete web asset set from logo with proper sizes and compression."""
import sys, os, json
from PIL import Image

logo_path = sys.argv[1]
output_dir = sys.argv[2]
brand_color = sys.argv[3] if len(sys.argv) > 3 else "#ffffff"
os.makedirs(output_dir, exist_ok=True)

src = Image.open(logo_path).convert("RGBA")
report = {"files": [], "total_bytes": 0}

def save(img, name, fmt="PNG", quality=None, resize=None):
    if resize:
        img = img.resize((resize, resize), Image.LANCZOS)
    path = os.path.join(output_dir, name)
    kwargs = {"optimize": True}
    if fmt == "WEBP":
        kwargs["quality"] = quality or 85
        kwargs["method"] = 6
    elif fmt == "PNG":
        kwargs["optimize"] = True
    elif fmt == "JPEG":
        kwargs["quality"] = quality or 90
        kwargs["optimize"] = True
        # JPEG needs RGB, composite onto white
        if img.mode == "RGBA":
            bg = Image.new("RGB", img.size, (255, 255, 255))
            bg.paste(img, mask=img.split()[3])
            img = bg
    img.save(path, format=fmt, **kwargs)
    fsize = os.path.getsize(path)
    w, h = img.size if not resize else (resize, resize)
    report["files"].append({"name": name, "size": f"{w}x{h}", "bytes": fsize})
    report["total_bytes"] += fsize
    print(f"OK: {name} ({w}x{h}, {fsize // 1024}KB, {fmt})")
    return img

# === FAVICONS (must be pixel-perfect at small sizes) ===
save(src, "favicon-16x16.png", resize=16)
save(src, "favicon-32x32.png", resize=32)
save(src, "favicon-48x48.png", resize=48)

# Multi-size .ico (16+32+48)
ico_imgs = [src.resize((s, s), Image.LANCZOS) for s in [16, 32, 48]]
ico_path = os.path.join(output_dir, "favicon.ico")
ico_imgs[0].save(ico_path, format="ICO",
    sizes=[(16, 16), (32, 32), (48, 48)], append_images=ico_imgs[1:])
fsize = os.path.getsize(ico_path)
report["files"].append({"name": "favicon.ico", "size": "multi", "bytes": fsize})
report["total_bytes"] += fsize
print(f"OK: favicon.ico (multi-size, {fsize // 1024}KB)")

# SVG favicon placeholder (text file with instructions)
svg_path = os.path.join(output_dir, "favicon.svg.txt")
with open(svg_path, "w") as f:
    f.write("<!-- Replace this with a hand-crafted SVG version of your logo -->\n")
    f.write("<!-- SVG favicons support dark mode via prefers-color-scheme -->\n")
print("OK: favicon.svg.txt (placeholder)")

# === APPLE / ANDROID ICONS ===
save(src, "apple-touch-icon.png", resize=180)
save(src, "android-chrome-192x192.png", resize=192)
save(src, "android-chrome-512x512.png", resize=512)

# === LOGO VARIANTS (multiple sizes for different uses) ===
save(src, "logo-64.png", resize=64)              # Navbar, small UI
save(src, "logo-128.png", resize=128)             # Medium UI, email headers
save(src, "logo-256.png", resize=256)             # Large UI, about pages
save(src, "logo-512.png", resize=512)             # High-res, retina
save(src, "logo-1024.png", resize=1024)           # Print, marketing

# WebP versions (smaller file size for web)
save(src, "logo-64.webp", fmt="WEBP", resize=64, quality=90)
save(src, "logo-128.webp", fmt="WEBP", resize=128, quality=90)
save(src, "logo-256.webp", fmt="WEBP", resize=256, quality=90)
save(src, "logo-512.webp", fmt="WEBP", resize=512, quality=85)

# === SOCIAL / OG IMAGES ===
# OG image: 1200x630 (logo centered on brand-colored background)
def hex_to_rgb(h):
    h = h.lstrip("#")
    return tuple(int(h[i:i+2], 16) for i in (0, 2, 4))

og = Image.new("RGB", (1200, 630), hex_to_rgb(brand_color))
logo_for_og = src.resize((300, 300), Image.LANCZOS)
x = (1200 - 300) // 2
y = (630 - 300) // 2
og.paste(logo_for_og, (x, y), logo_for_og)
save(og, "og-image.png", fmt="PNG")
save(og, "og-image.jpg", fmt="JPEG", quality=85)
save(og, "og-image.webp", fmt="WEBP", quality=80)

# Twitter card: 1200x600
tw = Image.new("RGB", (1200, 600), hex_to_rgb(brand_color))
tw.paste(logo_for_og, (x, (600 - 300) // 2), logo_for_og)
save(tw, "twitter-card.jpg", fmt="JPEG", quality=85)

# Square social: 1080x1080 (Instagram, etc.)
sq = Image.new("RGB", (1080, 1080), hex_to_rgb(brand_color))
logo_sq = src.resize((540, 540), Image.LANCZOS)
sq.paste(logo_sq, (270, 270), logo_sq)
save(sq, "social-square.jpg", fmt="JPEG", quality=85)
save(sq, "social-square.webp", fmt="WEBP", quality=80)

# === MANIFEST FILES ===
manifest = {
    "name": "",
    "short_name": "",
    "icons": [
        {"src": "/android-chrome-192x192.png", "sizes": "192x192", "type": "image/png"},
        {"src": "/android-chrome-512x512.png", "sizes": "512x512", "type": "image/png"}
    ],
    "theme_color": brand_color,
    "background_color": "#ffffff",
    "display": "standalone"
}
manifest_path = os.path.join(output_dir, "site.webmanifest")
with open(manifest_path, "w") as f:
    json.dump(manifest, f, indent=2)
print("OK: site.webmanifest")

# browserconfig.xml for Microsoft
bc_path = os.path.join(output_dir, "browserconfig.xml")
with open(bc_path, "w") as f:
    f.write(f'<?xml version="1.0" encoding="utf-8"?>\n')
    f.write(f'<browserconfig><msapplication><tile>\n')
    f.write(f'<square150x150logo src="/android-chrome-192x192.png"/>\n')
    f.write(f'<TileColor>{brand_color}</TileColor>\n')
    f.write(f'</tile></msapplication></browserconfig>\n')
print("OK: browserconfig.xml")

# === HEAD TAG SNIPPET ===
head_path = os.path.join(output_dir, "head-tags.html")
with open(head_path, "w") as f:
    f.write('<!-- Favicon and brand assets - copy into <head> -->\n')
    f.write('<link rel="icon" type="image/x-icon" href="/favicon.ico">\n')
    f.write('<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">\n')
    f.write('<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">\n')
    f.write('<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">\n')
    f.write('<link rel="manifest" href="/site.webmanifest">\n')
    f.write(f'<meta name="theme-color" content="{brand_color}">\n')
    f.write(f'<meta name="msapplication-TileColor" content="{brand_color}">\n')
    f.write('<meta property="og:image" content="/og-image.jpg">\n')
    f.write('<meta property="og:image:width" content="1200">\n')
    f.write('<meta property="og:image:height" content="630">\n')
    f.write('<meta name="twitter:card" content="summary_large_image">\n')
    f.write('<meta name="twitter:image" content="/twitter-card.jpg">\n')
print("OK: head-tags.html")

# === REPORT ===
total_kb = report["total_bytes"] // 1024
print(f"\nTOTAL: {len(report['files'])} files, {total_kb}KB")
print(f"\nFile sizes:")
for f in sorted(report["files"], key=lambda x: -x["bytes"]):
    print(f"  {f['name']:40s} {f['size']:>10s}  {f['bytes'] // 1024:>5d}KB")
```

**Step 2: Run it**

```bash
python3 /tmp/brand-assets-gen.py "/tmp/brand-logo-chosen.png" "/tmp/brand-assets/" "#PRIMARY_HEX" 2>&1
```

Replace `#PRIMARY_HEX` with the primary brand color from Phase 3.

If the logo is too complex for 16px favicon (check the 16x16 output), generate a separate simplified icon via Gemini with a prompt focused on minimal single-shape mark. Then rerun the script with the simplified icon for favicon sizes only.

**Step 3: Install into project**

Detect framework:

```bash
[ -f "next.config.js" ] || [ -f "next.config.ts" ] || [ -f "next.config.mjs" ] && echo "NEXTJS"
[ -f "vite.config.ts" ] || [ -f "vite.config.js" ] && echo "VITE"
[ -f "nuxt.config.ts" ] && echo "NUXT"
[ -d "public" ] && echo "HAS_PUBLIC"
```

AskUserQuestion before copying:

> **Brand skill for [project name], Phase 5: Installing brand assets.**
>
> I've generated a complete web asset set:
> - Favicons: .ico (multi-size), 16px, 32px, 48px PNG
> - App icons: apple-touch-icon (180px), Android Chrome (192px, 512px)
> - Logo variants: 64px to 1024px in PNG + WebP
> - Social/OG: og-image (1200x630), twitter-card (1200x600), social-square (1080x1080) in JPG + WebP
> - Config: site.webmanifest, browserconfig.xml, head-tags.html snippet
>
> Total: [N] files, [X]KB. WebP versions are 30-50% smaller than PNG.
>
> RECOMMENDATION: Choose A to install to your project.
>
> A) Install to [detected public/ path]
> B) Save to /tmp/brand-assets/ only, I'll move them myself
> C) Skip asset installation

If A:
- Copy favicon files, app icons, and config files to `public/`
- Copy logo variants to `public/brand/` or `public/images/brand/`
- Copy OG/social images to `public/`
- For Next.js App Router: also copy favicon.ico to `app/favicon.ico`
- Print the content of `head-tags.html` so the user can add it to their layout

---

## Phase 6: Brand preview page

Generate a self-contained HTML preview showing the full identity.

```bash
PREVIEW_FILE="/tmp/brand-preview-$(date +%s).html"
```

**The preview page must include:**

1. Brand header with logo (embedded as base64 if generated) + product name in brand typography
2. Brand essence section (mission, personality, positioning)
3. Logo showcase on light bg, dark bg, and primary color bg
4. Color palette as visual swatches with hex values
5. Typography specimen showing each font in its role with real product copy
6. Brand voice examples in the brand's actual tone
7. Favicon preview at actual sizes
8. UI mockup: realistic product screen using the brand system (buttons, cards, form, nav)
9. Light/dark mode toggle via CSS custom properties + JS
10. Do/Don't section: correct vs incorrect logo usage

Load fonts from Google Fonts via `<link>` tags. Use the actual brand colors throughout. Use real product copy, not lorem ipsum.

The preview page should look like a professional branding agency deliverable.

```bash
open "$PREVIEW_FILE"
```

If `open` fails, tell the user: "Preview written to [path]. Open it in your browser."

---

## Phase 7: Write BRAND.md and install assets

Write `BRAND.md` to the repo root with this structure:

```markdown
# Brand Identity -- [Project Name]

## Brand essence
- **Mission:** [one sentence]
- **Personality:** [3-4 adjectives]
- **Positioning:** [one-line differentiator]
- **Tagline:** [if created]

## Logo
- **Primary logo:** [path to file]
- **Icon/Favicon:** [path to simplified icon]
- **Style:** [description]
- **Concept:** [what it represents]
- **Clear space:** Minimum padding = height of the logo mark
- **Minimum size:** 24px height for digital, 10mm for print

### Logo usage rules
- DO: Use on solid backgrounds (light, dark, or brand primary)
- DO: Maintain aspect ratio
- DON'T: Stretch, rotate, add effects, or change colors
- DON'T: Place on busy backgrounds without contrast overlay

## Color palette
| Role | Name | Hex | Usage |
|------|------|-----|-------|
| Primary | [name] | [hex] | Main brand color, CTAs, links |
| Secondary | [name] | [hex] | Supporting elements |
| Accent | [name] | [hex] | Highlights, notifications |
| Background | [name] | [hex] | Page background |
| Surface | [name] | [hex] | Cards, elevated elements |
| Text | [name] | [hex] | Primary text |
| Muted | [name] | [hex] | Secondary text, borders |

### Semantic colors
- Success: [hex]
- Warning: [hex]
- Error: [hex]
- Info: [hex]

### Dark mode
[Strategy and hex overrides]

## Typography
| Role | Font | Weight | Size | Usage |
|------|------|--------|------|-------|
| Display | [font] | [weight] | [size] | Hero headings |
| Heading | [font] | [weight] | [scale] | Section headings |
| Body | [font] | [weight] | [size] | Paragraphs |
| UI | [font] | [weight] | [size] | Buttons, labels |
| Mono | [font] | [weight] | [size] | Code, data |

**Loading:** [CDN links or self-hosted strategy]

## Brand voice
- **Tone:** [description]
- **Do:** [3 guidelines]
- **Don't:** [3 anti-patterns]
- **Example:** [sample headline + paragraph]

## Assets
| Asset | Path | Dimensions |
|-------|------|------------|
| Logo (PNG) | [path] | [size] |
| Favicon (ICO) | public/favicon.ico | multi-size |
| Apple Touch Icon | public/apple-touch-icon.png | 180x180 |
| OG Image Icon | public/og-icon.png | 512x512 |

## Decisions log
| Date | Decision | Rationale |
|------|----------|-----------|
| [today] | Initial brand identity | Created by /brand |
```

**Update CLAUDE.md** (append if exists, create if not):

```markdown
## Brand identity
Read BRAND.md before making any visual, UI, or copy decisions.
Logo, colors, typography, and brand voice are defined there.
Do not deviate without explicit user approval.
Follow brand voice guidelines when writing copy.
```

**Final confirmation via AskUserQuestion:**

> **Brand skill for [project name], Phase 7: Final review.**
>
> Brand identity complete. Here's what I've created:
> - Logo: [generated/skipped]
> - Favicons: [installed to X / skipped]
> - Colors: [N colors defined]
> - Typography: [N fonts]
> - Preview: [path]
>
> RECOMMENDATION: Choose A to write everything to your project.
>
> A) Ship it. Write BRAND.md and install assets.
> B) Adjust something (tell me what)
> C) Generate different logo variations
> D) Start over

After writing files, report completion status:

```
STATUS: DONE
DELIVERABLES:
- BRAND.md written to repo root
- [Logo files installed to X]
- [Favicon set installed to X]
- [CLAUDE.md updated with brand section]
- [Preview page at /tmp/brand-preview-XXX.html]
```

---

## Important rules

1. **Be opinionated.** Propose specific choices with rationale. Consultant, not menu.
2. **Coherence above all.** Every element (logo, color, font, voice) must reinforce the others.
3. **No AI slop.** No generic logos, no purple gradients, no template-marketplace energy.
4. **Logo prompts are everything.** Spend time crafting the Gemini prompt. Be hyper-specific.
5. **Favicon must work at 16px.** If logo is too complex, generate a separate simplified icon.
6. **Brand preview = portfolio piece.** The HTML page should look like a professional deliverable.
7. **Real content, not lorem ipsum.** Use the actual product name and realistic copy.
8. **Accept the user's taste.** Nudge on coherence, never block.
9. **Iterate fast.** Logo not right? Generate more. Don't make the user work hard.
10. **Install cleanly.** Detect framework, put files in the right places, don't break existing code.
11. **Always use AskUserQuestion** for decisions. Never assume. Always re-ground the user.
12. **Graceful degradation.** No Gemini key? Skip logos. No WebSearch? Use built-in knowledge. No Chrome? Skip visual research. The skill always produces value.
