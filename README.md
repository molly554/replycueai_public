# ReplyCue AI — Claude Skills for YouTube Brand Intelligence

[![Claude Code](https://img.shields.io/badge/Claude_Code-Skills-blue?logo=anthropic)](https://claude.ai/code)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**AI-powered YouTube comment analysis for brands, marketers, and creators.** Analyze comments, detect threats, generate smart replies, and measure KOL collaboration ROI — all from your terminal with Claude Code.

<video width="640" controls>
  <source src="https://raw.githubusercontent.com/molly554/replycueai_public/main/replycueai.mp4" type="video/mp4">
</video>

> Built by the team behind [ReplyCue AI](https://replycueai.com) — the AI comment moderation platform for YouTube.
> 

---

## What's Inside

| Skill | Description | Use Case |
|-------|-------------|----------|
| [YouTube Comment Analyzer](skills/youtube-comment-analyzer/) | Analyze video comments with 5-layer brand relevance scoring | "Is anyone talking about my brand in this KOL video?" |
| [Smart Reply Generator](skills/smart-reply-generator/) | Generate brand-appropriate replies in 3 tones | "Help me write a professional reply to this complaint" |
| [KOL ROI Report](skills/kol-roi-report/) | Comprehensive collaboration effectiveness report | "Was this $5K sponsorship worth it?" |
| [Brand Keyword Extractor](skills/brand-keyword-extractor/) | Extract keywords from any website for SEO and monitoring | "What keywords should I track for my brand?" |
| [Comment Threat Scanner](skills/comment-threat-scanner/) | Detect scams, fake claims, and brand safety risks | "Are there any dangerous comments I need to handle?" |

## Quick Start

### 1. Prerequisites

You need two API keys:

```bash
# Required for all skills
export ANTHROPIC_API_KEY="sk-ant-..."

# Required for YouTube-related skills (1, 3, 5)
export YOUTUBE_API_KEY="AIza..."
```

- Get your Anthropic API key at [console.anthropic.com](https://console.anthropic.com)
- Get your YouTube API key at [Google Cloud Console](https://console.cloud.google.com/apis/credentials) (enable YouTube Data API v3)

### 2. Install Skills

Clone this repo and add the skills directory to your Claude Code configuration:

```bash
git clone https://github.com/YOUR_USERNAME/replycue-ai-skills.git

# Add to your project's .claude/settings.json
# or copy skills/ into your .claude/skills/ directory
```

### 3. Use in Claude Code

Just describe what you want in natural language:

```
> Analyze the comments on this YouTube video for my brand ShieldVPN:
  https://www.youtube.com/watch?v=VIDEO_ID

> Generate a professional reply to this comment:
  "Your product crashed 3 times today, worst purchase ever"

> Give me a KOL ROI report for this sponsored video:
  https://www.youtube.com/watch?v=VIDEO_ID
  Brand: Notion, Collaboration type: sponsored

> Extract brand keywords from https://notion.so

> Scan this video for brand safety threats:
  https://www.youtube.com/watch?v=VIDEO_ID
  Brand: Notion
```

## How the Scoring Works

Our **5-layer brand relevance model** goes beyond simple keyword matching:

```
Final Score = 50% Semantic Score      ← Does the comment text relate to the brand?
            + 15% Video Relevance     ← Is the video itself about the brand?
            + 15% Entity Match        ← Is the brand name explicitly mentioned?
            + 10% Intent Score        ← Is this a purchase intent, question, or risk?
            + 10% Context Score       ← How deep is the brand integration in the video?
```

Different collaboration types have different relevance thresholds:

| Type | Threshold | Why |
|------|-----------|-----|
| Dedicated (full review) | 50% | Most comments on a dedicated video are relevant |
| Sponsored (segment) | 60% | Filter off-topic discussion |
| Mention (brief) | 85% | Only clearly brand-related comments pass |
| Placement (background) | 85% | Product is barely discussed |

[Read the full scoring model documentation →](docs/scoring-model.md)

## Output Formats

All analysis skills support multiple output formats:

- **Markdown** (default) — Human-readable reports
- **CSV** — Import into spreadsheets or databases
- **JSON** — Integrate into your own tools and pipelines

[See example outputs →](docs/examples/)

## Multi-Language Support

All skills support comments in any language:
- English, Chinese (中文), Japanese (日本語), Korean (한국어)
- Hindi (हिन्दी), Arabic (العربية), Spanish, Portuguese
- And 20+ more languages

Output language automatically matches your preference or the comment's language.

## Skill Details

### YouTube Comment Analyzer

The flagship skill. Fetches comments from any YouTube video and classifies each one into 7 business categories:

| Category | What it means | Priority |
|----------|--------------|----------|
| `purchase_intent` | User wants to buy | Medium |
| `product_question` | Questions about features, pricing, bugs | Medium |
| `brand_risk` | False claims, scams, privacy concerns | **High** |
| `competitor_mention` | Compares with competitors | Medium |
| `positive_feedback` | Praises the product | Low |
| `negative_feedback` | Complaints, disappointment | Medium |
| `irrelevant` | Not about the brand | — (filtered) |

### Smart Reply Generator

Generates 3 reply options per comment with strategy explanations:

- **Professional** — Clear, direct, respectful
- **Friendly** — Warm, approachable, personable
- **Casual** — Conversational, like texting a friend

### KOL ROI Report

Full collaboration analysis including:
- Engagement quality score
- Purchase signal strength
- Brand risk level
- Audience-brand alignment
- Actionable recommendations

### Brand Keyword Extractor

Extracts 5 categories of keywords:
- Primary keywords (high-value, core brand terms)
- Secondary keywords (related, broader reach)
- Long-tail phrases (with search intent classification)
- Competitor keywords
- Negative keywords (terms to exclude)

### Comment Threat Scanner

Focused brand safety scan with 3 severity tiers:
- **HIGH** — Scams, impersonation, false claims, doxxing
- **MEDIUM** — KOL misrepresentation, privacy concerns, competitor attacks
- **LOW** — Negative experiences, general toxicity, spam

## Cost Estimate

All skills use `claude-haiku-4-5-20251001` for cost efficiency:

| Skill | Typical API Cost | Comments Analyzed |
|-------|-----------------|-------------------|
| Comment Analyzer | ~$0.02-0.05 | 100 comments |
| Smart Reply | ~$0.001 | 1 comment |
| KOL ROI Report | ~$0.03-0.08 | 100 comments + summary |
| Keyword Extractor | ~$0.002 | 1 website |
| Threat Scanner | ~$0.02-0.05 | 100 comments |

## Want Continuous Monitoring?

These skills are great for one-time analysis. For **ongoing, automated** YouTube comment monitoring:

**[ReplyCue AI](https://replycueai.com)** provides:
- Real-time comment monitoring across multiple videos
- Automatic AI analysis as new comments arrive
- Team collaboration with role-based access
- One-click AI reply generation
- Dashboard with business intelligence
- Alert rules for high-priority comments
- Multi-video KOL performance comparison

[Try ReplyCue AI →](https://replycueai.com)

## License

MIT License — use these skills freely in your projects.

## Contributing

Found a bug or have an improvement? PRs welcome!

1. Fork this repo
2. Create your feature branch (`git checkout -b feature/improvement`)
3. Commit your changes
4. Push and open a PR
