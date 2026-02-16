PROJECT STRUCTURE

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
