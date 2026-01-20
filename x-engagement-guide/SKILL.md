---
name: x-engagement-guide
description: |
  Practical guide for maximizing organic engagement on X (Twitter) based on reverse-engineering the open-sourced recommendation algorithm. Use this skill when users ask about: (1) How to get more reach/engagement on X/Twitter, (2) How the X algorithm works, (3) Content strategy for X, (4) Why posts aren't getting visibility, (5) Best practices for posting on X, (6) Understanding the For You feed algorithm.
---

# X Engagement Guide

Actionable strategies derived from reverse-engineering X's open-sourced recommendation algorithm.

## How the Algorithm Actually Works

The For You feed uses a **two-stage pipeline**:

1. **Candidate Retrieval**: Pulls posts from people you follow (Thunder/In-Network) + ML-discovered posts (Phoenix/Out-of-Network)
2. **Ranking**: Transformer model predicts 19 engagement probabilities, combines them with weights into final score

**Key insight**: The algorithm predicts what YOU specifically will engage with based on YOUR history. There's no universal "algorithm hack" - it's personalized prediction.

## The 19 Signals That Determine Your Score

The algorithm predicts probability of each action:

**Positive signals (boost visibility):**
- Like, Reply, Repost, Quote
- Profile click, Follow author
- Photo expand, Video quality view
- Share (DM, copy link, native)
- Dwell time (how long someone reads)

**Negative signals (kill visibility):**
- "Not interested" clicks
- Block author, Mute author
- Report

**Critical**: One block/mute/report can tank a post's score significantly. The weights for negative actions are high.

## What Actually Matters for Reach

### 1. In-Network Advantage (~15% of ranking)

Posts from accounts someone follows get priority. Out-of-network content is multiplied by a factor <1.

**Action**: Building genuine followers matters more than going viral. Your followers see you; non-followers face a handicap.

### 2. Engagement History Alignment (~60% of ranking)

The transformer learns from user's last 32 engagement actions. If they like long threads, they see more. If they reply to debates, they get debates.

**Action**: Define your content type clearly. Mixed signals = lower relevance scores across all audience segments.

### 3. Author Diversity Penalty

Multiple posts from same author in one session get exponentially downweighted:
```
score_multiplier = (1 - floor) × decay^position + floor
```

**Action**: Quality over quantity. Posting 10 times won't give 10x reach - it may give 3-4x at best due to decay.

### 4. Freshness Filter

Posts older than threshold (likely 24-48h) get hard-filtered.

**Action**: Timing matters. Dead posts don't resurrect.

### 5. Negative Action Sensitivity

The scoring formula treats negatives specially:
```
if combined_score < 0:
    adjusted = (score + NEGATIVE_SUM) / WEIGHTS_SUM × OFFSET
```

**Action**: Avoid content that triggers blocks/mutes/reports. Controversial ≠ engaging if it causes negative actions.

## Actionable Strategy Framework

### Content That Scores High

| Content Type | Why It Works |
|--------------|--------------|
| **Conversation starters** | Replies + quotes both scored positively |
| **Thread openers with value** | Dwell time + click-through both measured |
| **Visual content** | Photo expand + video quality view are distinct signals |
| **Relationship builders** | Profile clicks + follows heavily weighted |

### Content That Scores Low

| Content Type | Why It Fails |
|--------------|--------------|
| **Rage bait** | May get replies but also blocks/mutes |
| **Spam-like frequency** | Author diversity decay kills reach |
| **Generic content** | Low relevance to any specific user segment |
| **Stale reposts** | Freshness filter + seen-before filter |

### Optimal Posting Strategy

1. **Segment your audience mentally**: The algorithm segments for you based on engagement patterns
2. **Maximize positive actions per impression**: Aim for likes + replies + shares, not just one
3. **Minimize negative actions**: One mute costs more than multiple likes gain
4. **Build in-network first**: Followers see you reliably; discovery is handicapped
5. **Space out posts**: Beat the author diversity decay
6. **Post fresh insights**: Freshness filter is binary - old = invisible

## What the Algorithm Ignores

Based on code analysis, these don't appear in ranking:
- Hashtag count or presence
- URL presence (no boost or penalty in scoring)
- Post length (except indirectly via dwell time)
- Account age or tenure
- Follower count (available but not confirmed as direct signal)
- Time of day (only freshness matters)

## Debugging Low Reach

Check these filters that hard-remove posts:

1. **Muted keywords**: Does your content contain words your audience mutes?
2. **Previously seen**: Reposting same content gets filtered
3. **Author socialgraph**: Are you blocked/muted by potential viewers?
4. **Age filter**: Is the content too old?
5. **Visibility filtering**: Is content flagged as unsafe/spam?

## The Uncomfortable Truth

The algorithm is designed to be **ungameable** at the weight level. You can see the 19 signals but not their relative weights. The transformer learns user preferences from raw engagement sequences - no hand-crafted features to exploit.

**What works**: Genuine value that people want to engage with positively.
**What doesn't work**: Tricks, hacks, optimal posting times, hashtag strategies.

The algorithm is essentially asking: "Will THIS specific user want to engage with THIS specific post based on what they've engaged with before?" If yes, you rank. If no, you don't.

## Quick Reference

```
BOOST: Likes, Replies, Reposts, Quotes, Shares, Follows, Dwell Time
KILL:  Blocks, Mutes, Reports, "Not Interested"
DECAY: Multiple posts from same author
FILTER: Old posts, Seen posts, Muted keywords, Blocked authors
```

For detailed technical analysis of the algorithm architecture, see [references/algorithm-deep-dive.md](references/algorithm-deep-dive.md).
