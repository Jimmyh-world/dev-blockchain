# dev-blockchain - Blockchain Knowledge Repository

<!--
TEMPLATE_VERSION: 1.5.0
TEMPLATE_SOURCE: /home/jimmyb/templates/AGENTS.md.template
LAST_SYNC: 2025-10-19
SYNC_CHECK: Run ~/templates/tools/check-version.sh to verify you have the latest template version
AUTO_SYNC: Run ~/templates/tools/sync-templates.sh to update (preserves your customizations)
CHANGELOG: See ~/templates/CHANGELOG.md for version history
-->

**STATUS: IN DEVELOPMENT** - Last Updated: 2025-10-19

## Repository Information
- **GitHub Repository**: https://github.com/Jimmyh-world/dev-blockchain
- **Local Directory**: `~/dev-blockchain`
- **Primary Purpose**: Centralized knowledge repository for blockchain development, serving as the foundation for blockchain RAG system (dev-rag project)

## Important Context

<!-- PROJECT_SPECIFIC START: IMPORTANT_CONTEXT -->
**Migration Source:** This repository consolidates blockchain knowledge from:
- dev-lab/docs/cardano/ (primary source)
- dev-lab/docs/midnight/
- dev-lab/docs/ergo/
- dev-lab/docs/research/cardano-rag/
- templates/cardano/ (duplicates removed)
- hash-licence/ai-dev-ref/ (duplicates removed)

**RAG Integration:** This repository serves as the knowledge base for the blockchain
RAG system documented in ~/dev-rag/. All blockchain documentation here is structured
for optimal RAG retrieval and will be ingested into the Qdrant vector database.

**Three-Machine Context:** This is a knowledge repository maintained on Chromebook
(Orchestrator). Content is research/documentation only - no code execution required.
<!-- PROJECT_SPECIFIC END: IMPORTANT_CONTEXT -->

## Core Development Principles (MANDATORY)

### 1. KISS (Keep It Simple, Stupid)
- Avoid over-complication and over-engineering
- Choose simple solutions over complex ones
- Question every abstraction layer
- If a feature seems complex, ask: "Is there a simpler way?"

### 2. TDD (Test-Driven Development)
- Write tests first
- Run tests to ensure they fail (Red phase)
- Write minimal code to pass tests (Green phase)
- Refactor while keeping tests green
- Never commit code without tests

### 3. Separation of Concerns (SOC)
- Each module/component has a single, well-defined responsibility
- Clear boundaries between different parts of the system
- Services should be loosely coupled
- Avoid mixing business logic with UI or data access code

### 4. DRY (Don't Repeat Yourself)
- Eliminate code duplication
- Extract common functionality into reusable components
- Use configuration files for repeated settings
- Create shared libraries for common operations

### 5. Documentation Standards
- Always include the actual date when writing documentation
- Use objective, factual language only
- Avoid marketing terms like "production-ready", "world-class", "highly sophisticated", "cutting-edge", etc.
- State current development status clearly
- Document what IS, not what WILL BE

### 5.5. AI-Optimized Documentation
**CRITICAL**: Documentation is structured data for both humans AND AI consumption

**Purpose**: Enable AI assistants to effectively help during:
- **Development** (now) - Building the system
- **Deployment** (later) - Setting up and configuring
- **Operations** (ongoing) - Monitoring, troubleshooting
- **User Support** (ongoing) - Helping users use the system

**Key Principles**:
1. **Structured Data Over Prose** - Use tables, JSON, YAML instead of paragraphs
2. **Explicit Context** - Never assume prior knowledge
3. **Cause-Effect Relationships** - Clear "if X then Y" statements
4. **Machine-Readable Examples** - Complete, runnable code blocks
5. **Searchable Patterns** - Consistent headings, markers, formats
6. **Version-Stamped** - Date all documentation updates
7. **Cross-Referenced** - Explicit links between related docs

**Example** (Good AI-optimized documentation):
```markdown
## Database Configuration

**Required Environment Variables**:
| Variable | Format | Example | Required |
|----------|--------|---------|----------|
| DATABASE_URL | postgresql://... | postgresql://postgres:secret@localhost:5432/db | Yes |

**Validation**:
\```bash
npm run db:test-connection
# Expected output: "✅ Connected successfully"
\```
```

**Documentation Layers**:
- **Layer 1**: Development Phase (AGENTS.md, Architecture docs, Phase plans)
- **Layer 2**: Deployment Phase (DEPLOYMENT-GUIDE.md, OPERATIONS.md, API-REFERENCE.md)
- **Layer 3**: User Phase (USER-GUIDE.md, TROUBLESHOOTING.md, Configuration patterns)

All documentation follows these principles to maximize AI assistant effectiveness.

### 6. Jimmy's Workflow (Red/Green Checkpoints)
**MANDATORY for all implementation tasks**

Use the Red/Green/Blue checkpoint system to prevent AI hallucination and ensure robust implementation:

- 🔴 **RED (IMPLEMENT)**: Write code, build features, make changes
- 🟢 **GREEN (VALIDATE)**: Run explicit validation commands, prove it works
- 🔵 **CHECKPOINT**: Mark completion with machine-readable status, document rollback

**Critical Rules:**
- NEVER skip validation phases
- NEVER proceed to next checkpoint without GREEN passing
- ALWAYS document rollback procedures
- ALWAYS use explicit validation commands (not assumptions)

**Reference**: See **JIMMYS-WORKFLOW.md** for complete workflow system, templates, and patterns

**Usage**: When working with AI assistants, say: *"Let's use Jimmy's Workflow to execute this plan"*

**Benefits:**
- Prevents "AI says done ≠ Actually done" problem
- Forces validation at every step
- Enables autonomous execution with safety gates
- Provides clear rollback paths
- Integrates seamlessly with TDD approach

### 7. YAGNI (You Ain't Gonna Need It)
- Don't implement features until they're actually needed
- Resist the urge to "future-proof" or add "might be useful later" code
- Build for current requirements, not hypothetical future ones
- Question every feature: "Do we need this NOW?"
- Refactor when requirements change, don't pre-optimize
- Every line of code is a liability - only write what's necessary

**Why This Matters:**
- Prevents scope creep and over-engineering
- Reduces technical debt (unused code is still debt)
- Speeds up development (focus on actual requirements)
- Forces clear prioritization of features

**AI Assistant Reminder:** Don't add "helpful" features like caching, abstraction layers, or config systems unless explicitly required by current needs.

### 8. Fix Now, Not Later
- Fix vulnerabilities immediately when discovered (npm audit, security warnings)
- Fix warnings immediately (don't suppress or accumulate)
- Fix failing tests immediately (understand root cause, don't skip)
- Fix linter errors immediately (don't disable rules without reason)
- Address build errors and deprecation warnings as they appear
- Don't use suppressions (@ts-ignore, eslint-disable, etc.) without documented justification

**Exception Clause:**
- If you MUST defer or bypass an issue:
  1. Investigate the root cause thoroughly
  2. Weigh multiple solution options
  3. Make an explicit decision
  4. DOCUMENT why (in code comments, KNOWN_ISSUES.md, or technical debt tracker)
  5. Create a tracking issue/ticket

**Why This Matters:**
- Prevents technical debt accumulation
- Keeps codebase healthy and maintainable
- Vulnerabilities don't ship to production
- Warnings don't become noise
- Tests remain meaningful

**AI Assistant Reminder:** Never suggest "we'll fix this later" or "skip for now". Always investigate root cause and fix immediately. If deferring is necessary, document comprehensively.

## GitHub Workflow

### Use GitHub CLI (gh) for All GitHub Operations

**Standard Tool**: Use `gh` CLI for all GitHub interactions (issues, PRs, CI/CD monitoring, releases)

**Installation**: `gh` should already be installed. Verify with `gh --version`

**Common Operations:**

**Pull Requests:**
```bash
gh pr create --title "Feature" --body "Description"
gh pr list                          # View open PRs
gh pr checks                        # Check CI/CD status
gh pr view [number]                 # View PR details
gh pr merge [number]                # Merge PR
```

**CI/CD Monitoring:**
```bash
gh run list                         # List workflow runs
gh run view [id]                    # View run details
gh run watch                        # Watch current run (live updates)
gh workflow list                    # List workflows
```

**Issues:**
```bash
gh issue create --title "Bug" --body "Description"
gh issue list                       # View open issues
gh issue view [number]              # View issue details
gh issue close [number]             # Close issue
```

**Releases:**
```bash
gh release create v1.0.0            # Create release
gh release list                     # List releases
gh release view [tag]               # View release details
```

**Why GitHub CLI:**
- ✅ Scriptable and automation-friendly
- ✅ Consistent across all projects
- ✅ Works seamlessly with AI assistants
- ✅ Faster than web UI for most operations
- ✅ Built-in CI/CD monitoring
- ✅ Integrates with Jimmy's Workflow checkpoints

**AI Assistant Note**: Always use `gh` commands instead of suggesting "check the GitHub web UI" or manual git operations for GitHub-specific tasks.

## Service Overview

<!-- PROJECT_SPECIFIC START: SERVICE_OVERVIEW -->
dev-blockchain is the **Single Source of Truth** for all blockchain development knowledge across three blockchain ecosystems:

1. **Cardano** (Primary): Aiken smart contracts, Atlas framework, security patterns, off-chain integration
2. **Midnight**: Privacy-focused blockchain research and deep technical documentation
3. **Ergo**: UTXO blockchain fundamentals and development basics

This repository consolidates ~500KB of unique blockchain knowledge previously scattered across 5+ locations, eliminating ~1MB of exact duplicates. All content is AI-optimized (tables, code blocks, explicit examples) and structured for RAG system ingestion.

**Key Responsibilities:**
- Centralize scattered blockchain documentation from dev-lab/, templates/, hash-licence/, etc.
- Provide AI-optimized knowledge base for blockchain RAG system (dev-rag project)
- Maintain up-to-date blockchain development best practices
- Enable efficient knowledge retrieval for AI assistants
- Serve as foundation for Qdrant vector database ingestion

**Important Distinctions:**
- This is a **DOCUMENTATION repository**, not a code repository
- Content is **knowledge/research**, not executable code
- Purpose is **RAG foundation**, not direct application development
- All content follows **factual, dated, AI-optimized** documentation standards
- This feeds into **dev-rag project's vector database** for retrieval
<!-- PROJECT_SPECIFIC END: SERVICE_OVERVIEW -->

## Current Status

<!-- PROJECT_SPECIFIC START: CURRENT_STATUS -->
🔄 **Migration in Progress** - 0% Complete (Phase 1 in progress)

**Migration Phases:**
- 🔄 Phase 1: Repository Initialization (in progress)
- ⚪ Phase 2: Content Migration (not started)
- ⚪ Phase 3: Duplicate Cleanup & Validation (not started)

**Content Status:**
- ⚪ Cardano knowledge (13 files, ~372KB) - pending migration
- ⚪ Midnight knowledge (2 files, ~70KB) - pending migration
- ⚪ Ergo knowledge (1 file, ~4KB) - pending migration
- ⚪ Research/RAG documentation (4 files, ~64KB) - pending migration

**Infrastructure:**
- 🔄 Git repository initialized
- 🔄 Template files copied (customization in progress)
- ⚪ GitHub repository not yet created
- ⚪ Cross-references not yet updated
<!-- PROJECT_SPECIFIC END: CURRENT_STATUS -->

## Technology Stack

**Documentation Format:**
- **Markdown**: All documentation in GitHub-flavored markdown
- **Format Standards**: AI-optimized (tables, code blocks, structured headings)
- **Version Control**: Git + GitHub

**Target Integration (Future):**
- **Vector Database**: Qdrant (from dev-rag project)
- **Embeddings**: mxbai-embed-large via Ollama
- **RAG Framework**: Custom (documented in dev-rag)

**No Build/Test Commands Required:**
This is a documentation-only repository. No compilation, testing, or deployment commands needed.

## Repository Structure

```
dev-blockchain/
├── cardano/                           # Cardano blockchain (primary)
│   ├── core/                          # Core concepts & workflow
│   ├── aiken/                         # Aiken smart contracts
│   ├── frameworks/                    # Development frameworks
│   ├── security/                      # Security patterns
│   ├── integration/                   # Off-chain integration
│   └── reference/                     # Supporting docs
├── midnight/                          # Midnight blockchain
├── ergo/                              # Ergo blockchain
├── research/                          # Cross-chain research
│   └── cardano-rag/                   # RAG system design
├── archive/                           # Historical records
│   └── migration-2025-10-19/          # Migration manifest
├── AGENTS.md                          # AI assistant guidelines (this file)
├── CLAUDE.md                          # Quick reference
├── JIMMYS-WORKFLOW.md                 # RED→GREEN→CHECKPOINT workflow
├── STATUS.md                          # Project status tracking
├── NEXT-SESSION-START-HERE.md         # Session continuity
└── README.md                          # Repository overview
```

## Development Workflow

### Starting Work on a Task
1. Read this AGENTS.md file for context
2. Check current implementation status above
3. Review known issues and TODOs below
4. **Use Jimmy's Workflow**: Plan → Implement → Validate → Checkpoint
5. Follow TDD approach - write tests first
6. Implement minimal code to pass tests
7. Refactor while maintaining green tests

### Before Committing Documentation
1. Check for unfilled placeholders: `grep "\[" *.md | grep -v http | grep -v example`
2. Verify all dates are current and factual
3. Ensure no credentials or sensitive data exposed
4. Validate markdown formatting
5. Update STATUS.md with progress
6. Use Jimmy's Workflow checkpoints to validate completeness

### Documentation Updates
1. Update README.md with any API changes
2. Add inline comments for complex logic
3. Update this AGENTS.md if development approach changes
4. Document all decisions with dates
5. Keep development diary current (if applicable)

## Known Issues & Technical Debt

<!-- PROJECT_SPECIFIC START: KNOWN_ISSUES -->
### 🔴 Critical Issues
None currently. Migration is in progress (Phase 1).

### 🟡 Important Issues
1. Content migration pending (Phase 2 not yet started)
2. Old duplicate locations need deprecation notices (Phase 3)

### 📝 Technical Debt
1. Update all blockchain knowledge to latest versions (post-migration)
   - Estimated effort: Ongoing maintenance
   - Priority: Medium (after migration complete)
<!-- PROJECT_SPECIFIC END: KNOWN_ISSUES -->

## Project-Specific Guidelines

<!-- PROJECT_SPECIFIC START: PROJECT_SPECIFIC_GUIDELINES -->
This is a documentation-only repository. Guidelines focus on documentation quality and organization.

### Documentation Style
- Use GitHub-flavored markdown
- AI-optimized format (tables, code blocks, clear headings)
- Include dates in all documentation
- Use factual, objective language only
- No marketing terms or speculative language

### File Organization
- Organize by blockchain (cardano/, midnight/, ergo/)
- Each directory has README.md overview
- Use subdirectories for logical grouping (core/, aiken/, security/, etc.)
- Include source citations and last updated dates

### Content Quality Standards
- All claims must be verifiable (link to sources)
- Code examples must be complete and runnable
- Include metadata (dates, versions, sources)
- Cross-reference related docs explicitly
- Optimize for RAG retrieval (clear hierarchies, searchable patterns)

### Security Considerations
- [Project-specific security requirements]
- [Sensitive data handling]
- [Authentication/authorization patterns]
<!-- PROJECT_SPECIFIC END: PROJECT_SPECIFIC_GUIDELINES -->

## Common Patterns & Examples

<!-- PROJECT_SPECIFIC START: COMMON_PATTERNS -->
### [Pattern Name 1]
[Description of common pattern used in this project]

```[language]
// Example code
```

### [Pattern Name 2]
[Description of another common pattern]

```[language]
// Example code
```
<!-- PROJECT_SPECIFIC END: COMMON_PATTERNS -->

## Dependencies & Integration

<!-- PROJECT_SPECIFIC START: DEPENDENCIES -->
### External Services
- **[Service Name 1]**: [Purpose and integration details]
- **[Service Name 2]**: [Purpose and integration details]

### Related Services (if part of multi-service platform)
- **[Related Service 1]**: [Relationship and dependencies]
- **[Related Service 2]**: [Relationship and dependencies]
<!-- PROJECT_SPECIFIC END: DEPENDENCIES -->

## Environment Variables

<!-- PROJECT_SPECIFIC START: ENVIRONMENT_VARIABLES -->
```bash
# Required
[REQUIRED_VAR_1]=[description]
[REQUIRED_VAR_2]=[description]

# Optional
[OPTIONAL_VAR_1]=[description] # Default: [default_value]
[OPTIONAL_VAR_2]=[description] # Default: [default_value]
```
<!-- PROJECT_SPECIFIC END: ENVIRONMENT_VARIABLES -->

## Troubleshooting

<!-- PROJECT_SPECIFIC START: TROUBLESHOOTING -->
### Common Issues

**Issue**: [Common problem description]
**Solution**: [How to fix it]

**Issue**: [Another common problem]
**Solution**: [How to fix it]
<!-- PROJECT_SPECIFIC END: TROUBLESHOOTING -->

## Resources & References

### Documentation
- [Link to main documentation]
- [Link to API documentation]
- [Link to architecture diagrams]

### External Resources
- [Relevant external documentation]
- [Tutorial or guide links]
- [Community resources]

## Template Version Management

**Current Template Version**: See `<!-- TEMPLATE_VERSION -->` comment at top of this file

**This project uses versioned templates** from `/home/jimmyb/templates/`

### Check if Templates are Up to Date

```bash
~/templates/tools/check-version.sh
```

**What it does:**
- Compares your AGENTS.md version with master template version
- Exit code 0 = up to date ✅
- Exit code 1 = out of date ⚠️

### View Template Changelog

```bash
cat ~/templates/CHANGELOG.md
```

See what's new in each version and migration instructions.

### Sync to Latest Version (Manual for now)

```bash
~/templates/tools/sync-templates.sh --dry-run   # Preview changes
~/templates/tools/sync-templates.sh             # Apply changes (with confirmation)
~/templates/tools/sync-templates.sh --auto      # Auto-apply without confirmation
```

**What gets preserved during sync:**
- ✅ All `<!-- PROJECT_SPECIFIC -->` sections (your customizations)
- ✅ All placeholder values (PROJECT_NAME, commands, etc.)
- ✅ Custom additions to Known Issues, Technical Debt, etc.
- ✅ Project-specific guidelines and patterns

**What gets updated during sync:**
- 🔄 Core Development Principles (if new ones added)
- 🔄 Template structure improvements
- 🔄 Standard sections and formatting
- 🔄 Tool integrations (GitHub CLI, etc.)

**Important Notes:**
- Always review the diff before applying
- Backups are created automatically in `.template-sync-backup/`
- If sync fails, restore from backup
- Commit template updates separately from feature work

### Template Compliance Checking

**AI Assistant Behavior**: When user asks "check templates" or "are we up to date?", automatically:
1. Run: `~/templates/tools/audit-project.sh` OR manually execute checks
2. Generate compliance report
3. Offer remediation based on findings

**User can also run manually:**
```bash
~/templates/tools/audit-project.sh          # Full audit
~/templates/tools/audit-project.sh --quick  # Quick check
```

**Quick Manual Check:**
```bash
# Are we up to date?
~/templates/tools/check-version.sh

# What's new?
cat ~/templates/CHANGELOG.md
```

## Important Reminders for AI Assistants

1. **Always use Jimmy's Workflow** for implementation tasks
2. **Follow TDD** - Write tests before implementation
3. **Keep it KISS** - Simplicity over complexity
4. **Apply YAGNI** - Only implement what's needed now, not future "might need" features
5. **Use GitHub CLI** - Use `gh` for all GitHub operations (PRs, issues, CI/CD monitoring)
6. **Fix Now** - Never defer fixes for vulnerabilities, warnings, or test failures. No suppressions without documented justification
7. **Document dates** - Include actual dates in all documentation
8. **Validate explicitly** - Run commands, don't assume
9. **Never skip checkpoints** - Each phase must complete before proceeding
10. **Update this file** - Keep AGENTS.md current as project evolves

---

**This document follows the [agents.md](https://agents.md/) standard for AI coding assistants.**

**Template Version**: 1.0
**Last Updated**: [YYYY-MM-DD]
