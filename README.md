## Product Review Sentiment Filter & Recommendation System

![Architecture](architecture.png)

[![GitHub stars](https://img.shields.io/github/stars/your-username/your-repo?style=social)](https://github.com/your-username/your-repo/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/your-username/your-repo?style=social)](https://github.com/your-username/your-repo/network/members)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](#)

### Introduction
This project tackles a practical e‑commerce challenge: when buying products (e.g., laptops, headphones), buyers face oceans of noisy reviews — astroturfing, irrelevant or empty comments — making it hard to quickly find trustworthy, decision‑critical feedback (e.g., “Does this laptop meet its battery life claims?”). Our system performs parallel sentiment analysis and usefulness ranking, then retrieves and recommends high‑quality, relevant reviews across similar products.

中文简介：本项目聚焦真实有用评论的高效发现。通过并行情感分析与有用性特征抽取，过滤“水军/无意义”评论；结合关键词检索与轻量级关联推荐，为用户快速定位“续航/音质”等属性相关的优质评论，并跨品牌推荐相似商品的高价值评价。

### Features
- **Parallel sentiment analysis**: Multi-threaded VADER/TextBlob classifies reviews into positive/negative/neutral and filters low-signal neutral content.
- **Usefulness scoring & ranking**: Rank by signals such as upvotes/likes, word count, presence of media (images), and recency.
- **Faceted filtering**: Filter by sentiment polarity and usefulness thresholds (e.g., likes ≥ X, words ≥ Y).
- **Attribute keyword search**: Search for properties like "battery life", "sound quality", "build" to surface relevant reviews.
- **Lightweight recommendation**: Given a query (e.g., "laptop battery"), recommend top reviews from similar products using TF‑IDF/keyword association — no heavy CF needed.
- **Batch-friendly pipeline**: CSV in, CSV/JSON out; designed for offline or service mode.

### System Architecture
- **Languages & Libraries**: Python 3.9+, NLTK/VADER or TextBlob for sentiment; scikit‑learn for TF‑IDF; pandas for I/O.
- **Parallelism**: Python `concurrent.futures` or `multiprocessing` for two main stages:
  - Sentiment classification on batches of reviews.
  - Parallel extraction of usefulness features (length, likes, has_image, etc.).
- **IR & Recommendation (Core IR Design)**:
  - Indexing with TF‑IDF or BM25; per‑field weighting (e.g., title > review_text).
  - Query parsing with tokenization, language detection, synonym/alias expansion.
  - Ranking: combine lexical relevance with usefulness score (likes/length/media).

High‑level flow:
1) Ingest dataset → 2) Parallel sentiment tagging → 3) Parallel feature extraction → 4) Usefulness score & filter → 5) Keyword search → 6) Cross‑product recommendation.

### Information Retrieval Details
- **Indexing**
  - Tokenization: English uses word tokenizers; Chinese uses word segmentation (e.g., jieba) to support CJK terms like “续航/音质”.
  - Representation: TF‑IDF vectors (scikit‑learn) or BM25 (via rank‑bm25) built over `review_text` and optionally `product_title/brand`.
  - Field boosts: `title: 1.5`, `brand: 1.2`, `review_text: 1.0` (tunable).
- **Query Processing**
  - Language detection → pick tokenizer.
  - Normalization: lowercasing, stop‑word removal, punctuation stripping.
  - Synonyms/aliases: lightweight dictionary (e.g., “battery life” ↔ “续航”, “sound quality” ↔ “音质/声质”).
  - Optional query expansion: add top‑k terms from pseudo‑relevance feedback.
- **Filtering**
  - Sentiment facet: keep `positive/negative` or both; drop `neutral` by default.
  - Usefulness thresholds: `min_words`, `min_likes`, `has_image` flag.
  - Time window: filter by `created_at` to prioritize recency.
- **Ranking**
  - Base lexical score: cosine similarity (TF‑IDF) or BM25 score.
  - Usefulness score: e.g., `alpha*log1p(likes) + beta*has_image + gamma*word_norm`.
  - Final score: `lambda*lexical + (1-lambda)*usefulness` (all coefficients tunable via CLI).
- **Recommendation (Query‑Aligned)**
  - For a user query (e.g., “laptop battery”), retrieve top results from other brands’ products using the same index, then re‑rank by usefulness and diversity (MMR).
- **Evaluation (optional)**
  - Offline: precision@k/recall@k for a set of labeled queries.
  - Online: click‑through on recommended reviews; dwell time as proxy.

### Installation & Quick Start
1. Clone the repository
```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

2. Create environment (optional but recommended)
```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\\Scripts\\activate
```

3. Install dependencies
```bash
pip install -r requirements.txt
python -c "import nltk; nltk.download('vader_lexicon')"  # if using VADER
```

4. Prepare datasets (see “Datasets” below) and place CSVs under `data/`

5. Run a full pipeline example
```bash
python main.py \
  --input data/amazon_small_reviews.csv \
  --out filtered_reviews.csv \
  --sentiment vader \
  --min_words 20 \
  --min_likes 5 \
  --keep positive,negative \
  --query "laptop battery life" \
  --recommend_topk 10
```

### Usage Examples
- **CLI: filter by sentiment and usefulness**
```bash
python main.py --input data/reviews.csv --keep positive --min_words 30 --min_likes 10
```

- **CLI: attribute keyword search + recommendation**
```bash
python main.py \
  --input data/amazon_small_reviews.csv \
  --query "sound quality" \
  --recommend_topk 5 \
  --export_json results.json
```

- **Python API (example stub)**
```python
from pipeline import ReviewPipeline

pipe = ReviewPipeline(
    sentiment_backend="vader",  # or "textblob"
    parallel_workers=8,
)

df = pipe.load_csv("data/reviews.csv")
df = pipe.tag_sentiment_parallel(df)
df = pipe.extract_features_parallel(df)
df = pipe.rank_and_filter(df, keep=("positive","negative"), min_words=20, min_likes=5)

results = pipe.search_and_recommend(df, query="battery life", topk=10)
pipe.save(results, path="results.json")
```

### Datasets
- **Amazon Product Reviews (Small)** on Kaggle: includes >5k reviews with ratings, text, and helpfulness signals. Download via Kaggle: [Kaggle – Amazon Product Reviews (Small)](https://www.kaggle.com/).
- **JD/JD.com (京东) samples**: Publicly visible product review pages provide text, ratings, likes, and images; scrape only where permitted and cache to CSV with fields listed below.
- **Taobao/Tmall public product pages**: Public facing review snippets can be used similarly where allowed; ensure compliance with site policies and local laws.
- Place final CSVs under `data/`; one file per product category is recommended for clean indexing.

Expected CSV schema (minimal):
```text
review_id,product_id,product_title,brand,language,rating,review_text,likes,has_image,created_at
```

#### Social Media Sources (evidence of accessibility)
- **Reddit**: The Reddit API supports authenticated access to posts and comments; third‑party libraries exist for collection (e.g., PRAW). See the official API docs via the developer portal: [Reddit Developer Platform](https://developers.reddit.com/). Adhere to API rate limits and terms.
- **Twitter/X**: Official API access (paid/free tiers subject to change) allows programmatic retrieval of tweets and engagement metrics; consult [X Developer Platform](https://developer.twitter.com/). Respect policy and rate limits.
- **Weibo (新浪微博)**: Public pages expose posts and comments; availability depends on login/anti‑scraping measures and legal constraints. Obtain permission where required and throttle requests.

Use any social data strictly for research/educational purposes, respect robots.txt, platform terms, and regional regulations. Prefer platform APIs over HTML scraping where possible.

### What’s Innovative Here
- **Sentiment filtering + usefulness ranking** combined end‑to‑end to directly tackle the “real vs. noisy” review pain point.
- **Lightweight, explainable recommendation** driven by keyword association/TF‑IDF — transparent and easy to deploy, no heavy collaborative filtering.
- **Parallel-first pipeline** to keep latency low on large batches.

### Contributing
Contributions are welcome! Please open an issue to discuss ideas or submit a PR for:
- New feature extractors (e.g., verified‑purchase, reviewer reputation)
- Additional languages/lexicons
- Better ranking functions or evaluation scripts

### License & Contact
This project is released under the MIT License. See `LICENSE` for details.

Author: nikk909 (yinghua253659@163.com)

### TODO / Future Work
- Integrate transformer‑based sentiment (e.g., DistilBERT) behind a feature flag
- Add weak‑supervision for “usefulness” labels (Snorkel‑style heuristics)
- Online service mode with FastAPI + streaming ingestion
- Evaluation suite (precision@k for search/recommendation, A/B‑ready metrics)
- CJK tokenization improvements for mixed Chinese/English content

---

Keywords: sentiment analysis, product recommendation, review filtering, information retrieval, TF‑IDF, VADER, TextBlob, parallel processing, e‑commerce reviews


