# Brand — Complete Brand Identity for Claude Code

A Claude Code skill that builds your entire brand identity from scratch. Learns your product, researches the competitive landscape, proposes a cohesive identity system, generates logos via Gemini API, creates favicon sets, and writes `BRAND.md` as your single source of truth.

One command. Full brand kit.

## What It Does

| Phase | Description |
|-------|-------------|
| Discovery | Reads your codebase, asks one question to understand your product |
| Research | Analyzes competitor branding via web search and browser screenshots |
| Proposal | Full identity: brand essence, logo direction, colors, typography, voice |
| Logo Generation | 3 variations via Google Gemini API with multi-turn refinement |
| Favicon | Complete set (16px to 512px + .ico) from approved logo |
| Preview | Agency-quality HTML brand kit with dark mode toggle |
| Install | Writes BRAND.md, installs assets into your project, updates CLAUDE.md |

## Install

```bash
git clone https://github.com/beknazar/brand.git ~/.claude/skills/brand
```

## Usage

In any project directory, run:

```
/brand
```

The skill walks you through the full brand identity process interactively.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- `GEMINI_API_KEY` environment variable (for logo generation)
  - Get one free at [Google AI Studio](https://aistudio.google.com/apikey)
  - Logo generation is optional — the skill works without it

```bash
export GEMINI_API_KEY=your_key_here
```

## What You Get

After running `/brand`, your project will have:

- **BRAND.md** — Complete brand guidelines (colors, fonts, voice, logo usage rules)
- **Logo files** — PNG in multiple sizes, generated via AI
- **Favicon set** — All standard sizes (16x16 to 512x512) + .ico + apple-touch-icon
- **Brand preview page** — Self-contained HTML showing the full identity system
- **CLAUDE.md update** — So every future AI interaction respects your brand

## Design Philosophy

Inspired by how [gstack](https://github.com/garrytan/gstack) approaches design consultation:

- **Opinionated consultant, not a form wizard.** The skill proposes specific choices with rationale, not generic menus.
- **Coherence above individual choices.** Every element (logo, color, font, voice) reinforces the others.
- **No AI slop.** No generic shield logos, no purple gradients, no template-marketplace energy.
- **SAFE/BOLD breakdown.** Shows which choices match industry conventions and which are deliberate departures.
- **Accept the user's taste.** Nudges on coherence issues, never blocks.

## Output Example

The skill generates a `BRAND.md` that covers:

- Brand essence (mission, personality, positioning)
- Logo specifications and usage rules
- Color palette with semantic colors and dark mode strategy
- Typography scale with specific fonts and weights
- Brand voice guidelines with do/don't examples
- Asset inventory with paths and dimensions

## Author

**Bek** — [@beknabdik](https://x.com/beknabdik) / [GitHub](https://github.com/beknazar)

## License

MIT
