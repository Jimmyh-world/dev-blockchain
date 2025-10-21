# RAG System Integration Architecture Blueprint

**Version:** 1.0
**Date:** 2025-10-21
**Status:** Reference Blueprint
**Purpose:** Complete system architecture and integration patterns for all projects

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Four Integration Patterns](#four-integration-patterns)
3. [Specialized Scrapers](#specialized-scrapers)
4. [Universal Chunker](#universal-chunker)
5. [Complete Flow Examples](#complete-flow-examples)
6. [Cross-Project Coordination](#cross-project-coordination)
7. [API Contracts](#api-contracts)

---

## System Overview

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│  1. GUARDIAN (Event Publisher - Raspberry Pi 5)             │
│     "The Watcher" - Monitors for new information            │
│                                                              │
│  Monitors:                                                   │
│  - CVE feeds (every 15 min) → security-events               │
│  - GitHub releases (webhooks) → dev-events                  │
│  - Crypto prices (every 5 min) → trading-events             │
│                                                              │
│  When event detected → Publishes to Kafka                   │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│  KAFKA EVENT BUS (Beast - 192.168.68.100:9092)              │
│                                                              │
│  Topics:                                                     │
│  - security-events (CVEs, exploits, advisories)             │
│  - dev-events (releases, docs, frameworks)                  │
│  - trading-events (prices, news, analysis)                  │
│                                                              │
│  Retention: 7 days, Replication: 1, Partitions: 3/topic     │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│  2. SCRAPING PIPELINE (Event Consumer - Beast)              │
│     "The Fetcher" - Reacts to events and gets full content  │
│                                                              │
│  Pipeline:                                                   │
│  Kafka Consumer → Router → Specialized Scraper              │
│       ↓              ↓            ↓                          │
│  News Scraper   PDF Scraper   YouTube Scraper               │
│  GitHub Scraper Twitter Scraper RSS Scraper                 │
│       ↓              ↓            ↓                          │
│            Raw Content (clean text)                          │
│                      ↓                                       │
│            Universal Chunker (type-aware)                    │
│                      ↓                                       │
│            Ollama mxbai-embed-large (1024 dims)              │
│                      ↓                                       │
│            Qdrant (domain collections)                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│  QDRANT VECTOR DATABASE (Beast - :6333)                     │
│                                                              │
│  Collections:                                                │
│  - development-rag (Cardano, Aiken, frameworks)             │
│  - security-rag (CVEs, exploits, mitigations)               │
│  - trading-rag (prices, news, predictions)                  │
│                                                              │
│  Vectors: 1024 dims, Cosine distance, Binary quant enabled  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│  RAG QUERY ROUTER (Beast API - :8000)                       │
│                                                              │
│  - Auto-detects domain from query                           │
│  - Searches appropriate collection(s)                       │
│  - Returns answer + sources + metadata                      │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│  4. ANY PROJECT (Query API)                                 │
│     "The Consumers" - Ask questions, get answers            │
│                                                              │
│  Projects using RAG:                                         │
│  - dev-guardian (query security status)                     │
│  - dev-scraping (check if content exists)                   │
│  - dev-blockchain (reference in docs)                       │
│  - Your CLI tools (answer user questions)                   │
│  - Custom applications                                       │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow Summary

```
Event Detection → Kafka Queue → Specialized Scraper → Raw Content
                                        ↓
                                Universal Chunker
                                        ↓
                                Ollama Embeddings
                                        ↓
                                Qdrant Storage
                                        ↓
                                Query Router
                                        ↓
                                Answer + Sources
```

---

## Four Integration Patterns

### Pattern 1: Content Provider (dev-blockchain, documentation repos)

**Role:** Structure your markdown docs so RAG can ingest them intelligently.

**Implementation:**

```markdown
<!-- File: dev-blockchain/docs/cardano/aiken-validator-guide.md -->

---
domain: development
category: cardano
tags: [aiken, validator, smart-contract, tutorial]
version: 1.2.0
date: 2025-10-21
priority: high
language: aiken
author: your-name
---

# How to Create an Aiken Validator

This guide covers creating validators in Aiken for Cardano smart contracts.

## Basic Validator Structure

A spend validator in Aiken follows this pattern:

```aiken
validator my_validator {
  fn spend(datum: Datum, redeemer: Redeemer, context: ScriptContext) -> Bool {
    // Validation logic here
    True
  }
}
```

## Best Practices

When writing validators, always:
1. Validate all inputs thoroughly
2. Check script context carefully
3. Handle all edge cases
4. Test exhaustively

Code blocks are preserved intact, headings create chunk boundaries.
```

**Key Requirements:**

1. **YAML Frontmatter** (required fields)
   ```yaml
   domain: development | security | trading
   category: string (e.g., cardano, cve, price)
   tags: [array of keywords]
   version: semver or date
   date: ISO8601 timestamp
   priority: high | medium | low
   ```

2. **Structure Best Practices**
   - Use `##` for main sections (creates chunk boundaries)
   - Use ` ```language ` for code blocks (preserved intact)
   - One concept per section
   - Clear, descriptive headings
   - External links for references

3. **What RAG Does**
   - Extracts frontmatter metadata
   - Chunks on `##` boundaries
   - Preserves code blocks (never splits)
   - Generates 1024-dim embeddings
   - Stores with full metadata in domain collection

**Example Integration:**

```python
# In dev-blockchain or any doc repo
# Trigger ingestion when docs update

from dev_rag_client import RAGClient

client = RAGClient()

# Ingest single file
client.ingest_file(
    filepath="docs/cardano/aiken-validator-guide.md",
    auto_extract_metadata=True  # Reads YAML frontmatter
)

# Or ingest entire directory
client.ingest_directory(
    dirpath="docs/cardano/",
    pattern="*.md",
    domain="development"
)
```

---

### Pattern 2: Event Publisher (dev-guardian)

**Role:** Monitor external sources and publish events when new content appears.

**Implementation:**

```python
# File: dev-guardian/monitoring/cve_monitor.py

from kafka import KafkaProducer
import requests
import json
from datetime import datetime

class GuardianEventPublisher:
    """Publishes events to Kafka when new content detected"""

    def __init__(self):
        self.producer = KafkaProducer(
            bootstrap_servers=['192.168.68.100:9092'],
            value_serializer=lambda v: json.dumps(v).encode('utf-8')
        )

    def publish_security_event(self, cve_data: dict):
        """Publish CVE detection event"""
        event = {
            "event_type": "cve_published",
            "source": "nvd.nist.gov",
            "data": {
                "cve_id": cve_data['id'],
                "severity": cve_data['severity'],
                "description": cve_data['description'],
                "url": f"https://nvd.nist.gov/vuln/detail/{cve_data['id']}",
                "published_date": cve_data['published_date']
            },
            "timestamp": datetime.utcnow().isoformat(),
            "priority": self._calculate_priority(cve_data['severity'])
        }

        self.producer.send('security-events', event)
        print(f"⚠️  Published: {cve_data['id']} (severity: {cve_data['severity']})")

    def publish_dev_event(self, release_data: dict):
        """Publish GitHub release event"""
        event = {
            "event_type": "github_release",
            "source": f"github.com/{release_data['repo']}",
            "data": {
                "repo": release_data['repo'],
                "version": release_data['tag_name'],
                "release_notes_url": release_data['html_url'],
                "prerelease": release_data['prerelease']
            },
            "timestamp": release_data['published_at'],
            "priority": "high" if not release_data['prerelease'] else "medium"
        }

        self.producer.send('dev-events', event)
        print(f"📦 Published: {release_data['repo']} {release_data['tag_name']}")

    def publish_trading_event(self, price_data: dict):
        """Publish price alert event"""
        event = {
            "event_type": "price_threshold",
            "source": "coinbase_api",
            "data": {
                "symbol": price_data['symbol'],
                "price": price_data['price'],
                "change_24h": price_data['change_24h'],
                "volume": price_data['volume']
            },
            "timestamp": datetime.utcnow().isoformat(),
            "priority": "high" if abs(price_data['change_24h']) > 10 else "medium"
        }

        self.producer.send('trading-events', event)
        print(f"📈 Published: {price_data['symbol']} ${price_data['price']}")

    def _calculate_priority(self, severity: str) -> str:
        """Calculate event priority from CVE severity"""
        severity_map = {
            "critical": "high",
            "high": "high",
            "medium": "medium",
            "low": "low"
        }
        return severity_map.get(severity.lower(), "medium")

# Example usage in Guardian monitoring loop
publisher = GuardianEventPublisher()

# Monitor CVE feed (runs every 15 minutes)
def monitor_cve_feed():
    response = requests.get("https://services.nvd.nist.gov/rest/json/cves/2.0")

    for vuln in response.json()["vulnerabilities"]:
        cve_data = {
            "id": vuln["cve"]["id"],
            "severity": vuln["cve"]["metrics"]["cvssMetricV31"][0]["baseSeverity"],
            "description": vuln["cve"]["descriptions"][0]["value"],
            "published_date": vuln["cve"]["published"]
        }

        publisher.publish_security_event(cve_data)
```

**Event Schema Contract:**

All events MUST follow this schema:

```yaml
event:
  event_type: string          # Required: cve_published, github_release, price_threshold, etc.
  source: string              # Required: Origin URL or service name
  data: object                # Required: Event-specific payload
  timestamp: ISO8601          # Required: When event occurred
  priority: string            # Required: high, medium, low

  # Optional but recommended
  domain: string              # Optional: security, development, trading (auto-detected if missing)
  tags: [string]              # Optional: Keywords for filtering
```

**Supported Event Types:**

**Security Domain:**
- `cve_published` - New CVE from NVD
- `exploit_released` - Exploit code published
- `security_advisory` - Vendor advisory released

**Development Domain:**
- `github_release` - GitHub release published
- `npm_package_updated` - NPM package version bump
- `documentation_updated` - Docs changed

**Trading Domain:**
- `price_threshold` - Price crossed threshold
- `volume_spike` - Unusual volume detected
- `news_published` - Relevant news article

---

### Pattern 3: Event Consumer (dev-scraping)

**Role:** Listen to Kafka events, fetch detailed content, ingest into RAG.

**Implementation:**

```python
# File: dev-scraping/src/pipeline/consumer_pipeline.py

from kafka import KafkaConsumer
from src.scrapers.news_scraper import NewsScraper
from src.scrapers.pdf_scraper import PDFScraper
from src.scrapers.youtube_scraper import YouTubeScraper
from src.scrapers.github_scraper import GitHubScraper
from src.chunkers.content_chunker import UniversalChunker
from src.embeddings.embedder import get_embedding
from qdrant_client import QdrantClient
import json

class EventConsumerPipeline:
    """
    Complete event consumption pipeline:
    Kafka → Scraper → Chunker → Embeddings → Qdrant
    """

    def __init__(self):
        # Initialize all scrapers
        self.scrapers = {
            "news": NewsScraper(),
            "pdf": PDFScraper(),
            "youtube": YouTubeScraper(),
            "github": GitHubScraper()
        }

        self.chunker = UniversalChunker()
        self.qdrant = QdrantClient(host="192.168.68.100", port=6333)

        # Initialize Kafka consumer
        self.consumer = KafkaConsumer(
            'security-events',
            'dev-events',
            'trading-events',
            bootstrap_servers=['192.168.68.100:9092'],
            value_deserializer=lambda m: json.loads(m.decode('utf-8')),
            group_id='rag-ingestion-pipeline',
            auto_offset_reset='latest'
        )

    def run(self):
        """Main consumption loop"""
        print("🎧 Listening for events...")

        for message in self.consumer:
            event = message.value

            try:
                self.process_event(event)
            except Exception as e:
                print(f"❌ Error processing event: {e}")
                # Log error but continue processing

    def process_event(self, event: dict):
        """
        Process a single event through the pipeline

        Steps:
        1. Detect content type from URL
        2. Route to appropriate scraper
        3. Chunk the content
        4. Generate embeddings
        5. Store in Qdrant
        """
        print(f"\n📨 Event received: {event['event_type']}")

        # Extract URL from event data
        url = event['data'].get('url') or event['data'].get('release_notes_url')

        if not url:
            print(f"⚠️  No URL in event, skipping")
            return

        # Step 1: Detect content type
        content_type = self._detect_content_type(url, event)
        print(f"🔍 Content type: {content_type}")

        # Step 2: Scrape content
        scraper = self.scrapers.get(content_type)
        if not scraper:
            print(f"⚠️  No scraper for type: {content_type}")
            return

        raw_data = scraper.scrape(url)
        print(f"✅ Scraped: {len(raw_data['content'])} chars")

        # Step 3: Chunk content
        chunks = self.chunker.chunk(
            content=raw_data['content'],
            content_type=content_type,
            chunk_size=1000
        )
        print(f"📄 Chunked: {len(chunks)} chunks")

        # Step 4 & 5: Embed and store each chunk
        domain = self._detect_domain(event)
        collection = f"{domain}-rag"

        for i, chunk in enumerate(chunks):
            # Generate embedding
            embedding = get_embedding(chunk['text'])

            # Combine all metadata
            metadata = {
                **raw_data['metadata'],      # From scraper
                **event['data'],              # From event
                "chunk_index": i,
                "total_chunks": len(chunks),
                "domain": domain,
                "event_type": event['event_type'],
                "ingested_at": datetime.utcnow().isoformat(),
                "source_event": event['source']
            }

            # Store in Qdrant
            point_id = f"{url}_{i}_{int(time.time())}"

            self.qdrant.upsert(
                collection_name=collection,
                points=[{
                    "id": point_id,
                    "vector": embedding,
                    "payload": {
                        "text": chunk['text'],
                        **metadata
                    }
                }]
            )

        print(f"✅ Stored {len(chunks)} chunks in {collection}")

    def _detect_content_type(self, url: str, event: dict) -> str:
        """Detect content type from URL and event"""
        if 'youtube.com' in url or 'youtu.be' in url:
            return 'youtube'
        elif url.endswith('.pdf'):
            return 'pdf'
        elif 'github.com' in url:
            return 'github'
        else:
            return 'news'

    def _detect_domain(self, event: dict) -> str:
        """Detect domain from event type"""
        event_type = event.get('event_type', '').lower()

        if 'cve' in event_type or 'security' in event_type or 'exploit' in event_type:
            return 'security'
        elif 'price' in event_type or 'trading' in event_type or 'market' in event_type:
            return 'trading'
        else:
            return 'development'

# Run the pipeline
if __name__ == "__main__":
    pipeline = EventConsumerPipeline()
    pipeline.run()
```

---

### Pattern 4: Query API (ALL projects)

**Role:** Query the RAG system from any project to get answers.

**Client Library:**

```python
# File: dev-rag/src/api/client.py
# Install in other projects: pip install dev-rag-client

import requests
from typing import Optional, List, Dict

class RAGClient:
    """
    Client library for querying the RAG system

    Install in your project:
        pip install dev-rag-client

    Usage:
        from dev_rag_client import RAGClient
        client = RAGClient()
        result = client.query("How do I create an Aiken validator?")
    """

    def __init__(self, base_url: str = "http://192.168.68.100:8000"):
        self.base_url = base_url

    def query(
        self,
        query: str,
        domain: Optional[str] = None,
        domains: Optional[List[str]] = None,
        filters: Optional[Dict] = None,
        top_k: int = 5
    ) -> Dict:
        """
        Query the RAG system

        Args:
            query: Your question
            domain: Specific domain to search (development, security, trading)
            domains: Multiple domains to search
            filters: Metadata filters (e.g., {"severity": "critical"})
            top_k: Number of sources to retrieve

        Returns:
            {
                "answer": str,
                "sources": [
                    {
                        "file": str,
                        "text": str,
                        "score": float,
                        "metadata": dict
                    }
                ],
                "domain": str,
                "retrieved_at": str,
                "latency_ms": int
            }
        """
        response = requests.post(
            f"{self.base_url}/api/v1/query",
            json={
                "query": query,
                "domain": domain,
                "domains": domains,
                "filters": filters,
                "top_k": top_k
            }
        )

        response.raise_for_status()
        return response.json()

    def ingest_file(
        self,
        filepath: str,
        metadata: Optional[Dict] = None,
        auto_extract_metadata: bool = True
    ) -> Dict:
        """
        Ingest a file into RAG system

        Args:
            filepath: Path to markdown file
            metadata: Manual metadata (optional if file has YAML frontmatter)
            auto_extract_metadata: Extract YAML frontmatter

        Returns:
            {
                "chunks_created": int,
                "collection": str,
                "status": "success"
            }
        """
        with open(filepath, 'r') as f:
            content = f.read()

        response = requests.post(
            f"{self.base_url}/api/v1/ingest",
            json={
                "content": content,
                "metadata": metadata,
                "auto_extract_metadata": auto_extract_metadata
            }
        )

        response.raise_for_status()
        return response.json()

    def ingest_directory(
        self,
        dirpath: str,
        pattern: str = "*.md",
        domain: Optional[str] = None
    ) -> Dict:
        """
        Ingest entire directory of markdown files

        Returns:
            {
                "files_processed": int,
                "total_chunks": int,
                "status": "success"
            }
        """
        response = requests.post(
            f"{self.base_url}/api/v1/ingest/directory",
            json={
                "dirpath": dirpath,
                "pattern": pattern,
                "domain": domain
            }
        )

        response.raise_for_status()
        return response.json()

    def health_check(self) -> Dict:
        """Check if RAG system is healthy"""
        response = requests.get(f"{self.base_url}/health")
        return response.json()
```

**Usage Examples:**

```python
# Example 1: In dev-guardian (query security info)
from dev_rag_client import RAGClient

client = RAGClient()

# Guardian sees CVE mentioned in logs
result = client.query("What is CVE-2025-12345?")

print(result["answer"])
# "CVE-2025-12345 is a critical vulnerability..."

if result["sources"][0]["score"] > 0.8:
    # High confidence - use in alert
    guardian.send_alert(f"Known CVE: {result['answer'][:200]}")


# Example 2: In dev-scraping (check if already scraped)
result = client.query(
    "Aiken v1.2.0 release notes",
    domain="development",
    filters={"version": "1.2.0"}
)

if result["sources"]:
    print("Already have this release, skipping scrape")
else:
    scraper.scrape_release("https://github.com/aiken-lang/aiken/releases/tag/v1.2.0")


# Example 3: In your CLI tool
result = client.query("How do I test Aiken validators?")

# Show answer to user
print(f"Answer:\n{result['answer']}\n")
print("Sources:")
for i, source in enumerate(result['sources'][:3], 1):
    print(f"{i}. {source['file']} (relevance: {source['score']:.1%})")


# Example 4: Multi-domain query
result = client.query(
    "Security best practices for Cardano smart contracts",
    domains=["development", "security"]
)

# Gets results from BOTH domains
# - cardano-security-patterns.md (development)
# - recent-cardano-vulnerabilities.md (security)


# Example 5: Advanced filtering
result = client.query(
    "Critical Cardano vulnerabilities from this month",
    domain="security",
    filters={
        "severity": "critical",
        "tags": "cardano",
        "timestamp_after": "2025-10-01"
    }
)
```

---

## Specialized Scrapers

### 1. News Scraper

**Purpose:** Extract clean text from news articles (removes ads, navigation, etc.)

**File:** `dev-scraping/src/scrapers/news_scraper.py`

```python
import requests
from newspaper import Article
from bs4 import BeautifulSoup

class NewsScraper:
    """Intelligent news article scraper"""

    def scrape(self, url: str) -> dict:
        """
        Scrape news article using newspaper3k

        Returns:
            {
                "content": "clean article text",
                "metadata": {
                    "title": str,
                    "author": str,
                    "published_date": ISO8601,
                    "source": str,
                    "content_type": "news_article"
                }
            }
        """
        article = Article(url)
        article.download()
        article.parse()

        # Optional: Extract additional metadata
        article.nlp()  # Keywords, summary

        return {
            "content": article.text,
            "metadata": {
                "title": article.title,
                "author": ", ".join(article.authors) if article.authors else "Unknown",
                "published_date": article.publish_date.isoformat() if article.publish_date else None,
                "source": article.source_url,
                "content_type": "news_article",
                "keywords": article.keywords,  # Auto-extracted
                "summary": article.summary      # Auto-generated
            }
        }
```

**Handles:**
- Removes ads, navigation, comments
- Extracts clean article text
- Detects author, publish date
- Auto-generates keywords and summary

---

### 2. PDF Scraper

**Purpose:** Extract text from PDFs, preserve tables

**File:** `dev-scraping/src/scrapers/pdf_scraper.py`

```python
import pdfplumber
import requests
from io import BytesIO

class PDFScraper:
    """PDF scraper with table preservation"""

    def scrape(self, url: str) -> dict:
        """
        Scrape PDF and preserve structure

        Returns:
            {
                "content": "text with markdown tables",
                "metadata": {
                    "pages": int,
                    "has_tables": bool,
                    "content_type": "pdf"
                }
            }
        """
        # Download PDF
        response = requests.get(url)
        pdf_bytes = BytesIO(response.content)

        full_text = []
        tables = []

        with pdfplumber.open(pdf_bytes) as pdf:
            for page_num, page in enumerate(pdf.pages, 1):
                # Extract text
                text = page.extract_text()
                if text:
                    full_text.append(f"## Page {page_num}\n\n{text}")

                # Extract tables
                page_tables = page.extract_tables()
                if page_tables:
                    for table in page_tables:
                        tables.append({
                            "page": page_num,
                            "data": table
                        })

        # Combine content
        content = "\n\n".join(full_text)

        # Add tables as markdown
        if tables:
            content += "\n\n## Tables\n\n"
            for i, table_info in enumerate(tables, 1):
                content += f"### Table {i} (Page {table_info['page']})\n\n"
                content += self._table_to_markdown(table_info['data'])
                content += "\n\n"

        return {
            "content": content,
            "metadata": {
                "source": url,
                "content_type": "pdf",
                "pages": len(pdf.pages),
                "has_tables": len(tables) > 0,
                "table_count": len(tables)
            }
        }

    def _table_to_markdown(self, table: list) -> str:
        """Convert table array to markdown"""
        if not table or not table[0]:
            return ""

        # Header
        md = "| " + " | ".join(str(cell or "") for cell in table[0]) + " |\n"
        md += "| " + " | ".join(["---"] * len(table[0])) + " |\n"

        # Rows
        for row in table[1:]:
            md += "| " + " | ".join(str(cell or "") for cell in row) + " |\n"

        return md
```

**Handles:**
- Multi-column layouts
- Tables (converted to markdown)
- Page preservation
- Missing text handling

---

### 3. YouTube Scraper

**Purpose:** Extract transcripts from videos

**File:** `dev-scraping/src/scrapers/youtube_scraper.py`

```python
from youtube_transcript_api import YouTubeTranscriptApi
import yt_dlp
import re

class YouTubeScraper:
    """YouTube transcript scraper"""

    def scrape(self, url: str) -> dict:
        """
        Scrape YouTube video transcript

        Returns:
            {
                "content": "formatted transcript with timestamps",
                "metadata": {
                    "title": str,
                    "channel": str,
                    "video_id": str,
                    "duration": int
                }
            }
        """
        # Extract video ID
        video_id = self._extract_video_id(url)

        # Get transcript
        try:
            transcript = YouTubeTranscriptApi.get_transcript(video_id)
        except:
            # Try auto-generated
            transcript = YouTubeTranscriptApi.get_transcript(
                video_id,
                languages=['en', 'en-US']
            )

        # Get metadata
        metadata = self._get_metadata(video_id)

        # Format content
        content = f"# {metadata['title']}\n\n"
        content += f"**Channel:** {metadata['channel']}\n"
        content += f"**Published:** {metadata['published_date']}\n"
        content += f"**Duration:** {metadata['duration']}s\n\n"
        content += "## Transcript\n\n"

        for entry in transcript:
            timestamp = self._format_timestamp(entry['start'])
            content += f"**[{timestamp}]** {entry['text']}\n\n"

        return {
            "content": content,
            "metadata": {
                **metadata,
                "video_id": video_id,
                "source": url,
                "content_type": "youtube_transcript"
            }
        }

    def _extract_video_id(self, url: str) -> str:
        """Extract video ID from various YouTube URL formats"""
        patterns = [
            r'(?:youtube\.com\/watch\?v=)([\w-]+)',
            r'(?:youtu\.be\/)([\w-]+)',
            r'(?:youtube\.com\/embed\/)([\w-]+)'
        ]

        for pattern in patterns:
            match = re.search(pattern, url)
            if match:
                return match.group(1)

        raise ValueError(f"Could not extract video ID from {url}")

    def _get_metadata(self, video_id: str) -> dict:
        """Get video metadata using yt-dlp"""
        ydl_opts = {'quiet': True, 'no_warnings': True}

        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(
                f"https://youtube.com/watch?v={video_id}",
                download=False
            )

        return {
            "title": info['title'],
            "channel": info['uploader'],
            "published_date": info['upload_date'],
            "duration": info['duration'],
            "view_count": info.get('view_count', 0)
        }

    def _format_timestamp(self, seconds: float) -> str:
        """Convert seconds to HH:MM:SS or MM:SS"""
        hours = int(seconds // 3600)
        minutes = int((seconds % 3600) // 60)
        secs = int(seconds % 60)

        if hours > 0:
            return f"{hours:02d}:{minutes:02d}:{secs:02d}"
        else:
            return f"{minutes:02d}:{secs:02d}"
```

**Handles:**
- Auto-generated captions
- Manual captions
- Timestamp preservation
- Multiple URL formats

---

### 4. GitHub Scraper

**Purpose:** Scrape GitHub release notes, issues, discussions

**File:** `dev-scraping/src/scrapers/github_scraper.py`

```python
import requests
from bs4 import BeautifulSoup

class GitHubScraper:
    """GitHub content scraper"""

    def __init__(self, github_token: str = None):
        self.token = github_token
        self.headers = {}
        if github_token:
            self.headers['Authorization'] = f'token {github_token}'

    def scrape(self, url: str) -> dict:
        """
        Scrape GitHub content (releases, issues, etc.)

        Detects content type and routes appropriately
        """
        if '/releases/' in url:
            return self._scrape_release(url)
        elif '/issues/' in url:
            return self._scrape_issue(url)
        else:
            return self._scrape_generic(url)

    def _scrape_release(self, url: str) -> dict:
        """Scrape GitHub release notes"""
        # Parse URL: github.com/owner/repo/releases/tag/v1.0.0
        parts = url.split('/')
        owner = parts[3]
        repo = parts[4]
        tag = parts[-1]

        # Use GitHub API for clean data
        api_url = f"https://api.github.com/repos/{owner}/{repo}/releases/tags/{tag}"
        response = requests.get(api_url, headers=self.headers)
        release = response.json()

        # Format content
        content = f"# {repo} {release['tag_name']}\n\n"
        content += f"**Released:** {release['published_at']}\n\n"
        content += f"## Release Notes\n\n{release['body']}\n\n"

        if release.get('assets'):
            content += "## Assets\n\n"
            for asset in release['assets']:
                content += f"- [{asset['name']}]({asset['browser_download_url']})\n"

        return {
            "content": content,
            "metadata": {
                "repo": f"{owner}/{repo}",
                "version": release['tag_name'],
                "published_date": release['published_at'],
                "prerelease": release['prerelease'],
                "source": url,
                "content_type": "github_release"
            }
        }

    def _scrape_issue(self, url: str) -> dict:
        """Scrape GitHub issue"""
        # Similar to release but for issues
        # Parse URL and use API
        pass

    def _scrape_generic(self, url: str) -> dict:
        """Scrape generic GitHub page (fallback)"""
        response = requests.get(url)
        soup = BeautifulSoup(response.text, 'html.parser')

        # Extract markdown content
        content_div = soup.find('article', class_='markdown-body')
        if content_div:
            content = content_div.get_text()
        else:
            content = soup.get_text()

        return {
            "content": content,
            "metadata": {
                "source": url,
                "content_type": "github_page"
            }
        }
```

**Handles:**
- Release notes (via API)
- Issues (via API)
- Generic pages (fallback to scraping)
- Authentication (higher rate limits)

---

## Universal Chunker

**Purpose:** Intelligently chunk content based on type, preserving structure

**File:** `dev-scraping/src/chunkers/content_chunker.py`

```python
import re

class UniversalChunker:
    """
    Type-aware content chunker

    Preserves structure based on content type:
    - News: Paragraphs
    - PDF: Tables intact
    - YouTube: Topic boundaries (timestamp gaps)
    - Code: Function/class boundaries
    """

    def chunk(
        self,
        content: str,
        content_type: str,
        chunk_size: int = 1000
    ) -> list[dict]:
        """
        Chunk content intelligently

        Args:
            content: Raw text
            content_type: news_article, pdf, youtube_transcript, code, etc.
            chunk_size: Target size in characters

        Returns:
            [{"text": str, "metadata": dict}, ...]
        """
        if content_type == "news_article":
            return self._chunk_article(content, chunk_size)
        elif content_type == "pdf":
            return self._chunk_pdf(content, chunk_size)
        elif content_type == "youtube_transcript":
            return self._chunk_transcript(content, chunk_size)
        elif content_type == "code":
            return self._chunk_code(content, chunk_size)
        else:
            return self._chunk_generic(content, chunk_size)

    def _chunk_article(self, content: str, size: int) -> list[dict]:
        """Chunk news article on paragraph boundaries"""
        paragraphs = content.split('\n\n')
        chunks = []
        current_chunk = ""

        for para in paragraphs:
            # Skip empty paragraphs
            if not para.strip():
                continue

            # Check if adding paragraph exceeds size
            if len(current_chunk) + len(para) < size:
                current_chunk += para + "\n\n"
            else:
                # Save current chunk
                if current_chunk:
                    chunks.append({"text": current_chunk.strip()})
                # Start new chunk
                current_chunk = para + "\n\n"

        # Add remaining
        if current_chunk:
            chunks.append({"text": current_chunk.strip()})

        return chunks

    def _chunk_pdf(self, content: str, size: int) -> list[dict]:
        """
        Chunk PDF preserving tables

        Never splits markdown tables
        """
        # Split on markdown headers
        sections = re.split(r'\n(?=#{1,3} )', content)
        chunks = []

        for section in sections:
            # Check if section contains a table
            has_table = '|' in section and '---' in section

            if has_table:
                # Keep table sections intact
                chunks.append({
                    "text": section,
                    "has_table": True
                })
            else:
                # Can split this section if too large
                if len(section) > size:
                    # Split on paragraph boundaries
                    sub_chunks = self._chunk_generic(section, size)
                    chunks.extend(sub_chunks)
                else:
                    chunks.append({"text": section})

        return chunks

    def _chunk_transcript(self, content: str, size: int) -> list[dict]:
        """
        Chunk YouTube transcript on topic boundaries

        Detects topic changes using:
        - Large timestamp gaps (>30s silence)
        - Semantic shifts (future: use embeddings)
        """
        lines = content.split('\n')
        chunks = []
        current_chunk = ""
        last_timestamp = 0

        for line in lines:
            # Detect timestamp format: **[MM:SS]**
            timestamp_match = re.search(r'\*\*\[(\d+):(\d+)\]\*\*', line)

            if timestamp_match:
                minutes = int(timestamp_match.group(1))
                seconds = int(timestamp_match.group(2))
                current_timestamp = minutes * 60 + seconds

                # If large time gap (>30s), likely topic change
                if current_timestamp - last_timestamp > 30 and current_chunk:
                    chunks.append({
                        "text": current_chunk.strip(),
                        "timestamp_start": last_timestamp
                    })
                    current_chunk = ""

                last_timestamp = current_timestamp

            # Add line to current chunk
            current_chunk += line + "\n"

            # Also chunk by size
            if len(current_chunk) >= size:
                chunks.append({
                    "text": current_chunk.strip(),
                    "timestamp_start": last_timestamp
                })
                current_chunk = ""

        # Add remaining
        if current_chunk:
            chunks.append({
                "text": current_chunk.strip(),
                "timestamp_start": last_timestamp
            })

        return chunks

    def _chunk_code(self, content: str, size: int) -> list[dict]:
        """
        Chunk code on function/class boundaries

        Preserves:
        - Function definitions
        - Class definitions
        - Docstrings
        """
        # Simple implementation: split on function/class keywords
        # Advanced: use AST parsing

        # Pattern: def, class, async def
        pattern = r'\n(?=(?:async\s+)?(?:def|class)\s+\w+)'
        sections = re.split(pattern, content)

        chunks = []
        current_chunk = ""

        for section in sections:
            if len(current_chunk) + len(section) < size:
                current_chunk += section
            else:
                if current_chunk:
                    chunks.append({"text": current_chunk})
                current_chunk = section

        if current_chunk:
            chunks.append({"text": current_chunk})

        return chunks

    def _chunk_generic(self, content: str, size: int) -> list[dict]:
        """
        Generic chunker for unknown types

        Splits on sentence boundaries
        """
        sentences = re.split(r'(?<=[.!?])\s+', content)
        chunks = []
        current_chunk = ""

        for sentence in sentences:
            if len(current_chunk) + len(sentence) < size:
                current_chunk += sentence + " "
            else:
                if current_chunk:
                    chunks.append({"text": current_chunk.strip()})
                current_chunk = sentence + " "

        if current_chunk:
            chunks.append({"text": current_chunk.strip()})

        return chunks
```

**Key Features:**

1. **Preserves Structure**
   - Code blocks stay intact
   - Tables don't split
   - Functions/classes kept together

2. **Type-Aware Boundaries**
   - News: Paragraphs
   - PDF: Sections and tables
   - YouTube: Topic changes (time gaps)
   - Code: Function/class definitions

3. **Smart Sizing**
   - Target ~1000 chars
   - Never exceed max (for embeddings)
   - Preserve semantic units

---

## Complete Flow Examples

### Example 1: CVE Security Alert Flow

```
┌─ STEP 1: Guardian Detects CVE ─────────────────────────┐
│                                                          │
│  Guardian monitoring NVD feed (every 15 min)            │
│  → Finds: CVE-2025-99999 (Critical - Cardano RCE)       │
│  → Publishes to Kafka 'security-events':                │
│     {                                                    │
│       "event_type": "cve_published",                    │
│       "data": {                                          │
│         "cve_id": "CVE-2025-99999",                     │
│         "severity": "critical",                          │
│         "url": "https://nvd.nist.gov/..."              │
│       }                                                  │
│     }                                                    │
└──────────────────────────────────────────────────────────┘
                         ↓
┌─ STEP 2: Scraping Receives Event ──────────────────────┐
│                                                          │
│  Event consumer listening to 'security-events'          │
│  → Receives CVE-2025-99999 alert                        │
│  → Detects content_type: "pdf" (from .pdf URL)          │
│  → Routes to PDFScraper                                 │
└──────────────────────────────────────────────────────────┘
                         ↓
┌─ STEP 3: PDF Scraper Fetches Content ──────────────────┐
│                                                          │
│  PDFScraper.scrape(url)                                 │
│  → Downloads PDF from NVD                               │
│  → Extracts text with pdfplumber                        │
│  → Preserves tables (affected versions)                │
│  → Returns: 5 pages, 2 tables, 8,432 chars             │
└──────────────────────────────────────────────────────────┘
                         ↓
┌─ STEP 4: Universal Chunker Processes ──────────────────┐
│                                                          │
│  UniversalChunker.chunk(content, "pdf", 1000)           │
│  → Splits on section headers                            │
│  → Keeps tables intact (not split)                      │
│  → Creates 9 chunks (~1000 chars each)                  │
└──────────────────────────────────────────────────────────┘
                         ↓
┌─ STEP 5: Embedding Generation ─────────────────────────┐
│                                                          │
│  For each of 9 chunks:                                  │
│  → get_embedding(chunk_text)                            │
│  → Ollama mxbai-embed-large generates 1024-dim vector  │
│  → Takes ~24ms per chunk (9 × 24ms = 216ms total)      │
└──────────────────────────────────────────────────────────┘
                         ↓
┌─ STEP 6: Qdrant Storage ───────────────────────────────┐
│                                                          │
│  qdrant.upsert(collection="security-rag", points=[      │
│    {                                                     │
│      "vector": [1024 floats],                          │
│      "payload": {                                        │
│        "text": "CVE-2025-99999 affects...",           │
│        "domain": "security",                            │
│        "cve_id": "CVE-2025-99999",                     │
│        "severity": "critical",                          │
│        "chunk_index": 0,                                │
│        "has_table": true                                │
│      }                                                   │
│    },                                                    │
│    ... (8 more chunks)                                  │
│  ])                                                      │
│  → Stored 9 vectors in 'security-rag'                  │
└──────────────────────────────────────────────────────────┘
                         ↓
┌─ STEP 7: Now Queryable ────────────────────────────────┐
│                                                          │
│  // From dev-guardian                                   │
│  result = client.query("What is CVE-2025-99999?")      │
│                                                          │
│  RAG System:                                            │
│  → Auto-detects domain: "security"                     │
│  → Searches 'security-rag' collection                  │
│  → Finds chunk with score 0.95                         │
│  → Sends to Claude Haiku 4.5                           │
│  → Returns:                                             │
│     "CVE-2025-99999 is a critical RCE vulnerability    │
│      in Cardano node 8.0.0-8.5.2. Upgrade to 8.5.3     │
│      immediately. See mitigation table..."             │
│                                                          │
│  Guardian uses this in automated alert! ✅             │
└──────────────────────────────────────────────────────────┘
```

**Total Time:** ~5 minutes from detection to queryable

---

### Example 2: YouTube Video Processing

```
Guardian detects: Charles Hoskinson Cardano update video
         ↓
Kafka: dev-events {"url": "youtube.com/watch?v=ABC123"}
         ↓
Scraping: Routes to YouTubeScraper
         ↓
YouTube API: Downloads 45-minute transcript
         ↓
Chunker: Splits on topic changes (timestamp gaps)
  - Chunk 1: [00:00-05:30] Introduction
  - Chunk 2: [05:30-12:45] Security update
  - Chunk 3: [12:45-25:10] Roadmap discussion
  - ... (8 chunks total)
         ↓
Embeddings: 8 × 24ms = 192ms
         ↓
Qdrant: Stores in 'development-rag'
         ↓
Queryable: "What did Charles say about security?"
  → Returns: "[05:30] The recent vulnerability..."
     Source: youtube.com/watch?v=ABC123 [05:30]
```

---

## Cross-Project Coordination

### Project Responsibilities

| Project | Role | Responsibilities |
|---------|------|------------------|
| **dev-guardian** | Event Publisher | Monitor feeds, publish events to Kafka |
| **dev-scraping** | Event Consumer | Scrape content, ingest into RAG |
| **dev-blockchain** | Content Provider | Structure docs with YAML frontmatter |
| **dev-rag** | RAG System | Embeddings, storage, query API |
| **Your CLI/Apps** | Query Consumer | Ask questions, get answers |

### Communication Channels

1. **Kafka Topics** (Guardian → Scraping)
   - `security-events`
   - `dev-events`
   - `trading-events`

2. **RAG Query API** (All projects → RAG)
   - HTTP REST API on port 8000
   - Client library: `dev-rag-client`

3. **Qdrant Direct** (Advanced users only)
   - Direct vector search on port 6333
   - Not recommended (use RAG API instead)

### Deployment Coordination

**On Guardian (Pi 5):**
```bash
# Install monitoring scripts
cd ~/dev-guardian
pip install kafka-python requests

# Run monitors
python monitors/cve_monitor.py &
python monitors/github_monitor.py &
python monitors/price_monitor.py &
```

**On Beast:**
```bash
# Deploy Kafka
cd ~/dev-rag/deployment/kafka
docker-compose up -d

# Deploy RAG services
cd ~/dev-rag
docker-compose up -d

# Run scraping pipeline
cd ~/dev-scraping
python src/pipeline/consumer_pipeline.py &
```

---

## API Contracts

### Kafka Event Schema

```typescript
interface KafkaEvent {
  event_type: string;        // cve_published, github_release, etc.
  source: string;            // Origin URL/service
  data: {                    // Event-specific payload
    url?: string;
    [key: string]: any;
  };
  timestamp: string;         // ISO8601
  priority: "high" | "medium" | "low";
  domain?: "security" | "development" | "trading";  // Optional
  tags?: string[];           // Optional
}
```

### RAG Query Request

```typescript
interface QueryRequest {
  query: string;
  domain?: string;           // Optional: filter to one domain
  domains?: string[];        // Optional: filter to multiple domains
  filters?: {                // Optional: metadata filters
    [key: string]: any;
  };
  top_k?: number;           // Default: 5
}
```

### RAG Query Response

```typescript
interface QueryResponse {
  answer: string;
  sources: Array<{
    file: string;
    text: string;
    score: number;
    metadata: {
      domain: string;
      category: string;
      [key: string]: any;
    };
  }>;
  domain: string;            // Detected/specified domain
  retrieved_at: string;      // ISO8601
  latency_ms: number;
}
```

### Content Metadata Schema

```typescript
interface ContentMetadata {
  // Required fields
  domain: "development" | "security" | "trading";
  category: string;
  timestamp: string;         // ISO8601

  // Recommended fields
  tags?: string[];
  version?: string;
  priority?: "high" | "medium" | "low";
  source?: string;

  // Content-type specific
  cve_id?: string;           // For security domain
  severity?: string;         // For security domain
  symbol?: string;           // For trading domain
  repo?: string;             // For development domain

  // Scraper metadata
  content_type?: string;     // news_article, pdf, youtube_transcript, etc.
  chunk_index?: number;
  total_chunks?: number;
  has_table?: boolean;

  // Optional
  [key: string]: any;
}
```

---

## Version History

- **v1.0** (2025-10-21): Initial blueprint
  - 4 integration patterns documented
  - 4 specialized scrapers defined
  - Universal chunker architecture
  - Complete flow examples

---

**This is the master reference blueprint for RAG system integration.**

All projects should reference this document for integration patterns and contracts.
