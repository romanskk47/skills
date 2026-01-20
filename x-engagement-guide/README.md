# X Algorithm Reverse-Engineered: A Practical Engagement Guide

A Claude Code skill that provides actionable insights for maximizing organic reach on X (Twitter), based on reverse-engineering the open-sourced recommendation algorithm.

## TL;DR - What Actually Matters

```
BOOST:  Likes, Replies, Reposts, Quotes, Shares, Follows, Dwell Time
KILL:   Blocks, Mutes, Reports, "Not Interested"
DECAY:  Multiple posts from same author
IGNORE: Hashtags, URLs, posting time, post length
```

**One block/mute hurts more than 10 likes help.**

## Key Findings

### The 19 Signals That Determine Your Score

The algorithm predicts probability of each user action:

| Positive (boost) | Negative (kill) |
|------------------|-----------------|
| Like | "Not interested" |
| Reply | Block author |
| Repost | Mute author |
| Quote | Report |
| Profile click | |
| Follow | |
| Photo expand | |
| Video quality view | |
| Share (DM, copy, native) | |
| Dwell time | |

### What the Algorithm Ignores

Based on code analysis, these are **NOT** ranking signals:
- Hashtags
- URLs
- Post length (except via dwell time)
- Account age
- Follower count
- Time of day (only freshness matters)

### The Uncomfortable Truth

The algorithm is designed to be **ungameable**. The transformer learns from raw engagement sequences - no hand-crafted features to exploit.

**What works**: Content people genuinely want to engage with.
**What doesn't work**: Tricks, hacks, optimal posting times, hashtag strategies.

## Installation

### Claude Code CLI

```bash
claude install github:romanskk47/x-engagement-guide
```

### Manual

Copy the `SKILL.md` and `references/` folder to your Claude Code skills directory:

```
~/.claude/skills/x-engagement-guide/
├── SKILL.md
└── references/
    └── algorithm-deep-dive.md
```

## Usage

Once installed, the skill activates automatically when you ask Claude Code about:

- "How do I get more reach on X?"
- "Why aren't my posts getting visibility?"
- "How does the X algorithm work?"
- "Best practices for posting on Twitter"

Example:
```
> How can I improve my X engagement?
```

Claude will provide algorithm-backed recommendations.

## What's Inside

| File | Purpose |
|------|---------|
| `SKILL.md` | Main guide with actionable strategies |
| `references/algorithm-deep-dive.md` | Technical deep-dive into pipeline architecture |

## Algorithm Architecture Overview

```
User Request
     ↓
┌─────────────────────────────────────┐
│         CANDIDATE RETRIEVAL         │
├─────────────────────────────────────┤
│ Thunder (In-Network)                │
│ → Posts from accounts you follow    │
│                                     │
│ Phoenix (Out-of-Network)            │
│ → ML-discovered posts from corpus   │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│            FILTERING                │
├─────────────────────────────────────┤
│ • Blocked/muted authors             │
│ • Muted keywords                    │
│ • Previously seen posts             │
│ • Age filter (24-48h+)              │
│ • Safety/spam classification        │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│             SCORING                 │
├─────────────────────────────────────┤
│ 1. PhoenixScorer                    │
│    → 19 engagement predictions      │
│                                     │
│ 2. WeightedScorer                   │
│    → Combine into final score       │
│                                     │
│ 3. AuthorDiversityScorer            │
│    → Penalize repeated authors      │
│                                     │
│ 4. OONScorer                        │
│    → Handicap out-of-network        │
└─────────────────────────────────────┘
     ↓
   For You Feed
```

## Quick Strategy Cheatsheet

| Do | Don't |
|----|-------|
| Create conversation starters | Rage bait (causes blocks) |
| Post valuable threads | Spam multiple posts (decay) |
| Build genuine followers | Chase virality |
| Space out your posts | Repost stale content |
| Use visuals | Rely on hashtags |

## Source

Based on analysis of X's open-sourced recommendation algorithm codebase.

## License

MIT
