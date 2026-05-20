# yandex-search

Command-line interface for [Yandex Search API](https://yandex.cloud/en/services/search-api) — web search and generative AI search optimized for the Russian internet.

Two commands: **yandex-search** (structured results with domain/date metadata) and **yandex-gen** (YandexGPT answers with cited sources).

## Install

```bash
# Recommended: uv tool (isolated, no venv needed)
uv tool install yandex-search

# Or pip
pip install yandex-search
```

## Setup

You need a [Yandex Cloud](https://cloud.yandex.ru) account with **Search API** enabled.

1. Create a service account and API key in [Yandex Cloud Console](https://console.cloud.yandex.ru)
2. Enable **Yandex Search API** for your folder
3. Set credentials:

**Option 1: config file (recommended)**

```bash
mkdir -p ~/.search-api
cat > ~/.search-api/config.json << 'EOF'
{
  "apiKey": "your-api-key",
  "folderId": "your-folder-id"
}
EOF
```

**Option 2: environment variables**

```bash
export YANDEX_API_KEY=your-api-key
export YANDEX_FOLDER_ID=your-folder-id
```

## Commands

### yandex-search

Web search with structured results — title, URL, domain, date, and text passages.

```bash
# Basic search
yandex-search "умный город цифровая платформа"

# More results
yandex-search "python async" -n 20

# Restrict to a domain
yandex-search "документация django" --site docs.djangoproject.com

# Search .com index (not .ru)
yandex-search "machine learning" -t com

# JSON output for pipelines
yandex-search "запрос" --json | jq '.[].url'
```

**Options:**

| Flag | Default | Description |
|---|---|---|
| `-n` / `--num-results` | 10 | Number of results |
| `-t` / `--type` | `ru` | Search index: `ru`, `com`, `tr`, `kk`, `be`, `uz` |
| `-r` / `--region` | — | Region code (e.g. `213` for Moscow) |
| `-p` / `--page` | 0 | Page number (0-indexed) |
| `--site` | — | Restrict results to this domain |
| `--json` | off | JSON array: `[{title, url, domain, date, passages}]` |

### yandex-gen

Generative search powered by YandexGPT — answers with cited sources.

```bash
# Ask a question
yandex-gen "объясни трансформеры в машинном обучении"

# Restrict sources to a specific site
yandex-gen "как настроить nginx reverse proxy" --site nginx.org

# JSON output
yandex-gen "запрос" --json
```

| Flag | Default | Description |
|---|---|---|
| `--site` | — | Restrict sources to this domain |
| `--json` | off | Raw JSON response from Yandex |

## JSON format (yandex-search)

```json
[
  {
    "title": "Page title",
    "url": "https://example.ru/page",
    "domain": "example.ru",
    "date": "2024-03-15",
    "passages": ["Relevant text snippet from the page..."]
  }
]
```

## Piping and scripting

```bash
# Extract all URLs
yandex-search "монографии умный город" --json | jq -r '.[].url'

# Filter results by domain pattern
yandex-search "документация" --json \
  | jq '[.[] | select(.domain | test("gov\\.ru|edu\\.ru"))]'

# Multi-page collection
for page in 0 1 2; do
  yandex-search "запрос" -p $page --json >> results.json
done
```

## Requirements

- Python 3.11+
- Yandex Cloud account with Search API enabled
- Credentials via `~/.search-api/config.json` or environment variables
