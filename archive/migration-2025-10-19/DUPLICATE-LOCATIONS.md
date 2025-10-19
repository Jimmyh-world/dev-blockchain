# Duplicate Locations Identified - 2025-10-19

**Purpose:** Document all locations where blockchain knowledge was duplicated before migration

---

## Exact Duplicates (Verified by MD5)

### Location 1: ~/templates/cardano/
**Status:** EXACT DUPLICATE of dev-lab/docs/cardano/
**Files:** 13 files (~372KB)
**Last Modified:** 2025-10-02 to 2025-10-05
**Action Taken:** ⚪ None yet - marked for deprecation notice

**Files:**
- AGENTS.md
- JIMMYS-WORKFLOW.md
- README.md
- aiken-development-rules.md
- atlas-framework-guide.md
- cardano-ai-rules.md
- cardano-offchain-integration.md
- cardano-security-patterns.md
- compact_dev_guide.txt
- ethereum-cardano-comparison.md
- midnight-deep-research.md
- supporting document.md
- validator-design-guide.md

---

### Location 2: ~/hash-licence/ai-dev-ref/
**Status:** EXACT DUPLICATE of dev-lab/docs/cardano/
**Files:** 13 files (~372KB)
**Last Modified:** 2025-10-02 to 2025-10-05
**Action Taken:** ⚪ None yet - marked for deprecation notice

**Files:** Same as Location 1

---

### Location 3: ~/projects/cardano/aiken/.ai-rules/
**Status:** PARTIAL DUPLICATE (subset of dev-lab + some unique files)
**Files:** 10 files (~180KB)
**Last Modified:** 2025-10-02
**Action Taken:** ⚪ None yet - needs review for unique content

**Overlapping Files:**
- aiken-development-rules.md
- cardano-security-patterns.md
- ethereum-cardano-comparison.md
- validator-design-guide.md

**Unique Files (may need to migrate):**
- INTEGRATION-PLAN.md
- branch-guidelines.md
- checklists/ (subdirectory)
- project-init/ (subdirectory)

---

## Migration Status

**Primary Source:** ~/dev-lab/docs/cardano/ (most recent, Oct 17)
**Migrated To:** ~/dev-blockchain/cardano/

**Duplicate Waste Before Migration:** ~1MB (3 exact copies of 372KB)

**Next Steps:**
1. Add DEPRECATED.md to templates/cardano/
2. Add DEPRECATED.md to hash-licence/ai-dev-ref/
3. Review projects/cardano/aiken/.ai-rules/ for unique content
4. **NO DELETIONS** until further review

---

**Last Updated:** 2025-10-19
