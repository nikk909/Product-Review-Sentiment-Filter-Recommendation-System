## Information Retrieval Combining Review Usefulness and Relevance (IR‑Focused)

### Project Objective
Build a  Information Retrieval (IR) pipeline over product reviews (CSV). Users enter attribute‑style queries (e.g., “battery life”, “sound quality”) and receive the most relevant, high‑value reviews.

### IR Pipeline 
1) Document representation: represent review text (optionally title/brand) with BM25 vectors; allow per‑field weights.
2) Indexing: build an inverted index/sparse structures and store metadata (likes, has_image, created_at).
3) Query processing: lowercasing, punctuation stripping, stopword removal; English tokenization / Chinese segmentation; simple synonyms (battery life ↔ 续航).
4) Candidate retrieval: use BM25 to retrieve top‑k candidate reviews.
5) Ranking fusion: FinalScore = λ·LexicalScore + (1−λ)·Usefulness, where Usefulness is a normalized linear mix of log(1+likes), word_count, has_image; λ is tunable.
6) Filtering: optional sentiment filter and usefulness thresholds (min_likes, min_words, has_image).



### Scoring Details
- Lexical score (BM25)
  - We use BM25 with parameters k1 (term frequency saturation) and b (length normalization). For a query q and document d:

```text
BM25(q, d) = Σ_{t ∈ q} IDF(t) · ( tf(t,d) · (k1 + 1) ) / ( tf(t,d) + k1 · (1 − b + b · |d|/avgdl) )
IDF(t)     = ln( (N − df(t) + 0.5) / (df(t) + 0.5) + 1 )
```

  - Where N is the number of documents, df(t) is document frequency of term t, |d| is document length, avgdl is average document length.
  - Field weighting (optional): if using multiple fields, compute BM25 per field and linearly combine with field weights (e.g., title 1.5, brand 1.2, review_text 1.0).

- Usefulness score
  - We combine simple, explainable features and normalize them to [0,1]:

```text
likes_norm      = log1p(likes) / log1p(max_likes)
wordcount_norm  = min(word_count / max_words_cap, 1.0)
has_image_norm  = 1.0 if has_image else 0.0

Usefulness(d) = α · likes_norm + β · wordcount_norm + γ · has_image_norm
```

  - Typical weights: α=0.5, β=0.3, γ=0.2 (tunable). Choose max_words_cap (e.g., 200) to avoid over‑rewarding extremely long reviews.
  - Optionally add recency: recency_norm ∈ [0,1] and extend the linear mix.

- Final score fusion

```text
FinalScore(q,d) = λ · BM25(q,d) + (1 − λ) · Usefulness(d)   with 0 ≤ λ ≤ 1
```

  - Start with λ≈0.8 to prioritize lexical relevance, then tune based on validation (precision@k).

### Data Sources & Compliance
- Kaggle Amazon Product Reviews (Small): small, well‑structured review set suitable for IR prototyping.
- Chinese e‑commerce public pages (JD/Tmall): collect only where permitted; store text, likes, images, timestamps to CSV.

Author: nikk909 (yinghua253659@163.com)
