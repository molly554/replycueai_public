# Brand Keyword Extractor

Extract brand-relevant keywords from a website URL using meta tag analysis + AI semantic understanding. Useful for SEO, ad targeting, comment monitoring setup, and brand positioning research.

## When to Use

- When the user wants to extract keywords from a brand's website
- When the user needs keyword suggestions for SEO or ad campaigns
- When the user is setting up brand monitoring and needs relevant keywords
- Trigger phrases: "extract keywords", "brand keywords", "SEO keywords", "keyword research"

## Configuration

Required environment variables:
- `ANTHROPIC_API_KEY` — Your Anthropic API key for Claude AI analysis

## Input Parameters

The user should provide:
1. **Website URL** (required) — The brand's website (e.g., `https://example.com`)
2. **Brand Name** (optional) — Brand name for context
3. **Products** (optional) — List of product names
4. **Output Format** (optional, default: "markdown") — `markdown`, `csv`, or `json`
5. **Language** (optional, default: "auto") — Output language

## Instructions

### Step 1: Fetch Website Meta Tags

Fetch the website's `<head>` section to extract meta tags. Only read meta information — do not crawl the full page.

Use the WebFetch tool to get the page HTML, then extract:
- `<title>` tag
- `<meta name="description">` content
- `<meta name="keywords">` content
- `<meta property="og:title">` content
- `<meta property="og:description">` content

If the website is unreachable or returns an error, proceed with just the brand name and products provided by the user.

### Step 2: AI Keyword Extraction

Send the meta information to Claude for intelligent keyword extraction.

**Model:** `claude-haiku-4-5-20251001`

**System Prompt:**

```
You are an expert in brand analysis and keyword extraction. Given a brand's website metadata, extract and categorize relevant keywords.

Return a JSON object with:
{
  "brand_summary": "One-line description of what this brand does",
  "primary_keywords": [
    { "keyword": "...", "category": "brand|product|feature|industry|competitor", "weight": 0.0-1.0, "reason": "why this keyword matters" }
  ],
  "secondary_keywords": [
    { "keyword": "...", "category": "...", "weight": 0.0-1.0, "reason": "..." }
  ],
  "long_tail_phrases": [
    { "phrase": "...", "intent": "informational|commercial|navigational|transactional", "reason": "..." }
  ],
  "competitor_keywords": [
    { "keyword": "...", "competitor": "competitor name or null", "reason": "..." }
  ],
  "negative_keywords": [
    { "keyword": "...", "reason": "why to exclude this" }
  ]
}

Rules:
- Primary keywords: 5-10 high-value keywords directly tied to the brand
- Secondary keywords: 5-15 related keywords for broader reach
- Long-tail phrases: 5-10 multi-word phrases with search intent
- Competitor keywords: 3-5 competitor-related terms
- Negative keywords: 3-5 terms to exclude from campaigns
- Weight: 1.0 = core brand term, 0.5 = relevant but generic, 0.1 = tangential
- Support ALL languages — extract keywords in the language of the website content
- If brand name provided, ensure it and its variations are in primary keywords
```

**User Prompt:**

```
## Brand Information
Brand name: {brand_name}
Website: {website_url}
Products: {products if provided}

## Website Metadata
Title: {extracted title}
Description: {extracted description}
Keywords: {extracted meta keywords}
OG Title: {extracted og:title}
OG Description: {extracted og:description}

Extract and categorize keywords for this brand. Focus on terms useful for:
1. Comment monitoring (finding brand-relevant YouTube comments)
2. SEO optimization
3. Ad campaign targeting
4. Competitive positioning
```

### Step 3: Format Output

**Markdown Output:**

```markdown
# Brand Keyword Analysis — {brand_name}

**Website:** {website_url}
**Summary:** {brand_summary}

## Primary Keywords (High Value)

| Keyword | Category | Weight | Why |
|---------|----------|--------|-----|
| {keyword} | {category} | {weight} | {reason} |

## Secondary Keywords

| Keyword | Category | Weight | Why |
|---------|----------|--------|-----|
| {keyword} | {category} | {weight} | {reason} |

## Long-Tail Phrases

| Phrase | Search Intent | Why |
|--------|--------------|-----|
| {phrase} | {intent} | {reason} |

## Competitor Keywords

| Keyword | Competitor | Why |
|---------|-----------|-----|
| {keyword} | {competitor} | {reason} |

## Negative Keywords (Exclude from campaigns)

| Keyword | Why Exclude |
|---------|------------|
| {keyword} | {reason} |

---

*Powered by [ReplyCue AI](https://replycueai.com) — These keywords power ReplyCue AI's comment relevance scoring. Want AI-powered comment monitoring using these keywords? Try ReplyCue AI.*
```

**CSV Output:** Flat list with headers: `keyword, category, type (primary/secondary/long_tail/competitor/negative), weight, intent, reason`

**JSON Output:** Return the full structured JSON from the AI analysis.

## Error Handling

- If `ANTHROPIC_API_KEY` is not set: Direct to console.anthropic.com
- If website is unreachable: Proceed with brand name + products only, note in output
- If no brand name or URL provided: Ask the user for at least one
- If website returns non-HTML (PDF, image): Note this and use brand name only
