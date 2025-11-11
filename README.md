## Information Retrieval Combining Review Usefulness and Relevance (IR‑Focused)

### Project Objective
Deliver a lightweight information-retrieval pipeline that surfaces the most useful and relevant product reviews for a user’s intent in real time. Users issue attribute-focused queries (e.g., “battery life”, “sound quality”), and the system returns high-quality snippets with direct links back to the original review pages so shoppers can verify context immediately.

### Real-Time IR & Recommendation Pipeline
1. **Document representation**: Encode review text (optionally title/brand) with BM25; support per-field weights so concise titles or spec bullet points can influence matching.
2. **Indexing**: Maintain an inverted index that is refreshable as new reviews arrive; persist metadata such as likes, has_image, created_at, and canonical review URLs for link-out.
3. **Query processing**: Normalize text (lowercase, punctuation stripping, stopword removal), apply English tokenization or Chinese segmentation, and expand with a small synonym list (e.g., “battery life” ↔ “续航”). Log user queries to support real-time analytics.
4. **Candidate retrieval**: Use BM25 to pull top-k candidates quickly; the index can be updated incrementally to keep latency low even when new reviews stream in.
5. **Ranking for usefulness**: Blend lexical relevance with a usefulness prior so the ranking favors reviews that both match the query and have strong social proof. The result set includes structured metadata (likes, timestamp, link) for downstream presentation or API clients.
6. **Filtering and delivery**: Enforce thresholds on likes, word count, sentiment polarity, or freshness. Final payload includes review text extract, computed scores, and the canonical link for user navigation.

### Scoring Details
- **Lexical score (BM25)**
  - Parameters: `k1` (term-frequency saturation) and `b` (length normalization). For query `q` and document `d`:

```
BM25(q, d) = Σ_{t∈q} IDF(t) · ( tf(t,d) · (k1 + 1) ) / ( tf(t,d) + k1 · (1 − b + b · |d|/avgdl) )
IDF(t)     = ln( (N − df(t) + 0.5) / (df(t) + 0.5) + 1 )
```

  - `N`: number of reviews; `df(t)`: review count containing term `t`; `|d|`: document length; `avgdl`: average length. With multiple fields, compute BM25 per field and linearly combine (e.g., title 1.5, brand 1.2, review_text 1.0).

- **Usefulness score**
  - Normalize observable signals to the range `[0,1]`:

```
likes_norm      = log1p(likes) / log1p(max_likes_window)
wordcount_norm  = min(word_count / max_words_cap, 1.0)
has_image_norm  = 1.0 if has_image else 0.0
recency_norm    = max(0, 1 − days_since_posted / freshness_horizon)

Usefulness(d) = α·likes_norm + β·wordcount_norm + γ·has_image_norm + δ·recency_norm
```

  - Typical weights: `α=0.4`, `β=0.25`, `γ=0.15`, `δ=0.2`. Tune per category; cap `max_words_cap` (e.g., 220) to prevent excessively long reviews dominating.

- **Final score fusion**

```
FinalScore(q, d) = λ · BM25(q, d) + (1 − λ) · Usefulness(d)   , 0 ≤ λ ≤ 1
```

  - Start with `λ ≈ 0.75` to emphasize textual relevance, then adjust using validation metrics such as precision@k or click-through rate on recommended links.

### Real-Time Data Sources & Compliance
- **Chinese e-commerce public pages (JD/Tmall)**: capture newly posted reviews with official APIs or compliant scraping; store text, likes, media flags, timestamps, and the direct review URL for link-outs. Respect robots.txt, rate limits, and legal constraints.
- **Social platforms (Reddit, Twitter/X, Weibo)**: when sourcing supplemental opinion data, rely on official APIs for up-to-date comments. Maintain attribution and include canonical links so users can inspect the original context. Follow platform policies and regional regulations.

Author: nikk909 (yinghua253659@163.com)
