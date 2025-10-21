# Integration Architecture - READ THIS FIRST

**⚠️ CORE SYSTEM DOCUMENT ⚠️**

This project is part of a multi-project RAG system. Before structuring documentation, read:

📘 **[INTEGRATION-ARCHITECTURE-BLUEPRINT.md](./INTEGRATION-ARCHITECTURE-BLUEPRINT.md)**

## Your Role: Content Provider

**dev-blockchain is Pattern 1: Content Provider**

You:
1. Structure markdown docs with YAML frontmatter
2. Use clear section headings (## for chunks)
3. Preserve code blocks (never split)
4. Tag with domain/category/version
5. RAG system ingests and indexes your docs

## Key Sections for You

- **Pattern 1: Content Provider** - Your markdown structure guide
- **YAML Frontmatter Schema** - Required metadata fields
- **Structure Best Practices** - How to format docs

## Document Template

```markdown
---
domain: development
category: cardano
tags: [aiken, validator, smart-contract]
version: 1.2.0
date: 2025-10-21
priority: high
---

# Your Title

## Section (creates chunk boundary)

Your content here...

\`\`\`aiken
// Code blocks preserved intact
validator my_validator { }
\`\`\`
```

## How RAG Uses Your Docs

1. Reads YAML frontmatter (metadata)
2. Chunks on `##` headings
3. Preserves code blocks
4. Generates embeddings
5. Stores in `development-rag` collection
6. Now queryable: "How do I create an Aiken validator?"

## Related Projects

- **dev-rag**: Ingests your documentation
- **dev-scraping**: Can trigger re-ingestion when docs update

---

**Always add YAML frontmatter to new docs!**
