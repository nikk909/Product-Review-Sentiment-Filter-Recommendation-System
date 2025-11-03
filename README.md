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
- **IR & Recommendation**:
  - Tokenization + TF‑IDF vectorization for keyword search and query expansion.
  - Cosine similarity to retrieve cross‑product reviews aligned with user intent.

High‑level flow:
1) Ingest dataset → 2) Parallel sentiment tagging → 3) Parallel feature extraction → 4) Usefulness score & filter → 5) Keyword search → 6) Cross‑product recommendation.

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
- **Amazon Product Reviews (Small)** on Kaggle: includes >5k reviews with ratings, text, and helpfulness signals. Download: [Kaggle – Amazon Product Reviews (Small)](https://www.kaggle.com/)
- **Chinese e‑commerce subsets** (e.g., JD.com sample reviews) for multilingual testing. Sources vary; you can collect publicly available samples that include text, rating, likes, and media flags.

Expected CSV schema (minimal):
```text
review_id,product_id,product_title,brand,language,rating,review_text,likes,has_image,created_at
```

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

Author: Your Name (your.email@example.com)

### TODO / Future Work
- Integrate transformer‑based sentiment (e.g., DistilBERT) behind a feature flag
- Add weak‑supervision for “usefulness” labels (Snorkel‑style heuristics)
- Online service mode with FastAPI + streaming ingestion
- Evaluation suite (precision@k for search/recommendation, A/B‑ready metrics)
- CJK tokenization improvements for mixed Chinese/English content

---

Keywords: sentiment analysis, product recommendation, review filtering, information retrieval, TF‑IDF, VADER, TextBlob, parallel processing, e‑commerce reviews


