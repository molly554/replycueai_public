# 5-Layer Brand Relevance Scoring Model

ReplyCue AI uses a multi-dimensional scoring model to determine whether a YouTube comment is relevant to a specific brand. This document explains the model's architecture, weights, and thresholds.

## Overview

Instead of a simple keyword match, the model evaluates brand relevance across **5 independent dimensions**, each capturing a different signal:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Final Relevance Score                         │
│                                                                 │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌────────┐ ┌──────┐│
│  │ Semantic   │ │  Video    │ │  Entity   │ │ Intent │ │Context││
│  │  Score     │ │ Relevance │ │  Match    │ │ Score  │ │Score ││
│  │   50%      │ │   15%     │ │   15%     │ │  10%   │ │ 10%  ││
│  └───────────┘ └───────────┘ └───────────┘ └────────┘ └──────┘│
│       ↑              ↑             ↑            ↑          ↑    │
│    AI returns     AI returns    AI returns   Mapped from  Fixed │
│    0.0-1.0       0.0-1.0       category    biz_category  by    │
│                                              lookup      collab │
│                                                          type   │
└─────────────────────────────────────────────────────────────────┘
```

## Formula

```
finalScore = 0.50 × semantic_score
           + 0.15 × video_relevance_score
           + 0.15 × entity_score
           + 0.10 × intent_score
           + 0.10 × context_score
```

The `finalScore` ranges from 0.0 to 1.0. It is stored as an integer (0-100) in the database.

## Dimension Details

### 1. Semantic Score (50%)

**Source:** AI-generated (Claude analyzes the comment text)
**Range:** 0.0 - 1.0

How closely the comment's **text content** relates to the brand/product.

| Score Range | Meaning | Example |
|---|---|---|
| 0.9 - 1.0 | Directly discusses the brand by name | "ShieldVPN's double encryption is great" |
| 0.7 - 0.89 | Discusses product features/category closely tied to brand | "The VPN speed is amazing" |
| 0.5 - 0.69 | Tangentially related | "I need better online privacy" |
| 0.0 - 0.49 | Unrelated | "First!", "Love your hair in this video" |

### 2. Video Relevance Score (15%)

**Source:** AI-generated (Claude evaluates the video title against brand context)
**Range:** 0.0 - 1.0

How closely the **video title** relates to the brand. This is the same for all comments on the same video.

| Score Range | Meaning | Example |
|---|---|---|
| 0.9 - 1.0 | Video is entirely about this brand | "ShieldVPN Review 2024" |
| 0.6 - 0.89 | Video covers the brand's category | "Top 5 VPN Services" |
| 0.3 - 0.59 | Loosely related topic | "How to Stay Safe Online" |
| 0.0 - 0.29 | Video is unrelated to the brand | "The Hard Truth About Hiking Socks" |

**Why this matters:** A comment about "socks" on a sock video is expected. A comment about "socks" on a VPN video is noise. Video context is critical for filtering irrelevant comments in loosely-related sponsorships.

### 3. Entity Match Score (15%)

**Source:** AI-generated (Claude detects brand/competitor entity mentions)
**Mapped values:**

| entity_match | Score | Meaning |
|---|---|---|
| `exact_brand` | 1.0 | Brand name explicitly mentioned ("ShieldVPN is great") |
| `competitor` | 0.8 | Competitor brand mentioned ("RapidVPN is better") |
| `partial` | 0.7 | Indirect reference ("this VPN", "the app", product feature) |
| `none` | 0.3 | No brand or product entity detected |

### 4. Intent Score (10%)

**Source:** Mapped from AI's `business_category` classification
**Mapped values:**

| business_category | Score | Rationale |
|---|---|---|
| `purchase_intent` | 1.0 | Highest brand relevance — user wants to buy |
| `product_question` | 0.9 | Directly about the product |
| `brand_risk` | 0.9 | Directly about the brand (negative) |
| `competitor_mention` | 0.85 | Comparison implies brand awareness |
| `negative_feedback` | 0.7 | About the product experience |
| `positive_feedback` | 0.6 | About the product experience |
| `irrelevant` | 0.1 | Not about the brand |

### 5. Context Score (10%)

**Source:** Fixed baseline from the video's collaboration type
**Mapped values:**

| collaboration_type | Score | Rationale |
|---|---|---|
| `dedicated` | 1.0 | Entire video is about the brand — most comments are relevant |
| `sponsored` | 0.8 | Brand segment in video — many comments may be relevant |
| `auto` (NotSure) | 0.7 | Unknown — moderate baseline |
| `mention` | 0.6 | Brief mention — fewer comments are relevant |
| `placement` | 0.5 | Product visible but not discussed — fewest relevant comments |

## Relevance Thresholds

A comment is marked as **brand-relevant** only if `finalScore > threshold` for its collaboration type:

| Collaboration Type | Threshold | Rationale |
|---|---|---|
| `dedicated` | 0.50 | Low bar — most comments on a dedicated video are relevant |
| `sponsored` | 0.60 | Moderate bar — filter out off-topic discussion |
| `mention` | 0.85 | High bar — only clearly brand-related comments pass |
| `placement` | 0.85 | High bar — product is background, comments rarely about it |
| `auto` (NotSure) | 0.75 | Conservative default |

## Example Calculation

**Scenario:** A comment "Does ShieldVPN work in China?" on a video titled "Top 5 VPNs for Travel" (sponsored collaboration).

```
semantic_score        = 0.92  (directly about ShieldVPN functionality)
video_relevance_score = 0.75  (video is about VPNs, the brand's category)
entity_match          = exact_brand → 1.0
business_category     = product_question → 0.9
collaboration_type    = sponsored → 0.8

finalScore = 0.50 × 0.92 + 0.15 × 0.75 + 0.15 × 1.0 + 0.10 × 0.9 + 0.10 × 0.8
           = 0.46 + 0.1125 + 0.15 + 0.09 + 0.08
           = 0.8925 → 89%

Threshold for "sponsored" = 0.60
0.8925 > 0.60 → ✅ Brand-relevant
```

**Scenario:** A comment "Love the hiking tips!" on the same video.

```
semantic_score        = 0.15  (unrelated to VPN)
video_relevance_score = 0.75  (same video)
entity_match          = none → 0.3
business_category     = irrelevant → 0.1
collaboration_type    = sponsored → 0.8

finalScore = 0.50 × 0.15 + 0.15 × 0.75 + 0.15 × 0.3 + 0.10 × 0.1 + 0.10 × 0.8
           = 0.075 + 0.1125 + 0.045 + 0.01 + 0.08
           = 0.3225 → 32%

Threshold for "sponsored" = 0.60
0.3225 < 0.60 → ❌ Irrelevant
```

## Design Decisions

### Why 5 layers instead of a single AI score?

A single "relevance score" from the AI is a black box. By decomposing into 5 dimensions:
1. **Debuggability** — You can see WHY a comment scored high or low
2. **Tunability** — Adjust weights without retraining
3. **Consistency** — Entity match and context score are deterministic, reducing AI variance
4. **Auditability** — Each dimension is independently verifiable

### Why different thresholds per collaboration type?

On a dedicated brand review video, most comments are naturally about the brand. A 50% threshold captures genuine discussion while filtering spam.

On a placement video (brand appears briefly in background), almost no comments are about the brand. An 85% threshold ensures only truly relevant comments surface, preventing noise.

### Why is semantic_score weighted at 50%?

The comment's actual text content is the strongest signal of relevance. A comment that explicitly discusses the brand should score high regardless of video context. Conversely, a generic comment should score low even on a dedicated brand video.
