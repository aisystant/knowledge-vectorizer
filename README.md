# vectorizer
Generates semantic embeddings from markdown-based repositories for search and retrieval

## Overview

A Python script packaged as a Docker image that processes markdown documents and generates vector embeddings for semantic search and retrieval.

## Inputs

- **docs/** - Directory containing markdown documents (mount to container or pass as parameter)
- **LanceDB path** - Local path or S3 URI for vector storage
- **Embedding credentials** - OpenAI API credentials

## Processing

- Markdown files must be under 10,000 characters
- Files exceeding 10k chars: only first 10k is processed, script exits with non-zero status
- Uses OpenAI's `text-embedding-3-large` model

## Stored Fields

| Field | Description |
|-------|-------------|
| filename | Original file path |
| content | Full markdown content |
| embedding | Vector embedding |
| hash | Content hash for incremental updates |

## Usage

### Prerequisites

```bash
pip install -r requirements.txt
export OPENAI_API_KEY=your-api-key
```

### Python

```bash
# Local storage
python vectorizer.py --docs ./example_docs --db ./vectors.lancedb

# S3 storage
python vectorizer.py --docs ./docs --db s3://my-bucket/vectors.lancedb
```

### Docker

```bash
# Build
docker build -t vectorizer .

# Run with local storage
docker run \
  -v $(pwd)/example_docs:/docs \
  -v $(pwd)/data:/data \
  -e OPENAI_API_KEY \
  vectorizer --docs /docs --db /data/vectors.lancedb

# Run with S3
docker run \
  -e OPENAI_API_KEY \
  -e AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY \
  vectorizer --docs /docs --db s3://my-bucket/vectors.lancedb
```

### CLI Options

| Option | Description |
|--------|-------------|
| `--docs` | Path to directory containing markdown files |
| `--db` | LanceDB path (local or S3 URI) |
| `--openai-key` | OpenAI API key (or use `OPENAI_API_KEY` env var) |

## Incremental Updates

The vectorizer only recomputes embeddings for files that have changed:

- New files: embedded and added
- Modified files: re-embedded (detected by hash)
- Deleted files: removed from database
- Unchanged files: skipped (saves API costs)
