# Studio Fluff

Social media content creation pipeline for two dogs:
- **Dante** — Male Poodle (see `assets/dogs/dante/`)
- **Flora** — Female Maltipoo (see `assets/dogs/flora/`)

## Project Structure
- Skills: `.claude/skills/` (7 individual skills + 1 master workflow)
- Content plans: `data/content-plans/` (JSON files per batch)
- Trend reports: `data/trends/` (JSON files per scan)
- Post history: `data/post-history.json`
- Generated images: `output/images/`
- Generated videos: `output/videos/`
- Dog reference photos: `assets/dogs/{dante,flora}/`

## Environment
- Node.js 20.6+ required (for `--env-file` support)
- All scripts use ESM imports (`"type": "module"` in package.json)
- Zero npm dependencies — scripts use Node built-in `fetch`, `fs`, `path`, `crypto`, `parseArgs`
- API keys stored in `.env` (see `.env.example`)

## Skills
| Skill | Command | Purpose |
|-------|---------|---------|
| brand-context | _(auto-loaded)_ | Ambient personality & tone guide for Dante and Flora |
| trend-scout | `/trend-scout` | Apify-powered trend analysis across TikTok, IG, Google |
| plan-content | `/plan-content` | Create content calendar entries, check history for dupes |
| generate-caption | `/generate-caption` | Platform-specific captions, hashtags, CTAs |
| generate-image | `/generate-image` | Generate reference images via Gemini Nano Banana |
| generate-video | `/generate-video` | Create short-form video via Veo 3 / Kling AI |
| post-content | `/post-content` | Publish to Instagram and TikTok |
| create-post | `/create-post` | End-to-end pipeline chaining all skills |
