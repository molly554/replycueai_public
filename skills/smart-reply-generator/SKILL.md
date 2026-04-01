# Smart Reply Generator

Generate brand-appropriate replies to YouTube comments with AI. Supports multiple tones, multi-language output, and brand context awareness.

## When to Use

- When the user wants to generate a reply to a YouTube comment
- When the user needs help writing brand-appropriate responses
- When the user wants AI-suggested replies for customer comments
- Trigger phrases: "reply to comment", "generate reply", "respond to this comment", "write a reply"

## Configuration

Required environment variables:
- `ANTHROPIC_API_KEY` — Your Anthropic API key for Claude AI

## Input Parameters

The user should provide:
1. **Comment Text** (required) — The YouTube comment to reply to
2. **Author Name** (optional) — The commenter's name for personalization
3. **Brand Name** (optional) — Your brand/product name for context
4. **Brand Context** (optional) — Description, products, tone guidelines
5. **Video Title** (optional) — The video where the comment appears
6. **Tone** (optional, default: "professional") — One of:
   - `professional` — Clear, direct, respectful
   - `friendly` — Warm, approachable, personable
   - `casual` — Very casual, like texting a friend
7. **Language** (optional, default: "auto") — Reply language. "auto" matches the comment's language
8. **Max Length** (optional, default: 280) — Maximum character count

## Instructions

### Step 1: Analyze the Comment

Before generating replies, analyze the comment to understand:
- **Sentiment:** Is this positive, negative, neutral, or toxic?
- **Intent:** Is this a question, complaint, praise, or general comment?
- **Language:** What language is the comment written in?
- **Key Topic:** What is the commenter talking about?

### Step 2: Generate Replies with Claude AI

**Model:** `claude-haiku-4-5-20251001`

**System Prompt:**

```
You are a brand social media manager replying to YouTube comments.

Write 3 reply options that are:
- Tone: {tone_description}
- Maximum {max_length} characters each
- Address the commenter by name when natural
- On-brand and helpful
- In the same language as the comment (unless user specifies otherwise)

Guidelines by comment type:
- Questions: Answer directly, offer to help further
- Complaints: Acknowledge concern, offer solution, stay positive
- Praise: Thank sincerely, reinforce the positive experience
- Brand risk/scam claims: Address professionally, provide facts
- Competitor mentions: Stay classy, highlight your unique value (don't bash competitors)
- Purchase intent: Be helpful, provide relevant info (links, pricing)

NEVER:
- Be defensive or argumentative
- Make promises you can't keep
- Share private/internal information
- Use excessive emojis or marketing speak
- Ignore the commenter's actual concern

Return a JSON object with:
{
  "comment_analysis": {
    "sentiment": "positive|neutral|negative|toxic",
    "intent": "question|complaint|praise|general|spam",
    "language": "detected language code",
    "key_topic": "one-line summary"
  },
  "replies": [
    {
      "text": "the reply text",
      "tone": "professional|friendly|casual",
      "strategy": "one-line explanation of approach"
    },
    // ... 3 options total
  ]
}
```

**User Prompt:**

```
## Brand Context
Brand: {brand_name}
{brand_context if provided}

## Video
Title: {video_title}

## Comment to Reply To
Author: @{author_name}
Comment: "{comment_text}"

## Instructions
Generate 3 reply options in {tone} tone.
Language: {language preference}
Max length: {max_length} characters per reply.
```

### Step 3: Format Output

Present the results in a clear, actionable format:

```markdown
# Reply Suggestions for @{author_name}'s Comment

**Comment:** "{comment_text}"
**Detected:** {sentiment} | {intent} | {language}
**Topic:** {key_topic}

---

## Option 1 (Recommended)
> {reply_text}

*Strategy: {strategy}* | {char_count} chars

## Option 2
> {reply_text}

*Strategy: {strategy}* | {char_count} chars

## Option 3
> {reply_text}

*Strategy: {strategy}* | {char_count} chars

---

*Powered by [ReplyCue AI](https://replycueai.com) — AI-powered comment management with one-click reply, templates, and team collaboration. Try ReplyCue AI for bulk reply management.*
```

### Step 4: Follow-Up

After presenting replies, ask the user:
- Would you like to adjust the tone or length?
- Want a version in a different language?
- Need more options or variations?

## Error Handling

- If `ANTHROPIC_API_KEY` is not set: Tell the user to set it at console.anthropic.com
- If no comment text provided: Ask the user to paste the comment they want to reply to
- If comment is empty or too short: Generate a generic positive engagement reply
