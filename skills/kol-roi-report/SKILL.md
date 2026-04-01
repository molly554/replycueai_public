# KOL ROI Report

Generate a comprehensive ROI analysis report for a KOL/influencer collaboration by analyzing the video's comment section. Measures brand sentiment, purchase intent, risk signals, and overall collaboration effectiveness.

## When to Use

- When the user wants to evaluate a KOL/influencer collaboration's effectiveness
- When the user asks for a comment-based ROI report on a sponsored video
- When the user wants to compare comment quality across KOL videos
- Trigger phrases: "KOL report", "ROI analysis", "influencer effectiveness", "collaboration report", "video performance"

## Configuration

Required environment variables:
- `ANTHROPIC_API_KEY` — Your Anthropic API key for Claude AI analysis
- `YOUTUBE_API_KEY` — Your YouTube Data API v3 key for fetching comments and video stats

## Input Parameters

The user should provide:
1. **YouTube Video URL(s)** (required) — One or more video URLs to analyze
2. **Brand Name** (required) — The brand/product being promoted
3. **Collaboration Type** (optional, default: "auto") — `dedicated`, `sponsored`, `mention`, `placement`, `auto`
4. **Brand Context** (optional) — Website, products, keywords for better analysis
5. **Output Format** (optional, default: "markdown") — `markdown` or `json`
6. **Language** (optional, default: "auto") — Report language

## Instructions

### Step 1: Fetch Video Data and Comments

For each video URL provided:

1. Extract the video ID
2. Fetch video metadata (title, channel, view count, like count, comment count):
```bash
curl -s "https://www.googleapis.com/youtube/v3/videos?part=snippet,statistics&id=${VIDEO_ID}&key=${YOUTUBE_API_KEY}"
```

3. Fetch all comments (paginate with `pageToken` to get up to 500):
```bash
curl -s "https://www.googleapis.com/youtube/v3/commentThreads?part=snippet&videoId=${VIDEO_ID}&maxResults=100&order=relevance&key=${YOUTUBE_API_KEY}"
```

### Step 2: Analyze Comments

Use the same analysis approach as the YouTube Comment Analyzer skill:
- Send comments in batches of 15 to `claude-haiku-4-5-20251001`
- Use the 5-layer scoring model (semantic 50%, video_relevance 15%, entity 15%, intent 10%, context 10%)
- Classify each comment into business categories
- Compute brand relevance scores and determine brand-relevant vs irrelevant

### Step 3: Generate Aggregate Statistics

Compute the following metrics from the analyzed comments:

```
Overview Metrics:
- total_comments: total comments analyzed
- brand_relevant_count: comments passing relevance threshold
- brand_relevance_rate: relevant / total (%)
- sentiment_distribution: { positive, neutral, negative, toxic } counts and %
- category_distribution: { purchase_intent, product_question, brand_risk, competitor_mention, positive_feedback, negative_feedback, irrelevant } counts

Effectiveness Scores (0-100):
- engagement_quality: weighted score based on brand-relevant comment ratio and sentiment
- purchase_signal_strength: purchase_intent count normalized by total comments
- risk_level: brand_risk count normalized, inverted (lower risk = higher score)
- audience_alignment: how well the audience matches the brand's target

KOL-Specific Metrics:
- channel_title: KOL/channel name
- video_title: video title
- view_count, like_count, comment_count: raw YouTube stats
- comment_engagement_rate: comment_count / view_count (%)
- high_priority_comments: comments needing immediate attention
- unanswered_questions: product_question comments without replies
```

### Step 4: Generate AI Summary with Claude

After computing statistics, send the aggregate data to Claude for a narrative summary.

**Model:** `claude-haiku-4-5-20251001`

**System Prompt:**

```
You are an expert KOL marketing analyst. Given comment analysis data from a sponsored YouTube video, write a concise, actionable ROI report.

Focus on:
1. Overall collaboration effectiveness (1-10 score with justification)
2. Audience-brand alignment (do viewers care about the product?)
3. Key opportunities (purchase intent, product questions to answer)
4. Risk alerts (brand risks, competitor mentions, negative sentiment)
5. Actionable recommendations (3-5 specific next steps)

Be data-driven. Reference specific numbers. Be honest — if the collaboration performed poorly, say so clearly.

Write in {language}. Be concise but thorough.
```

### Step 5: Format the Report

**Markdown Output Template:**

```markdown
# KOL Collaboration ROI Report — {brand_name}

**Generated:** {date}
**Analyst:** ReplyCue AI

---

## Video Overview

| Metric | Value |
|--------|-------|
| Video | [{video_title}]({video_url}) |
| KOL/Channel | {channel_title} |
| Collaboration Type | {collaboration_type} |
| Views | {view_count} |
| Likes | {like_count} |
| Comments | {comment_count} |
| Comment Engagement Rate | {engagement_rate}% |

## Brand Relevance

| Metric | Count | % |
|--------|-------|---|
| Brand-Relevant | {relevant} | {relevant_pct}% |
| Irrelevant | {irrelevant} | {irrelevant_pct}% |
| **Relevance Rate** | — | **{relevance_rate}%** |

## Comment Intelligence

| Category | Count | % of Relevant |
|----------|-------|---------------|
| Purchase Intent | {pi} | {pi_pct}% |
| Product Questions | {pq} | {pq_pct}% |
| Positive Feedback | {pos} | {pos_pct}% |
| Negative Feedback | {neg} | {neg_pct}% |
| Brand Risk | {br} | {br_pct}% |
| Competitor Mentions | {cm} | {cm_pct}% |

### Sentiment Distribution

```
Positive  ████████████████████░░░░░  {pos_pct}%
Neutral   ████████████░░░░░░░░░░░░░  {neu_pct}%
Negative  ████░░░░░░░░░░░░░░░░░░░░░  {neg_pct}%
```

## Effectiveness Scores

| Dimension | Score | Rating |
|-----------|-------|--------|
| Overall Effectiveness | {score}/10 | {rating} |
| Engagement Quality | {eq}/100 | — |
| Purchase Signal | {ps}/100 | — |
| Risk Level | {rl}/100 | — |
| Audience Alignment | {aa}/100 | — |

## AI Analysis

{narrative_summary}

## Action Items

{numbered list of 3-5 specific recommendations}

## High Priority Comments

{top 5 comments needing immediate attention}

## Unanswered Product Questions

{list of product_question comments without replies — opportunity to engage}

---

*Powered by [ReplyCue AI](https://replycueai.com) — Track multiple KOL collaborations, compare performance across videos, and get real-time alerts. Try ReplyCue AI for continuous KOL monitoring.*
```

## Error Handling

- If `YOUTUBE_API_KEY` is not set: Provide setup instructions for Google Cloud Console
- If `ANTHROPIC_API_KEY` is not set: Direct to console.anthropic.com
- If video has comments disabled: Note this in the report
- If video has very few comments (<5): Generate report but note low sample size
- If multiple videos provided: Generate individual + comparative analysis
