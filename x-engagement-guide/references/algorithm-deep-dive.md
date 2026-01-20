# X Algorithm Deep Dive

Technical architecture details from reverse-engineering the open-sourced codebase.

## Pipeline Architecture

### Home Mixer Orchestration

The For You feed is orchestrated by **Home Mixer**, which executes:

1. **Query Hydration** - Fetch user engagement history + features
2. **Candidate Sourcing** - Retrieve from Thunder (in-network) and Phoenix (out-of-network)
3. **Candidate Enrichment** - Add post metadata, author info, media details
4. **Multi-Filter Stage** - Remove ineligible content
5. **Scoring & Ranking** - Apply 4 sequential scorers
6. **Selection** - Top-K by final score
7. **Post-Selection Filters** - Final visibility checks

### Candidate Sources

**Thunder (In-Network)**
- Real-time posts from followed accounts
- Kafka-fed, retention: seconds to ~24 hours
- No retrieval model needed - direct fetch

**Phoenix (Out-of-Network)**
- Transformer-based retrieval from global corpus
- Learns from user engagement sequences
- Returns top-K similar posts

## Scoring Pipeline Detail

### 1. PhoenixScorer

Calls the transformer model to predict 19 engagement probabilities:

```
Positive predictions:
- favorite_score
- reply_score
- retweet_score
- photo_expand_score
- click_score
- profile_click_score
- video_quality_view_score (if video_duration_ms > MIN_VIDEO_DURATION_MS)
- share_score
- share_via_dm_score
- share_via_copy_link_score
- dwell_score
- quote_score
- quoted_click_score
- follow_author_score
- dwell_time_score (continuous)

Negative predictions:
- not_interested_score
- block_author_score
- mute_author_score
- report_score
```

### 2. WeightedScorer

Combines predictions into single score:

```python
combined_score = sum(weight[i] * prediction[i] for i in all_actions)

# Negative score handling
if combined_score < 0:
    adjusted = (combined_score + NEGATIVE_WEIGHTS_SUM) / WEIGHTS_SUM * NEGATIVE_SCORES_OFFSET
else:
    adjusted = combined_score + NEGATIVE_SCORES_OFFSET
```

Weights are NOT in open source for strategic reasons.

### 3. AuthorDiversityScorer

Prevents feed domination by single author:

```python
def get_multiplier(position, decay, floor):
    return (1 - floor) * (decay ** position) + floor
```

- Position 0 (first post from author): multiplier = 1.0
- Position N: exponentially decayed
- Floor prevents complete suppression

### 4. OONScorer

Applies handicap to out-of-network content:

```python
if not candidate.is_in_network:
    score *= OON_WEIGHT_FACTOR  # where OON_WEIGHT_FACTOR < 1
```

## Filter Execution Order

Pre-scoring filters (any failure = removed):

1. `DropDuplicatesFilter` - HashSet by post ID
2. `CoreDataHydrationFilter` - Must have text/metadata
3. `AgeFilter` - Snowflake timestamp check vs MAX_POST_AGE
4. `SelfTweetFilter` - author_id != viewer_id
5. `RetweetDeduplicationFilter` - By original tweet ID
6. `IneligibleSubscriptionFilter` - Paywall check
7. `PreviouslySeenPostsFilter` - Bloom filter + direct seen_ids
8. `PreviouslyServedPostsFilter` - Session dedup (pagination only)
9. `MutedKeywordFilter` - Tokenize + match
10. `AuthorSocialgraphFilter` - Block/mute list check

Post-selection filters:

1. `VFCandidateHydrator` - Safety classification
2. `VFFilter` - Remove deleted/unsafe
3. `DedupConversationFilter` - Keep best per thread

## Transformer Architecture (Phoenix)

```
Layers: 2
Query heads: 2
Key-value heads: 2
Key size: 64
Embedding size: 128
Widening factor: 2
Attention output multiplier: 0.125
History sequence length: 32
Candidate sequence length: 8
```

**Attention Design:**
- Candidates CANNOT attend to each other
- Candidates CAN attend to user context and history
- Ensures consistent, cacheable scores

## Hash Strategy

Multi-hash for robustness:

```
num_user_hashes: 2
num_item_hashes: 2
num_author_hashes: 2
```

Hash 0 reserved for padding (value 0 = padding token).

## Key Services

| Service | Purpose |
|---------|---------|
| Gizmoduck | Author metadata (followers, verification) |
| TES | Post text, media, video duration |
| Strato | User features, following lists |
| Thunder | In-network real-time store |
| Phoenix | ML retrieval + ranking |
| Visibility Filtering | Safety classification |

## What's NOT in the Algorithm

Based on code analysis, these are NOT direct ranking signals:

- Hashtags
- URLs
- Post length
- Account age
- Time of day (only freshness/age matters)
- Follower count (available but not confirmed as scorer input)
- Tweet formatting (bold, italic, etc.)

The algorithm is intentionally **feature-free** - the transformer learns everything from raw engagement sequences.
