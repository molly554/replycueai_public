# Comment Threat Scanner

Scan YouTube video comments for brand safety threats: scam links, false claims, KOL misrepresentation, phishing, toxic content, and competitive attacks. Lightweight and focused — designed for quick brand risk assessment.

## When to Use

- When the user wants to check a video's comments for brand risks
- When the user needs a quick safety scan before responding to comments
- When the user wants to identify scam, phishing, or toxic comments
- Trigger phrases: "scan for threats", "brand safety", "comment threats", "risk scan", "scam detection"

## Configuration

Required environment variables:
- `ANTHROPIC_API_KEY` — Your Anthropic API key for Claude AI analysis
- `YOUTUBE_API_KEY` — Your YouTube Data API v3 key for fetching comments

## Input Parameters

The user should provide:
1. **YouTube Video URL** (required) — The video to scan
2. **Brand Name** (required) — The brand to scan threats for
3. **Scan Depth** (optional, default: "standard") — One of:
   - `quick` — Top 50 comments by relevance
   - `standard` — Top 100 comments
   - `deep` — Up to 500 comments (all available)
4. **Output Format** (optional, default: "markdown") — `markdown` or `json`
5. **Language** (optional, default: "auto") — Report language

## Instructions

### Step 1: Fetch Comments

Extract the video ID and fetch comments via YouTube Data API:

```bash
# Fetch by relevance (threats often get engagement)
curl -s "https://www.googleapis.com/youtube/v3/commentThreads?part=snippet&videoId=${VIDEO_ID}&maxResults={scan_depth}&order=relevance&key=${YOUTUBE_API_KEY}"
```

Also fetch video metadata for context.

### Step 2: Threat Analysis with Claude

Send comments in batches of 20 to Claude for focused threat detection.

**Model:** `claude-haiku-4-5-20251001`

**System Prompt:**

```
You are a brand safety expert specializing in detecting threats in YouTube comment sections. Your job is to identify comments that pose risks to the brand.

## Threat Categories

### HIGH SEVERITY (immediate action needed)
- **scam**: Links to phishing sites, fake giveaways, "click here to win" schemes
- **impersonation**: Accounts pretending to be the brand or KOL
- **false_claim**: Fabricated claims about the product (safety issues, lawsuits, etc.)
- **doxxing**: Revealing private information about brand employees or KOL
- **hate_speech**: Targeted harassment related to the brand

### MEDIUM SEVERITY (review needed)
- **kol_misrepresentation**: Claims the KOL is lying about the product, paid shill accusations
- **privacy_concern**: Legitimate privacy/data collection concerns about the product
- **legal_threat**: Threats of lawsuits or legal action
- **competitor_attack**: Organized negative campaign by competitor fans
- **misinformation**: Incorrect product information that could mislead buyers

### LOW SEVERITY (monitor)
- **negative_experience**: Genuine bad experiences (complaints, bugs)
- **toxic_general**: General toxicity not specifically targeting the brand
- **spam**: Generic spam not specifically harmful to the brand

## Analysis Instructions

For each comment, determine:
1. Is this a threat? (yes/no)
2. If yes: threat_category, severity (high/medium/low), confidence (0.0-1.0)
3. threat_description: What specifically is the threat?
4. evidence: Quote the threatening content (max 80 chars)
5. recommended_action: "auto_hide" | "escalate" | "reply_professionally" | "monitor" | "report_to_youtube"
6. urgency: "immediate" | "within_24h" | "when_convenient"

IMPORTANT: Analyze in the comment's original language. Support all languages.
Only flag REAL threats — do not over-flag legitimate criticism or negative feedback.

Return a JSON object:
{
  "threats": [
    {
      "comment_id": "...",
      "author": "...",
      "text": "...",
      "threat_category": "...",
      "severity": "high|medium|low",
      "confidence": 0.0-1.0,
      "threat_description": "...",
      "evidence": "...",
      "recommended_action": "...",
      "urgency": "..."
    }
  ],
  "safe_count": number,
  "summary": "one paragraph overview of the threat landscape"
}

If NO threats are found, return an empty threats array with a positive summary.
```

**User Prompt:**

```
## Brand Context
Brand: {brand_name}
Video: {video_title}

## Comments to Scan
{comments as numbered list}

Scan for brand safety threats. Be precise — flag real risks, not general negativity.
```

### Step 3: Format Threat Report

**Markdown Output:**

```markdown
# Brand Safety Scan — {brand_name}

**Video:** [{video_title}]({video_url})
**Scanned:** {total_comments} comments
**Scan Depth:** {scan_depth}
**Date:** {date}

## Threat Summary

| Severity | Count | Action Required |
|----------|-------|----------------|
| HIGH | {high_count} | Immediate |
| MEDIUM | {medium_count} | Review within 24h |
| LOW | {low_count} | Monitor |
| **Safe** | **{safe_count}** | — |

{if no threats:}
### All Clear
No brand safety threats detected in {total_comments} comments. The comment section appears safe for the brand.
{end if}

{if threats found:}

## HIGH Severity Threats

{for each high severity threat:}
### @{author} — {threat_category}
> {comment_text}

- **Threat:** {threat_description}
- **Evidence:** "{evidence}"
- **Action:** {recommended_action} | **Urgency:** {urgency}
- **Confidence:** {confidence}%

---

## MEDIUM Severity Threats

{same format}

## LOW Severity Threats

{same format}

{end if}

## AI Assessment

{narrative summary of threat landscape}

## Recommended Actions

{numbered list of specific actions to take}

---

*Powered by [ReplyCue AI](https://replycueai.com) — Get real-time threat alerts, auto-hide dangerous comments, and protect your brand 24/7. Try ReplyCue AI for continuous brand safety monitoring.*
```

## Error Handling

- If `YOUTUBE_API_KEY` is not set: Provide Google Cloud Console setup instructions
- If `ANTHROPIC_API_KEY` is not set: Direct to console.anthropic.com
- If video has no comments or comments disabled: Report as safe
- If video URL is invalid: Ask for valid URL
