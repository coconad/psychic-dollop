# Studio Fluff

Content creation pipeline for Dante (Poodle) and Flora (Maltipoo), built around modular Claude skills and script-based automation.

## Overview

Studio Fluff uses 8 coordinated skills to go from trend discovery to published social posts:

1. `brand-context` (auto-loaded)
2. `trend-scout`
3. `plan-content`
4. `generate-caption`
5. `generate-image`
6. `generate-video`
7. `post-content`
8. `create-post` (orchestrator)

Core runtime assumptions:

- Node.js `>=20.6.0`
- ESM (`"type": "module"`)
- Zero npm dependencies (scripts use Node built-ins)
- Environment variables loaded via `--env-file=.env`

## Project Structure

```text
studio-fluff/
├── .claude/skills/
│   ├── brand-context/
│   │   ├── SKILL.md                     (user-invocable: false)
│   │   └── reference/
│   │       ├── dante-profile.md
│   │       └── flora-profile.md
│   ├── trend-scout/
│   │   ├── SKILL.md
│   │   ├── scripts/search_trends.js
│   │   └── lib/
│   │       ├── apify-client.js
│   │       ├── report-builder.js
│   │       └── platform-adapters/
│   │           ├── tiktok.js
│   │           └── instagram.js
│   ├── plan-content/
│   │   ├── SKILL.md
│   │   └── templates/content-plan-schema.json
│   ├── generate-caption/
│   │   └── SKILL.md                     (no scripts — pure LLM)
│   ├── generate-image/
│   │   ├── SKILL.md
│   │   └── scripts/gemini_generate.js
│   ├── generate-video/
│   │   ├── SKILL.md
│   │   ├── scripts/veo3_generate.js
│   │   └── scripts/kling_generate.js
│   ├── post-content/
│   │   ├── SKILL.md
│   │   ├── scripts/post_instagram.js
│   │   └── scripts/post_tiktok.js
│   └── create-post/
│       └── SKILL.md                     (orchestration only)
├── assets/dogs/
│   ├── dante/    (reference photos)
│   └── flora/    (reference photos)
├── data/
│   ├── content-plans/   (JSON per batch)
│   ├── trends/          (JSON per trend scan)
│   └── post-history.json
├── output/
│   ├── images/
│   └── videos/
├── package.json
├── .env.example
├── .env           (gitignored)
├── CLAUDE.md
└── .gitignore
```

## Environment Setup

1. Use Node.js `20.6.0` or newer.
2. Copy env template:

```bash
cp .env.example .env
```

3. Fill required keys in `.env`:

```env
# Apify (trend scouting)
APIFY_TOKEN=

# Google AI (image + video generation)
GOOGLE_API_KEY=
GOOGLE_CLOUD_PROJECT=

# Kling AI (video generation fallback)
KLING_API_KEY=
KLING_API_SECRET=

# Instagram Graph API (publishing)
INSTAGRAM_ACCESS_TOKEN=
INSTAGRAM_BUSINESS_ACCOUNT_ID=

# TikTok Content Posting API (publishing)
TIKTOK_ACCESS_TOKEN=
```

## Available Script Commands

From `package.json`:

```bash
npm run trend-scout
npm run generate-image
npm run generate-video:veo
npm run generate-video:kling
npm run post:instagram
npm run post:tiktok
```

## Skill Responsibilities

| Skill | Role |
|---|---|
| `brand-context` | Shared voice, persona, and style context for Dante and Flora |
| `trend-scout` | Collects trend signals and writes reports to `data/trends/` |
| `plan-content` | Builds content plans and writes JSON batches to `data/content-plans/` |
| `generate-caption` | Produces platform-ready caption variants |
| `generate-image` | Creates reference images into `output/images/` |
| `generate-video` | Generates short videos into `output/videos/` |
| `post-content` | Publishes to Instagram and TikTok, updates `data/post-history.json` |
| `create-post` | End-to-end orchestration with checkpoints |

## Pipeline Flow

```text
trend-scout -> plan-content -> generate-caption -> generate-image -> generate-video -> post-content
```

`brand-context` is ambient and auto-loaded.  
`create-post` coordinates the full flow and supports step-by-step progression.

## Data and Outputs

- Trend scans: `data/trends/*.json`
- Content plans: `data/content-plans/*.json`
- Post log: `data/post-history.json`
- Images: `output/images/*`
- Videos: `output/videos/*`

## Notes

- Keep `.env` out of source control.
- Keep reference photos in `assets/dogs/dante/` and `assets/dogs/flora/` for better generation consistency.
- Prefer resuming via the orchestrator skill (`create-post`) when a run is interrupted.
