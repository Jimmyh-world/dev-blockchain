# Cardano/Aiken RAG System Research

**Last Updated:** 2025-10-16
**Status:** Design Phase
**Version:** 1.0
**Topic:** Cardano blockchain knowledge base using RAG architecture

---

## Overview

Research and design documentation for building a specialized RAG (Retrieval-Augmented Generation) system focused on Cardano blockchain development, particularly Aiken smart contracts and off-chain architecture.

**Purpose:** Convert deep Cardano/Aiken expertise into queryable knowledge base for:
- Client consulting (instant expertise recall)
- Development assistance (pattern matching)
- Audit acceleration (security knowledge)
- Training material generation
- Content creation

---

## Documents

| Document | Purpose | Date | Status |
|----------|---------|------|--------|
| cardano_aiken_rag.md | Complete RAG system design | 2025-10-13 | Design complete |
| cardano_rag_system.md | Implementation architecture | 2025-10-13 | Design complete |

---

## Key Concepts

**Five-Collection Architecture:**
1. `cardano_aiken` - Aiken validator code, patterns, optimization
2. `cardano_offchain` - Lucid, Mesh, transaction building
3. `cardano_eutxo` - eUTXO model, state machines
4. `cardano_security` - Audit findings, vulnerabilities
5. `cardano_production` - Deployment, monitoring

**Technology:**
- Qdrant (vector database)
- Ollama (local embeddings)
- Multi-LLM routing (DeepSeek for code, CodeLlama for reasoning)

---

## Related Research

- `../agent-learning/` - General agent learning architecture (RAG + episodic memory)
- `../rag-systems/` - RAG agent introductions and patterns
- `/home/jimmyb/templates/haiku-4.5-research/` - Haiku 4.5 optimization patterns

---

## Related Implementation

- `/home/jimmyb/dev-lab/docs/infrastructure/SECURITY-RAG-ARCHITECTURE.md` - Security RAG implementation
- Future: Cardano RAG implementation (when Beast arrives)

---

## Next Steps

**When Beast Arrives (~2 weeks):**
1. Day 2-3: Deploy Cardano/Aiken RAG (priority ONE personal knowledge asset)
2. Ingest Aiken research and project documentation
3. Test query system
4. Integrate with development workflow

---

**Created:** 2025-10-16
**Topic:** Cardano/Aiken Knowledge Base
**Phase:** Research & Design
