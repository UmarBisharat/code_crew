# Webserver Logs Dataset

A synthetic 10,000-record web server log dataset formatted for OpenSearch bulk ingestion. Designed for testing, demos, and benchmarking OpenSearch queries without licensing or privacy concerns.

## Description

This project generates realistic e-commerce web server logs and packages them as an OpenSearch bulk index file. All data is entirely fabricated — no real user data is included.

The dataset simulates traffic for a fictional online store (`example.com`) over a 5-day period (March 18–22, 2024) and includes a realistic distribution of:

- HTTP methods (GET-heavy, with POST/PUT/DELETE on API endpoints)
- Status codes (65% 200, 15% 304, 8% 404, and others)
- Request paths (static assets, API calls, product pages, browse/filter pages)
- User agents (desktop, mobile, tablets, and search engine bots)
- Referrers (internal navigation, search engines, social media, direct)

## Files

| File | Description |
|------|-------------|
| `generate_synthetic_logs.py` | Python script that generates the synthetic dataset |
| `synthetic-webserver-logs-10k.txt` | Pre-generated 10K log dataset in OpenSearch bulk format (~3.9 MB) |

## Log Entry Schema

Each log entry contains the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `log_id` | string | Unique 7-digit identifier |
| `ip` | string | Randomized client IP address |
| `timestamp` | string | ISO 8601 timestamp (2024-03-18 to 2024-03-22) |
| `method` | string | HTTP method (GET, POST, PUT, DELETE, HEAD) |
| `path` | string | Request path (static, API, browse, product, filter, or page) |
| `protocol` | string | Always `HTTP/1.1` |
| `status` | integer | HTTP response status code |
| `bytes_sent` | integer | Response body size in bytes |
| `user_agent` | string | Client user agent string |
| `referer` | string | (Optional) Referring URL |

## Usage

### Option 1: Use the Pre-generated File

The dataset is ready to ingest into OpenSearch. The file contains:

1. An index creation command (`PUT /webserver-logs`) with 4 shards and 1 replica
2. A `POST /_bulk` request with all 10,000 documents

You can load it via OpenSearch Dev Tools or split it into the index creation and bulk API calls.

### Option 2: Regenerate the Dataset

```bash
python3 generate_synthetic_logs.py
```

This produces `synthetic-webserver-logs-10k.txt` with a fixed random seed (`42`) for reproducible output.

### Configuration

Edit these constants in `generate_synthetic_logs.py` to customize:

```python
NUM_RECORDS = 10000          # Number of log entries to generate
OUTPUT_FILE = "synthetic-webserver-logs-10k.txt"
INDEX_NAME = "webserver-logs"
```

## OpenSearch Index Settings

```json
{
  "settings": {
    "index": {
      "number_of_shards": 4,
      "number_of_replicas": 1
    }
  }
}
```

## Requirements

- Python 3.6+ (standard library only — no external dependencies)
- OpenSearch or Elasticsearch cluster (for ingestion)

## Example Queries

Once ingested, try these OpenSearch queries:

```json
// Count requests by status code
GET /webserver-logs/_search
{
  "size": 0,
  "aggs": {
    "status_codes": {
      "terms": { "field": "status" }
    }
  }
}

// Find all 500 errors
GET /webserver-logs/_search
{
  "query": {
    "term": { "status": 500 }
  }
}

// Traffic by HTTP method
GET /webserver-logs/_search
{
  "size": 0,
  "aggs": {
    "methods": {
      "terms": { "field": "method.keyword" }
    }
  }
}
```

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.

The dataset itself is entirely synthetic and contains no real user data.
