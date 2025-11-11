# ESG Action Extraction: LLM-Driven Knowledge Graph Construction

[![Paper](https://img.shields.io/badge/Paper-Cogent%20B%26M-blue)](https://www.tandfonline.com/toc/oabm20/current)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Dataset](https://img.shields.io/badge/Dataset-Zenodo-orange)](https://zenodo.org)

This repository contains the **LLM extraction prompts** and **methodological protocols** accompanying the research paper:

> **Beyond Ratings: Deconstructing ESG Disclosure Through AI-Driven Action-Level Analysis**
> *Cogent Business & Management*, 2025

## 📋 Overview

We propose a novel framework for extracting structured ESG (Environmental, Social, Governance) action records from corporate sustainability reports using Large Language Models (LLMs). Instead of relying on opaque third-party ESG ratings, our method:

- **Extracts granular actions**: 77,734 action records from 141 Japanese listed companies
- **Uses knowledge graphs**: Constructs multi-relational networks linking companies, topics, actors, targets, and actions
- **Provides transparency**: All extraction logic is documented and reproducible
- **Achieves high accuracy**: 89.5% validation accuracy (Cohen's κ=0.95) with 97.7% of records exceeding 0.9 confidence

## 🎯 Key Features

### Multi-Stage Extraction Pipeline

1. **Weak Extraction** (`weak_extraction.yaml`): High-recall initial extraction (≤3 triples per sentence)
2. **Strong Extraction** (`strong_extraction.yaml`): Context-enriched extraction with RAG (Retrieval-Augmented Generation)
3. **Verification** (`verification.yaml`): Four-dimensional confidence scoring (evidence, topic, action, context)
4. **Quality Assessment** (`judge.yaml`): LLM-as-a-Judge evaluation with 5 rubrics

### Eight-Element Action Schema

Each extracted ESG action contains:
- **Company**: Corporate entity (e.g., "Toyota Motor Corp")
- **Topic**: ESG category (39 topics from GRI/SASB/TCFD frameworks)
- **Actor**: Who performs the action (company/board/employees/suppliers)
- **Action**: Specific verb (install/reduce/train/audit/disclose)
- **Target**: Beneficiary or affected entity (employees/facilities/emissions)
- **Modality**: Temporal framing (past_action/ongoing/future_commitment)
- **Evidence**: Verbatim text span from source document (8-400 characters)
- **Confidence**: Multi-dimensional score (0.60-1.00)

## 📂 Repository Structure

```
.
├── prompts/
│   ├── weak_extraction.yaml       # Stage 1: Seed triple generation
│   ├── strong_extraction.yaml     # Stage 2: Context-aware extraction
│   ├── verification.yaml          # Stage 3: Confidence scoring
│   └── judge.yaml                 # Stage 4: Quality assessment
├── schema/
│   └── action_record_schema.json  # JSON schema for extracted records
├── examples/
│   ├── sample_input.txt           # Example ESG report excerpt
│   └── sample_output.json         # Example extracted triples
├── docs/
│   ├── extraction_protocol.md     # Detailed methodology
│   └── ontology_39topics.md       # ESG topic taxonomy
├── LICENSE
└── README.md
```

## 🚀 Quick Start

### Prerequisites

- Python 3.9+
- OpenAI API key (GPT-4o recommended)
- Neo4j database (for knowledge graph storage)

### Basic Usage

```python
import yaml
from openai import OpenAI

# Load prompt template
with open('prompts/strong_extraction.yaml', 'r', encoding='utf-8') as f:
    prompt_config = yaml.safe_load(f)

# Configure extraction
client = OpenAI(api_key="your-api-key")
company = "Toyota Motor Corp"
sentence = "工場AでLED照明へ更新し、年間電力使用量を12%削減。"
year = 2024

# Format prompt
user_prompt = prompt_config['user'].format(
    company=company,
    sentence=sentence,
    year=year,
    page=42,
    # ... additional context parameters
)

# Extract triples
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": prompt_config['system']},
        {"role": "user", "content": user_prompt}
    ],
    temperature=0.1,
    response_format={"type": "json_object"}
)

# Parse output
import json
triples = json.loads(response.choices[0].message.content)
print(json.dumps(triples, indent=2, ensure_ascii=False))
```

**Expected Output:**
```json
[
  {
    "topic": "energy_efficiency",
    "action": "install_led_lighting",
    "actor": "Toyota Motor Corp",
    "target": "factory_A",
    "modality": "past_action",
    "evidence_page": 42,
    "evidence_span_text": "工場AでLED照明へ更新し、年間電力使用量を12%削減。"
  }
]
```

## 📊 Dataset Access

### Corporate Sustainability Reports (Public)

All 141 sustainability reports analyzed in this study are publicly available:

- **Zenodo Repository**: [![DOI](https://img.shields.io/badge/DOI-10.5281%2Fzenodo.XXXXX-blue)](https://zenodo.org/record/XXXXX)
- **Format**: PDF files (141 companies × 1 report each, ~2.3 GB total)
- **Coverage**: Japanese listed companies, fiscal year 2024
- **License**: CC BY 4.0

### Extracted ESG Action Dataset (Upon Request)

The structured dataset of 77,734 action records is available from the corresponding author upon reasonable request:

- **Format**: JSON/CSV (45 MB)
- **Conditions**: Non-commercial research use, data use agreement required
- **Contact**: [corresponding author email]

## 📖 Documentation

### Extraction Protocol

See [`docs/extraction_protocol.md`](docs/extraction_protocol.md) for:
- Detailed prompt engineering strategies
- RAG (Retrieval-Augmented Generation) implementation
- Confidence scoring formula
- Validation procedures

### ESG Topic Taxonomy

Our 39-topic taxonomy ([`docs/ontology_39topics.md`](docs/ontology_39topics.md)) integrates:
- **GRI Standards** (Global Reporting Initiative)
- **SASB Materiality Map** (Sustainability Accounting Standards Board)
- **TCFD Recommendations** (Task Force on Climate-related Financial Disclosures)
- **Japan-specific topics** (e.g.,労働慣行, keiretsu governance)

**Dimensional breakdown**:
- Environmental (E): 14 topics
- Social (S): 15 topics
- Governance (G): 10 topics

## 🎓 Citation

If you use this code or dataset in your research, please cite:

```bibtex
@article{xu2025esg,
  title={Beyond Ratings: Deconstructing ESG Disclosure Through AI-Driven Action-Level Analysis},
  author={Xu, Tong and [Co-authors]},
  journal={Cogent Business \& Management},
  year={2025},
  publisher={Taylor \& Francis},
  note={Code and prompts available at: https://github.com/[your-username]/esg-action-extraction}
}
```

## 📜 License

This project is licensed under the **MIT License** - see [LICENSE](LICENSE) file for details.

**Summary**:
- ✅ Free to use, modify, and distribute
- ✅ Commercial use permitted
- ✅ Attribution required
- ❌ No warranty provided

## 🤝 Contributing

We welcome contributions! Please:
1. Fork this repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

**Areas for contribution**:
- Multi-language support (English, Chinese, Korean reports)
- Additional ESG topic ontologies (ISSB, EU CSRD)
- Improved entity standardization algorithms
- Visualization tools for knowledge graphs

## 🔬 Research Context

### Key Findings

From our analysis of 141 Japanese firms:

1. **Social-dominant disclosure** (55.4% vs. 26.9% environmental)
2. **Action-oriented rhetoric** (65.5% ongoing actions vs. 19.1% future commitments)
3. **Employee-centric stakeholder structure** (10.2% employee targets vs. 0.5% communities)
4. **Three strategic archetypes**: Balanced (38%), Social-oriented (43%), Environment-oriented (18%)

### Advantages Over Traditional ESG Ratings

| Feature | Traditional Ratings | Our Method |
|---------|-------------------|-----------|
| **Transparency** | Opaque scoring formulas | Fully documented prompts |
| **Granularity** | Aggregate scores (0-100) | 77,734 individual actions |
| **Customization** | Fixed methodology | User-defined queries |
| **Timeliness** | Annual/quarterly updates | Real-time extraction |
| **Cost** | $10,000-50,000/year | ~$2,400 one-time (API costs) |

## 📞 Contact

- **Corresponding Author**: Tong Xu
- **Institution**: [Your University/Institution]
- **Email**: [your-email]
- **Paper**: [Link to published paper]
- **Dataset Requests**: [Contact form or email]

## 🙏 Acknowledgments

This research was supported by:
- [Funding agency 1]
- [Funding agency 2]

We thank:
- OpenAI for GPT-4o API access
- Neo4j for graph database infrastructure
- 141 Japanese companies for publicly disclosing sustainability reports

## 📚 Related Resources

- **Original Paper**: [Link to Cogent B&M publication]
- **Supplementary Materials**: [Link to Overleaf/OSF]
- **Dataset Repository**: [Zenodo DOI]
- **Demo Application**: [Link to web app, if available]

---

**Disclaimer**: This research is for academic purposes only. ESG action extraction accuracy (89.5%) indicates that outputs should be verified before use in investment decisions. The authors are not responsible for financial losses resulting from reliance on this methodology.

**Last Updated**: December 2024
