# Studio Fluff — Content Creation Pipeline

Social media pipeline for **Dante** (male Poodle) and **Flora** (female Maltipoo).
8 Claude Code skills that chain into an end-to-end workflow.

---

## Tech Stack

| Layer | Tool | Auth |
|-------|------|------|
| Trends | Apify actors | `APIFY_TOKEN` |
| Image gen | Gemini (REST API) | `GOOGLE_API_KEY` |
| Video gen | Veo 3 (primary) / Kling AI (fallback) | `GOOGLE_API_KEY` / `KLING_API_KEY` + `KLING_API_SECRET` |
| Publishing | Instagram Graph API + TikTok Content Posting API | `INSTAGRAM_ACCESS_TOKEN` + `INSTAGRAM_BUSINESS_ACCOUNT_ID` / `TIKTOK_ACCESS_TOKEN` |
| State | Local JSON files (`data/`) | — |
| Runtime | Node.js 20.6+ (ESM, no npm deps, `--env-file`) | — |

---

## Skills Overview (8 total)

| # | Skill | Invocation | Purpose |
|---|-------|------------|---------|
| 1 | `brand-context` | Auto (Claude-only) | Ambient personality & tone guide |
| 2 | `trend-scout` | `/trend-scout` | Apify-powered trend analysis across TikTok and Instagram |
| 3 | `plan-content` | `/plan-content` | Create content calendar entries, check history for dupes |
| 4 | `generate-caption` | `/generate-caption` | Platform-specific captions, hashtags, CTAs |
| 5 | `generate-image` | `/generate-image` | Gemini reference image from dog photos |
| 6 | `generate-video` | `/generate-video` | Veo 3 / Kling short-form video from reference image |
| 7 | `post-content` | `/post-content` | Publish to Instagram + TikTok |
| 8 | `create-post` | `/create-post` | Master workflow chaining steps 2→7 with user checkpoints |

---

## Project Structure

```
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

---

## Skill Details

### 1. brand-context
Ambient personality and tone guide that auto-loads for all content tasks. Contains dog profiles for Dante (sophisticated, dramatic Poodle) and Flora (chaotic, sweet Maltipoo), brand voice guidelines, platform-specific posting styles, and visual direction. Not user-invocable — Claude loads it automatically when creating content.

### 2. trend-scout
Scans TikTok and Instagram via Apify actors for trending hashtags, sounds, and content patterns relevant to dog content. Accepts `--keywords`, `--platforms`, `--region`, and `--limit` arguments. Outputs normalized JSON reports to `data/trends/` with dog relevance scores (0–1) for each trend. Uses modular platform adapters and a shared Apify client library.

### 3. plan-content
Creates content calendar entries based on trend insights and brand context. Reads existing `data/post-history.json` to avoid duplicate concepts. Generates plan entries with UUID, title, concept, visual direction, caption direction, and scheduling info. Saves plans to `data/content-plans/YYYY-MM-DD-plan.json`. Pure LLM orchestration — no scripts.

### 4. generate-caption
Takes a plan entry and generates platform-specific captions. Instagram captions include an emotional hook, body, CTA, 5–8 hashtags, and alt text for accessibility. TikTok captions include a scroll-stopping hook, short body, 3–5 hashtags, and sound suggestions. Updates plan entry status from `planned` → `caption_done`. Pure LLM — no scripts.

### 5. generate-image
Generates reference images using Gemini `gemini-2.5-flash-image` model via REST API (zero npm deps). Reads dog reference photos from `assets/dogs/{dante,flora}/` as base64 inline data to maintain breed accuracy. Supports `--aspect-ratio` (9:16 default, 4:5, 1:1). Saves images + `.meta.json` sidecar to `output/images/`. Updates plan entry status from `caption_done` → `image_done`.

### 6. generate-video
Creates short-form video from a reference image using two providers. **Veo 3** (primary): Gemini API `veo-3.0-generate-001` with async polling, supports 4/5/6/8s durations. **Kling AI** (fallback): REST API with HS256 JWT auth via `node:crypto`, model `kling-v2-6`, supports 5/10s durations in std/pro modes. Both save videos + `.meta.json` sidecar to `output/videos/`. Updates plan entry status from `image_done` → `video_done`.

### 7. post-content
Publishes content to Instagram and TikTok. Runs pre-flight validation and viral optimization review before posting. **Instagram**: Graph API v21.0, creates REELS container → polls status → publishes (requires public video URL). **TikTok**: Content Posting API v2, supports `PULL_FROM_URL` or `FILE_UPLOAD` (chunked), default `SELF_ONLY` privacy. Both scripts append to `data/post-history.json`. Updates plan entry status to `published`. Includes embedded viral best practices (algorithm signals, posting times, hook formulas, hashtag strategies).

### 8. create-post
Master workflow that orchestrates the entire pipeline end-to-end. Pure orchestration — no scripts. Supports `--from concept|plan|caption|image|video` to resume from any stage. Shows a user checkpoint between every step (never auto-posts). Includes error recovery with options to retry, switch provider, skip, or abort. Displays a pipeline summary table at completion.

---

## Pipeline Order

The `create-post` master skill chains all production skills in this order:

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│ trend-scout │ ──→ │ plan-content │ ──→ │ generate-caption │
└─────────────┘     └──────────────┘     └──────────────────┘
                                                  │
                                                  ▼
┌──────────────┐     ┌────────────────┐     ┌────────────────┐
│ post-content │ ←── │ generate-video │ ←── │ generate-image │
└──────────────┘     └────────────────┘     └────────────────┘
```

| Step | Skill | Input | Output | Status After |
|------|-------|-------|--------|-------------|
| 1 | `trend-scout` | Keywords, platforms | Trend report in `data/trends/` | — |
| 2 | `plan-content` | Trend insights + brand context | Plan entry in `data/content-plans/` | `planned` |
| 3 | `generate-caption` | Plan entry | IG + TikTok captions, alt text, sound suggestion | `caption_done` |
| 4 | `generate-image` | Plan entry + dog reference photos | Reference image in `output/images/` | `image_done` |
| 5 | `generate-video` | Reference image + prompt | Short-form video in `output/videos/` | `video_done` |
| 6 | `post-content` | Video + captions | Published to IG + TikTok, logged to `post-history.json` | `published` |

The `brand-context` skill is not a pipeline step — it auto-loads as ambient context into `plan-content` and `generate-caption` via their `context:` frontmatter field.

A user checkpoint is presented between every step. Use `--from` to resume from any stage if the pipeline is interrupted.

---

## Key Design Decisions

1. **Zero npm dependencies** — All scripts use Node built-in `fetch`, `fs`, `path`, `crypto`, `parseArgs`
2. **ESM throughout** — `"type": "module"` in package.json
3. **`--env-file=.env`** — Native Node 20.6+ env loading, no dotenv
4. **Separate scripts per provider** — Simpler than abstracting; SKILL.md handles orchestration
5. **JSON state files** — Appropriate for low volume (~few posts/week), easy to debug and git-track
6. **User checkpoints in pipeline** — Never auto-publish; every step needs explicit approval

---

## Verification

| Skill | Test |
|-------|------|
| brand-context | Ask Claude about Dante/Flora personality → should reference profiles |
| trend-scout | `/trend-scout --keywords "dog,puppy" --platforms tiktok` |
| plan-content | `/plan-content "Dante playing in autumn leaves"` → check `data/content-plans/` |
| generate-caption | `/generate-caption PLAN_ID` → review IG + TikTok captions |
| generate-image | `node --env-file=.env .claude/skills/generate-image/scripts/gemini_generate.js --prompt "test" --output output/images/test.png` |
| generate-video | Test each script independently with a reference image |
| post-content | Use draft/private mode (TikTok `SELF_ONLY`, IG sandbox) |
| create-post | `/create-post --from concept "Dante and Flora sharing a toy"` → full pipeline |

**End-to-end**: Add dog photos → set API keys → `/create-post` → walk through all checkpoints → verify files in `data/` and `output/` → check draft post on platforms.
