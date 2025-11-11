# Pipeline Stage Correction Summary

## Issue Identified

The GitHub repository documentation originally described a **4-stage pipeline** with an "LLM-as-a-Judge" quality assessment stage, but the actual research paper only implements and describes a **3-stage pipeline**.

## Changes Made (2024-12-12)

### 1. Deleted Files
- **`prompts/judge.yaml`** - Removed as Stage 4 was not actually used in the paper

### 2. Updated Files

#### **README.md**
- Changed: "Multi-Stage Extraction Pipeline" → "**Three-Stage Extraction Pipeline**"
- Removed: All references to Stage 4 and judge.yaml
- Updated pipeline description:
  - Stage 1: Weak Extraction (weak_extraction.yaml)
  - Stage 2: Strong Verification (strong_extraction.yaml)
  - Stage 3: RAG-Enhanced Validation (verification.yaml)
- Updated repository structure section to show only 3 YAML files

#### **docs/extraction_protocol.md**
- Line 5: Changed "four-stage LLM pipeline" → "**three-stage LLM pipeline**"
- Lines 36-44: Updated Stage 3 output to show final 77,734 triples (removed Stage 4 box)
- Lines 224-252: **Removed entire "Stage 4: Quality Assessment (LLM-as-a-Judge)" section**
- Line 415: Changed "LLM-as-a-Judge for final validation" → "**Semantic deduplication with hierarchical clustering**"
- Removed Zheng et al. (2023) reference about LLM-as-a-Judge

#### **main.tex** (Manuscript)
- Line 1108: Changed "**Four-stage** prompt templates (weak extraction, strong extraction, verification, quality assessment)" → "**Three-stage** prompt templates (weak extraction, strong verification, RAG-enhanced validation)"

### 3. Git Commits

**Overleaf Repository:**
- Commit: `eb9df91` - "Fix Data Availability Statement: correct 4-stage to 3-stage pipeline"
- Pushed to: https://git.overleaf.com/68f830c10a707d7841a448db

## Corrected Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      INPUT: PDF Report                          │
│           (68 pages avg, Japanese/English text)                 │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 1: Weak Extraction (weak_extraction.yaml)               │
│  • High-recall seed extraction (≤3 triples/sentence)            │
│  • Abstract verb filtering                                      │
│  • Ontology constraint enforcement                              │
│  Output: ~92,000 raw triples                                    │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 2: Strong Verification (strong_extraction.yaml)         │
│  • Context-enriched re-extraction with RAG                      │
│  • 3-layer context: Local + Document + Graph                    │
│  • Evidence grounding (verbatim 8-400 char quotes)              │
│  • Specific verb normalization                                  │
│  Output: ~85,000 context-aware triples                          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 3: RAG-Enhanced Validation (verification.yaml)          │
│  • 4-dimensional confidence scoring:                            │
│    - Evidence consistency (weight: 0.4)                         │
│    - Topic alignment (weight: 0.2)                              │
│    - Action specificity (weight: 0.2)                           │
│    - Context consistency (weight: 0.2)                          │
│  • Threshold filtering (confidence ≥ 0.60)                      │
│  • Deduplication and entity standardization                     │
│  Output: 77,734 high-quality triples (97.7% conf ≥ 0.9)         │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              OUTPUT: Knowledge Graph (Neo4j)                    │
│  • Nodes: Companies (141), Topics (39), Actions (77K)          │
│  • Edges: DISCLOSES, BELONGS_TO, PERFORMED_BY, TARGETS         │
└─────────────────────────────────────────────────────────────────┘
```

## Files Now Consistent

✅ **main.tex** (manuscript) - Describes 3-stage pipeline
✅ **README.md** - Describes 3-stage pipeline
✅ **docs/extraction_protocol.md** - Describes 3-stage pipeline
✅ **prompts/** directory - Contains exactly 3 YAML files

## Verification

All documentation now accurately reflects the actual methodology implemented in the research paper:
- Paper Section 3: Describes Stages 1-3 only
- No mention of "Stage 4", "judge.yaml", or "LLM-as-a-Judge" in the paper methodology
- Final output of 77,734 triples comes from Stage 3 verification, not a separate Stage 4

---

**Last Updated**: 2024-12-12
**Issue Reporter**: User (Tong Xu)
**Resolution**: Complete
