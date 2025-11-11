# ESG Action Extraction Protocol

## Overview

This document provides detailed instructions for extracting structured ESG action records from corporate sustainability reports using our four-stage LLM pipeline.

## Pipeline Architecture

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
│  Stage 2: Strong Extraction (strong_extraction.yaml)           │
│  • Context-enriched re-extraction with RAG                      │
│  • 3-layer context: Local + Document + Graph                    │
│  • Evidence grounding (verbatim 8-400 char quotes)              │
│  • Specific verb normalization                                  │
│  Output: ~85,000 context-aware triples                          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 3: Verification (verification.yaml)                     │
│  • 4-dimensional confidence scoring:                            │
│    - Evidence consistency (weight: 0.4)                         │
│    - Topic alignment (weight: 0.2)                              │
│    - Action specificity (weight: 0.2)                           │
│    - Context consistency (weight: 0.2)                          │
│  • Threshold filtering (confidence ≥ 0.60)                      │
│  Output: ~81,000 filtered triples                               │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 4: Quality Assessment (judge.yaml)                      │
│  • LLM-as-a-Judge 5-rubric evaluation (0-2 scale each):         │
│    1. Evidence consistency                                      │
│    2. Action specificity                                        │
│    3. Topic alignment                                           │
│    4. Entity reasonableness                                     │
│    5. Non-redundancy                                            │
│  • Pass threshold: total_score ≥ 7/10                           │
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

## Stage 1: Weak Extraction

### Purpose
Generate high-recall seed triples that maximize coverage of potential ESG actions, even if some are low-quality.

### Key Parameters
- **Temperature**: 0.3 (moderate creativity for paraphrasing detection)
- **Max tokens**: 2000
- **Top-p**: 0.95

### Prompt Engineering Strategies

#### 1. Few-Shot Examples
```yaml
# Positive example
sentence: "2024年度、CO2排出量を前年比30%削減した。"
output: [{"topic":"emissions_reduction","action":"reduce_emissions",...}]

# Negative example (abstract verb)
sentence: "環境への取り組みを強化する。"
output: []  # Rejected due to abstract verb "強化"
```

#### 2. Ontology Constraints
- **Topic**: Must map to leaf node in 39-topic taxonomy (see `ontology_39topics.md`)
- **Action**: Must use specific verb (install/reduce/train/audit), not generic (推進/強化/実施)

#### 3. Language Handling
- Japanese input text preserved
- Schema keys in English (for compatibility)
- Output: Pure JSON array, no markdown code blocks

### Quality Filters
- Reject triples with abstract verbs: `取り組む, 推進, 強化, 実施, 実行, 対応`
- Reject triples without ontology match
- Limit: ≤3 triples per sentence

---

## Stage 2: Strong Extraction

### Purpose
Re-extract with full context to resolve ambiguities, verify specificity, and ground evidence.

### RAG (Retrieval-Augmented Generation) Implementation

#### Layer 1: Local Context (±k sentences)
```python
k_local = 3  # Retrieve 3 sentences before and after target
local_context = report[sentence_idx - k_local : sentence_idx + k_local + 1]
```

**Purpose**: Resolve pronouns, coreferences, and implied actors/targets.

**Example**:
```
Target sentence: "これにより、CO2排出量を300トン削減した。"
Local context: "工場AでLED照明を設置した。これにより、..."
Resolved: target = "factory_A"
```

#### Layer 2: Document-Level Context (top-k chunks)
```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
query_embedding = model.encode(target_sentence)
doc_embeddings = model.encode(all_chunks)

# Retrieve top-5 most similar chunks
similarities = cosine_similarity([query_embedding], doc_embeddings)[0]
top_k_indices = similarities.argsort()[-5:][::-1]
doc_context = [all_chunks[i] for i in top_k_indices]
```

**Purpose**: Provide domain-specific terminology and related actions for disambiguation.

**Example**:
```
Ambiguous: "排出量削減"
Doc context: "スコープ1: 直接排出, スコープ2: 間接排出, スコープ3: サプライチェーン"
Clarified: topic = "scope1_emissions" (not generic "emissions")
```

#### Layer 3: Graph Neighborhood Context
```cypher
MATCH (c:Company {name: $company})-[:DISCLOSES]->(a:Action)-[:BELONGS_TO]->(t:Topic)
WHERE a.year = $year - 1
RETURN t.name, COUNT(a) as action_count
ORDER BY action_count DESC
LIMIT 10
```

**Purpose**: Leverage historical disclosure patterns to infer likely topics/actions.

**Example**:
```
Target: "training program"
Graph shows: company disclosed 47 "diversity_inclusion" + 23 "talent_development" actions last year
Inference: If target mentions "managers" → likely "diversity_inclusion"
```

### Evidence Grounding

#### Requirements
- **Length**: 8-400 characters
- **Match**: Exact substring from local/doc/graph context (no paraphrasing)
- **Coverage**: Must support all extracted elements (actor, action, target)

#### Validation
```python
def validate_evidence(evidence_text, context):
    # Normalize whitespace
    evidence_clean = ' '.join(evidence_text.split())
    context_clean = ' '.join(context.split())

    # Check exact match
    if evidence_clean in context_clean:
        return True
    else:
        return False  # Reject triple
```

---

## Stage 3: Verification

### Confidence Scoring Formula

```
Confidence = 0.4 × S_evidence + 0.2 × S_topic + 0.2 × S_action + 0.2 × S_context
```

Where each score ∈ {0.0, 0.5, 1.0}.

#### Dimension 1: Evidence Consistency (S_evidence, weight=0.4)
- **1.0**: Evidence text exactly matches context, covers all extraction elements
- **0.5**: Evidence partially matches, some elements inferred
- **0.0**: Evidence not found in context, or contradicts extraction

**Why highest weight?** Answer faithfulness is the most critical quality dimension in RAG systems (Saad-Falcon et al., 2024).

#### Dimension 2: Topic Alignment (S_topic, weight=0.2)
- **1.0**: Topic correctly maps to GRI/SASB/TCFD leaf node
- **0.5**: Topic maps to parent node only (e.g., "environment" instead of "emissions_reduction")
- **0.0**: Topic mismatch or not in ontology

#### Dimension 3: Action Specificity (S_action, weight=0.2)
- **1.0**: Specific, verifiable verb (install/reduce/train/audit/disclose)
- **0.5**: Somewhat abstract but interpretable (improve/enhance/promote)
- **0.0**: Generic verb (取り組む/推進/強化)

#### Dimension 4: Context Consistency (S_context, weight=0.2)
- **1.0**: Extraction aligns with local+doc+graph context
- **0.5**: Minor inconsistencies (e.g., year mismatch)
- **0.0**: Contradicts known facts

### Threshold Selection

Empirical calibration (100-sample validation):
- **Conf ≥ 0.90**: 97.7% accuracy (Cohen's κ=0.95)
- **Conf 0.70-0.89**: 85.7% accuracy
- **Conf 0.60-0.69**: 77.5% accuracy
- **Conf < 0.60**: 62.3% accuracy (rejected)

**Selected threshold**: 0.60 (balances precision and recall)

---


---

## Deduplication Strategy

### Semantic Clustering

```python
from sentence_transformers import SentenceTransformer
from sklearn.cluster import AgglomerativeClustering

# Encode action descriptions
model = SentenceTransformer('all-MiniLM-L6-v2')
action_texts = [f"{a['action']} {a['target']}" for a in actions]
embeddings = model.encode(action_texts)

# Hierarchical clustering
clustering = AgglomerativeClustering(
    n_clusters=None,
    distance_threshold=0.15,  # Cosine similarity > 0.85
    linkage='average'
)
labels = clustering.fit_predict(embeddings)

# Within each cluster, keep highest confidence record
for cluster_id in set(labels):
    cluster_actions = [a for a, l in zip(actions, labels) if l == cluster_id]
    best_action = max(cluster_actions, key=lambda x: x['confidence'])
    deduped_actions.append(best_action)
```

### Manual Rules

1. **Same company + topic + action + target + modality**: Keep highest confidence
2. **Same evidence_span_text**: Merge if all other fields identical
3. **Temporal variants**: Keep most recent modality (e.g., prefer "past_action" over "future_commitment" if both exist for same underlying initiative)

---

## Entity Standardization

### Actor/Target Normalization

```python
STANDARD_ACTORS = {
    "workers": "employees",
    "staff": "employees",
    "workforce": "employees",
    "管理職": "management",
    "取締役会": "board_of_directors",
    ...
}

STANDARD_TARGETS = {
    "CO2排出": "CO2_emissions",
    "GHG排出": "GHG_emissions",
    "温室効果ガス": "GHG_emissions",
    "地域社会": "local_communities",
    "取引先": "suppliers",
    ...
}

def standardize_entity(entity, mapping):
    entity_lower = entity.lower().strip()
    if entity_lower in mapping:
        return mapping[entity_lower]
    else:
        # Use semantic similarity for fuzzy matching
        best_match = find_closest_match(entity, mapping.keys())
        if similarity(entity, best_match) > 0.8:
            return mapping[best_match]
    return entity  # Keep original if no match
```

---

## Performance Metrics

### Extraction Speed

- **Weak extraction**: ~0.5 sec/sentence (GPT-4o-mini)
- **Strong extraction**: ~1.2 sec/sentence (GPT-4o)
- **Verification**: ~0.3 sec/triple (GPT-4o-mini)
- **Judge**: ~0.8 sec/triple (GPT-4o)

**Total pipeline**: ~8 hours for 141 reports (68 pages avg)

### API Costs (as of Nov 2024)

- **GPT-4o**: $0.0025/1K input tokens, $0.010/1K output tokens
- **GPT-4o-mini**: $0.000150/1K input tokens, $0.000600/1K output tokens

**Total cost for 141 companies**: ~USD $2,400

### Accuracy Validation

Manual validation (100 stratified samples):
- **Overall accuracy**: 89.5% (all 8 elements correct)
- **Inter-rater reliability**: Cohen's κ = 0.95 (almost perfect agreement)
- **High-confidence subset** (conf ≥ 0.9): 97.7% accuracy

---

## Common Error Patterns

### Error Type 1: Overly Abstract Actions (54.5% of errors)

**Example**:
```json
{
  "action": "implement",  // ❌ Too generic
  "target": "measures"    // ❌ No specific object
}
```

**Fix**: Require specific verb + concrete object:
```json
{
  "action": "install_solar_panels",  // ✅ Specific
  "target": "factory_roof"          // ✅ Concrete
}
```

### Error Type 2: Incorrect Target (18.2%)

**Example**:
```json
{
  "action": "train",
  "target": "diversity"  // ❌ Topic, not stakeholder
}
```

**Fix**:
```json
{
  "action": "train",
  "target": "employees"  // ✅ Correct stakeholder
}
```

### Error Type 3: Compound Actions (18.2%)

**Example**:
```json
{
  "action": "install_and_monitor",  // ❌ Should be 2 triples
  ...
}
```

**Fix**: Split into atomic actions:
```json
[
  {"action": "install_solar_panels", ...},
  {"action": "monitor_energy_output", ...}
]
```

### Error Type 4: Evidence Mismatch (9.1%)

**Example**:
```json
{
  "evidence_span_text": "reduced emissions",  // ❌ Paraphrased
  // Original: "CO2排出量を削減した"
}
```

**Fix**: Exact quote:
```json
{
  "evidence_span_text": "CO2排出量を削減した",  // ✅ Verbatim
}
```

---

## Best Practices

### 1. Prompt Engineering
- ✅ Use few-shot examples (positive + negative)
- ✅ Enforce JSON-only output (no markdown)
- ✅ Provide ontology constraints in prompt
- ❌ Avoid open-ended generation without constraints

### 2. Context Management
- ✅ Use hierarchical RAG (local → doc → graph)
- ✅ Limit context window to 8K tokens (GPT-4o)
- ✅ Cache document embeddings for efficiency
- ❌ Don't exceed context limits (causes hallucination)

### 3. Quality Control
- ✅ Multi-dimensional confidence scoring
- ✅ LLM-as-a-Judge for final validation
- ✅ Human validation of random samples (n≥100)
- ❌ Don't rely on single confidence metric

### 4. Scalability
- ✅ Batch API requests (up to 50 parallel)
- ✅ Use cheaper models for simple tasks (weak extraction)
- ✅ Implement exponential backoff for rate limits
- ❌ Don't process reports sequentially

---

## References

1. Saad-Falcon et al. (2024). ARES: An Automated Evaluation Framework for Retrieval-Augmented Generation Systems. *NAACL*.
2. Zheng et al. (2023). Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena. *arXiv:2306.05685*.
3. Pan et al. (2024). Unifying Large Language Models and Knowledge Graphs: A Roadmap. *IEEE TKDE*.
