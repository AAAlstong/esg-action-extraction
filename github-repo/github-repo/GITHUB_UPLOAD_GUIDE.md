# GitHub Repository Upload Guide

## 📦 What's in This Package

This `github-repo/` directory contains all files ready for uploading to GitHub:

```
github-repo/
├── README.md                          # Main repository documentation
├── LICENSE                            # MIT License
├── CITATION.cff                       # Citation metadata
├── prompts/
│   ├── weak_extraction.yaml           # Stage 1 prompt
│   ├── strong_extraction.yaml         # Stage 2 prompt
│   ├── verification.yaml              # Stage 3 prompt
│   └── judge.yaml                     # Stage 4 prompt
├── schema/
│   └── action_record_schema.json      # JSON schema definition
├── examples/
│   ├── sample_input.txt               # Example ESG report text
│   └── sample_output.json             # Example extracted records
└── docs/
    ├── extraction_protocol.md         # Detailed methodology
    └── ontology_39topics.md           # ESG taxonomy (to be created)
```

## 🚀 Step-by-Step Upload Instructions

### Step 1: Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `esg-action-extraction`
3. Description: "LLM-driven ESG action extraction prompts and knowledge graph construction (Cogent B&M 2025)"
4. Visibility: **Public** ✅
5. Initialize with: **None** (we have files ready)
6. Click "Create repository"

### Step 2: Upload Files via GitHub Web Interface

**Option A: Drag-and-Drop (Easiest)**

1. On your new repository page, click "uploading an existing file"
2. Drag all files from `github-repo/` folder
3. Commit message: "Initial commit: LLM prompts and extraction protocols"
4. Click "Commit changes"

**Option B: Git Command Line**

```bash
cd /path/to/esg_graphrag/github-repo

# Initialize git
git init
git add .
git commit -m "Initial commit: LLM prompts and extraction protocols"

# Connect to GitHub
git remote add origin https://github.com/[YOUR-USERNAME]/esg-action-extraction.git
git branch -M main
git push -u origin main
```

### Step 3: Configure Repository Settings

1. Go to repository Settings → General
2. Enable:
   - ✅ Issues
   - ✅ Wikis (optional)
   - ✅ Discussions (optional)
3. Add topics (keywords):
   - `esg`
   - `sustainability`
   - `llm`
   - `knowledge-graphs`
   - `natural-language-processing`
   - `information-extraction`

### Step 4: Update URLs in Files

After uploading, replace placeholders in these files:

#### README.md
- Line 3: Replace `[your-username]` with your GitHub username
- Line 75: Update repository URL
- Line 111: Update Zenodo DOI (after uploading PDFs)

#### CITATION.cff
- Line 9: Add your ORCID
- Line 10: Replace `[your-username]`
- Line 29: Add paper DOI when published

### Step 5: Create Release (Optional)

1. Go to "Releases" → "Create a new release"
2. Tag version: `v1.0.0`
3. Release title: "Initial Release: ESG Action Extraction v1.0"
4. Description:
```
First stable release of LLM extraction prompts accompanying the paper:
"Beyond Ratings: Deconstructing ESG Disclosure Through AI-Driven Action-Level Analysis"

## What's Included
- 4 YAML prompt templates (weak/strong extraction, verification, judge)
- JSON schema for 8-element action records
- Sample input/output examples
- Detailed extraction protocol documentation
- MIT License

## Usage
See README.md for quick start guide and API integration examples.
```
5. Click "Publish release"

## 📊 Next Steps: Upload PDFs to Zenodo

### Why Zenodo?
- ✅ Free unlimited storage
- ✅ Permanent DOI
- ✅ Taylor & Francis recommended
- ✅ Academic standard

### Upload Process

1. **Create Account**: https://zenodo.org/signup
   - Use ORCID or GitHub login

2. **Create New Upload**:
   - Click "Upload" → "New upload"
   - Upload type: "Dataset"

3. **Upload Files**:
   - Drag 141 PDF files from `日本企業ESG報告汇总/` folder
   - **Note**: This may take 1-2 hours (2.3 GB total)
   - Zenodo supports batch upload

4. **Add Metadata**:
   ```
   Title: Sustainability Reports of 141 Japanese Listed Companies (2024)

   Description:
   This dataset contains 141 corporate sustainability reports from Japanese 
   listed companies for fiscal year 2024. The reports were analyzed in the 
   research paper "Beyond Ratings: Deconstructing ESG Disclosure Through 
   AI-Driven Action-Level Analysis" published in Cogent Business & Management.

   All reports are publicly available corporate disclosures downloaded from 
   company investor relations websites between January-March 2024. This 
   collection is provided for academic research and ESG analysis purposes.

   Related Resources:
   - Extraction code: https://github.com/[YOUR-USERNAME]/esg-action-extraction
   - Paper: [DOI when published]

   Keywords: ESG, sustainability reporting, Japan, corporate disclosure, 
   environmental reporting, social reporting, governance disclosure

   Language: Japanese (primary), English (some bilingual reports)

   License: CC BY 4.0 (original reports remain property of respective companies)

   Upload date: December 2024
   Coverage: Fiscal Year 2024 (April 2023 - March 2024)
   ```

5. **Add Authors**:
   - Name: Tong Xu
   - Affiliation: [Your University]
   - ORCID: [Your ORCID]

6. **Choose License**: **CC BY 4.0**
   - Allows reuse with attribution

7. **Related Identifiers** (add after GitHub upload):
   - "is supplemented by" → GitHub repository URL
   - "is cited by" → Paper DOI (when published)

8. **Publish**:
   - Click "Publish"
   - **Get DOI**: Will look like `10.5281/zenodo.12345678`
   - **Save this DOI!** You'll need it for the paper

### After Zenodo Upload

✅ **Already completed!** Your Zenodo DOI is: **10.5281/zenodo.17582500**

Your paper's Data Availability Statement already includes:
```
Corporate Sustainability Reports: All 141 reports are publicly accessible
via Zenodo (DOI: 10.5281/zenodo.17582500). The dataset includes PDF files
totaling 2.3 GB, with reports from Japanese listed companies for fiscal
year 2024.
```

## ✅ Final Checklist

Before submitting your paper revision:

**Completed ✅**:
- [x] Zenodo DOI obtained (10.5281/zenodo.17582500)
- [x] Contact email updated (xutongxutong@keio.jp)
- [x] Institution added (Keio University)
- [x] Paper status set to "Under Review"
- [x] Data Availability Statement updated in paper
- [x] Citation format adjusted for submitted manuscript

**Still to do**:
- [ ] Create GitHub repository (public)
- [ ] Upload all files from `github-repo/` folder
- [ ] Replace `[your-username]` with your actual GitHub username in:
  - README.md (will need to edit after upload)
  - CITATION.cff (will need to edit after upload)
- [ ] Add repository topics/keywords

## 🎯 Expected Impact

With code and data publicly available:
- ✅ **Reproducibility**: Other researchers can replicate your findings
- ✅ **Transparency**: Reviewers can inspect methodology
- ✅ **Citations**: GitHub/Zenodo provide citable DOIs
- ✅ **Community**: Others can extend your work
- ✅ **Compliance**: Meets Taylor & Francis data sharing policy

## 📧 Need Help?

If you encounter issues:
1. GitHub upload issues: https://docs.github.com/en/repositories
2. Zenodo upload issues: https://help.zenodo.org/
3. Data Availability Statement: Ask me to revise
4. Repository structure: I can reorganize files

Good luck with your submission! 🚀
