# Toronto Airbnb Review Text Analytics

<img width="2360" height="1232" alt="Main Topic" src="https://github.com/user-attachments/assets/01f11378-1edd-40f6-a01b-49414bc70fa1" />

---
<img width="2362" height="1232" alt="Subtopic" src="https://github.com/user-attachments/assets/3ec05955-00ed-4988-99db-3a18ff313735" />

---
## Problem Overview

Airbnb provides overall and category ratings, but these structured scores have two limitations:

- **Lack of granularity.** A single category rating does not reveal which specific issues drive satisfaction or dissatisfaction.
- **Hidden trade-offs.** Numeric ratings obscure trade-offs that guests typically only express in text (e.g., tolerating noise because of an excellent location).

To solve this, this project transforms unstructured Toronto Airbnb review text into topic-level metrics by combining **BERTopic**, **Neural Network Autoencoder**, **LDA**, **VADER sentiment scoring**, and a **Power BI** report, so hosts can identify which aspects of a stay drive guest attention and satisfaction.

---

## Dataset

- **Source:** [Inside Airbnb](https://insideairbnb.com/get-the-data/) — Toronto listings
- **Files used:** `reviews.csv.gz` (review text + listing IDs) and `listings.csv` (listing metadata for the dashboard)
- **Scope after filtering:** ~3K listings, ~8K reviews, ~29K topic-tagged review segments

---

## Repository Structure

```
airbnb_review_text_analytics/
├── data/
│   ├── raw/
│   │   └── reviews.csv.gz                        # raw Inside Airbnb review export
│   ├── processed/
│   │   ├── segment_df.csv                        # cleaned + segmented sentences
│   │   ├── segment_df_with_topics.csv            # raw BERTopic assignments
│   │   ├── segment_df_with_topics_final.csv      # after manual topic mapping
│   │   ├── topic_dictionary_table.csv            # raw 53-topic dictionary
│   │   ├── topic_dictionary_table_final.csv      # 33-topic dictionary after mapping
│   │   ├── bertopic_evaluation_summary.csv       # BERTopic quality metrics
│   │   ├── neural_topic_dictionary_table.csv     # comparison: NN + K-Means
│   │   └── lda_topic_dictionary_table.csv        # comparison: LDA
│   └── final/
│       └── segment_df_with_topics_and_vader.csv  # final dataset feeding Power BI
├── notebooks/
│   ├── 1_preprocessing_and_bertopic.ipynb
│   ├── 2_neuron_network_and_lda.ipynb
│   └── 3_vader_sentiment_scoring.ipynb
└── powerbi_report/
    ├── listings.csv
    └── toronto_airbnb_review_analytics_report.pbix
```

---

## Methodology

### 1. Preprocessing & Segmentation — `1_preprocessing_and_bertopic.ipynb`
- HTML and encoding cleanup with `ftfy` and `BeautifulSoup`.
- Sentence tokenization with NLTK, then **further split on commas, semicolons, and contrastive connectors** (`but`, `however`, `although`, `while`) so a single sentence with mixed sentiment becomes multiple analyzable segments.
- Language detection with `langdetect`; non-English segments are dropped.
- Text normalization with spaCy lemmatization and stop-word removal that **preserves negation words** (`not`, `no`, `never`) so polarity is not flipped during downstream sentiment scoring.

### 2. Topic Modeling — `1_preprocessing_and_bertopic.ipynb`
- Sentence embeddings: `all-MiniLM-L6-v2` from `sentence-transformers`.
- Dimensionality reduction: UMAP (`n_components=5`, cosine metric).
- Clustering: HDBSCAN (`min_cluster_size=200`, `min_samples=30`).
- Quality metrics: outlier rate, topic count, c_v coherence, topic diversity@10, silhouette score, Davies-Bouldin index.
- **Manual topic mapping:** the raw 53 BERTopic clusters were consolidated into **33 interpretable subtopics under 9 main topics** aligned with Airbnb's existing category structure (Cleanliness, Accuracy, Check-in, Communication, Location, Value).

### 3. Comparison Models — `2_neuron_network_and_lda.ipynb`
Two alternatives were benchmarked against BERTopic for transparency:
- **Autoencoder + K-Means** on sentence embeddings, with a hyperparameter grid over latent dimension, learning rate, and cluster count.
- **LDA** on count features, with a grid search over topic counts.

LDA scored highest on silhouette and topic diversity, and the neural pipeline scored highest on coherence — but **BERTopic produced the most interpretable, business-aligned topics**, so it was selected as the final method.

### 4. Sentiment Scoring — `3_vader_sentiment_scoring.ipynb`
- VADER compound score computed for each segment.
- Converted to a **0–100 Satisfaction Index**: `satisfaction = (compound + 1) / 2 * 100`.
- **Sentiment label** thresholds: `positive ≥ 0.05`, `negative ≤ -0.05`, otherwise `neutral`.
- Aggregated to main-topic and subtopic levels for the report.

---

## Power BI Report

Open `powerbi_report/toronto_airbnb_review_analytics_report.pbix`. The report has two pages and a listing-level filter, so hosts can switch between Toronto-wide benchmarks and a single-listing diagnosis.

### Page 1 — Main Topic Level
City-wide overview across the 9 main topics:
- KPI cards (listing, review, and segment counts) and a Toronto listing map.
- Main topic-share pie chart and average satisfaction radar.
- Sentiment distribution (positive / neutral / negative) per main topic.
- **Main Topic Attention vs Satisfaction matrix** — a four-quadrant view that flags topics where guests talk a lot but satisfaction is low, i.e. priority improvement areas.

### Page 2 — Subtopic Level
Drill into any main topic (e.g., Comfort → Interior Design, Bed Comfort, Overall Comfort, Interior Space & Natural Light, Basement & Ceiling):
- Sentence count and average satisfaction per subtopic.
- Subtopic-level Attention vs Satisfaction matrix.
- **Snippet table** of representative review segments with satisfaction scores and sentiment labels, so hosts see the actual guest wording behind every metric.

---

## Team

Rui Zhao, Wendy Xu, Al Cheaito, David Fogel, Zhaihan Gong
