# Content Engagement & Sentiment Analysis — Power BI Project

## Project title
Content Engagement & Sentiment Analysis Dashboard

## Short summary
A Power BI solution that ingests content metadata and textual feedback (social posts, comments, article metrics, video stats), calculates engagement metrics and sentiment scores, and provides interactive dashboards to guide content strategy: what to publish, where, and when — and why certain content performs well or poorly.

## Objectives
- Measure cross-channel engagement (views, likes, shares, comments).
- Quantify sentiment and topic-level sentiment trends.
- Surface top-performing content and risky content (negative sentiment spikes).
- Provide actionable recommendations for content strategy (format, topics, time).
- Automate daily/weekly reports publishable to Power BI Service.

## Domain
Digital content operations / marketing analytics (social media, blogs, video platforms).

## Data sources (examples)
- Social APIs: Twitter/X, Facebook, Instagram, YouTube Data API (post metadata, impressions, likes, comments).
- Website analytics: Google Analytics / server logs (pageviews, bounce, session duration).
- Internal CMS: publish date, author, tags, category.
- Comments / Reviews: scraped or via API (text to be used for NLP).
- Optional: CRM or ad spend data to correlate paid promotion vs organic engagement.

## Data pipeline (high level)
1. **Extract**: scheduled pulls from APIs or CSV exports.
2. **Transform**:
   - Standardize timestamp to UTC and create time dimension.
   - Normalize channel names and content types.
   - Clean comment text (remove URLs, stopwords, HTML).
3. **NLP / Sentiment**:
   - Language detection (ignore unsupported languages or route separately).
   - Sentiment scoring (VADER for short social text or a Transformer-based model for higher accuracy). Output: `sentiment_score` (float -1..+1) and `sentiment_label` (Negative / Neutral / Positive).
   - Topic extraction (LDA or BERTopic) to assign `topic_id` and `topic_label`.
4. **Load**: push processed tables (CSV or database) into Power BI data source (Azure SQL / PostgreSQL / folder with CSVs).

## Data model (Star schema)
- `fact_engagements` (fact)
  - engagement_id, content_id, channel_id, date_id, views, likes, shares, comments_count, clicks, impressions, paid_spend
- `dim_content`
  - content_id, title, author_id, publish_date, content_type, length_sec, category, tags
- `dim_channel`
  - channel_id, channel_name, platform_type
- `dim_date`
  - date_id, date, year, month, week_of_year, day_of_week, hour
- `fact_comments` (optional, for text-level analysis)
  - comment_id, content_id, user_id, comment_text, sentiment_score, sentiment_label, topic_id, created_at
- `dim_topic` (topic_id, topic_label)

## Key measures (DAX) — examples
```dax
-- Engagement rate per view
Engagement Rate = 
DIVIDE(
    SUM(fact_engagements[likes]) + SUM(fact_engagements[shares]) + SUM(fact_engagements[comments_count]),
    SUM(fact_engagements[views])
)

-- Average sentiment (content-level)
Avg Sentiment = AVERAGE(fact_comments[sentiment_score])

-- Positive comment % (label-based)
% Positive Comments = 
DIVIDE(
    CALCULATE(COUNTROWS(fact_comments), fact_comments[sentiment_label] = "Positive"),
    COUNTROWS(fact_comments)
)
