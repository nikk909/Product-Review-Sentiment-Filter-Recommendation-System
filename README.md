## Product Review Sentiment Filter & Recommendation System (IR‑Focused)

![Architecture](architecture.jpg)

### Project Objective
Build a course‑level Information Retrieval (IR) pipeline over product reviews (CSV). Users enter attribute‑style queries (e.g., “battery life”, “sound quality”) and receive the most relevant, high‑value reviews.

### IR Pipeline (Course Fundamentals)
1) Document representation: represent review text (optionally title/brand) with BM25 vectors; allow per‑field weights.
2) Indexing: build an inverted index/sparse structures and store metadata (likes, has_image, created_at).
3) Query processing: lowercasing, punctuation stripping, stopword removal; English tokenization / Chinese segmentation; simple synonyms (battery life ↔ 续航).
4) Candidate retrieval: use BM25 to retrieve top‑k candidate reviews.
5) Ranking fusion: FinalScore = λ·LexicalScore + (1−λ)·Usefulness, where Usefulness is a normalized linear mix of log(1+likes), word_count, has_image; λ is tunable.
6) Filtering: optional sentiment filter and usefulness thresholds (min_likes, min_words, has_image).
7) Evaluation (optional): precision@k/recall@k on a small labeled query set; compare configurations.

### How to Use
- CLI and Python API examples in this repo show how to: load CSV, build index, run queries, and save results.
- Typical command: run BM25 retrieval with top‑k and usefulness thresholds; adjust λ to blend usefulness.

### Data Sources & Compliance
- Kaggle Amazon Product Reviews (Small): small, well‑structured review set suitable for IR prototyping.
- Chinese e‑commerce public pages (JD/Tmall): collect only where permitted; store text, likes, images, timestamps to CSV.
- Social media (Reddit, Twitter/X, Weibo): prefer official APIs; respect robots.txt, platform policies, and local regulations.

Author: nikk909 (yinghua253659@163.com)