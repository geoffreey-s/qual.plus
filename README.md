# qual.plus
Scirpts for bibliometric analysis of qualitative research x AI
Qualitative-AI: A Hybrid Bibliometric Pipeline for Mapping AI in Qualitative Research
![Kerko](https://img.shields.io/badge/bibliography-live-FF6B35)
![Kumu](https://img.shields.io/badge/network_map-live-39ff14)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
This repository contains the complete replication pipeline for a scoping review of how artificial intelligence — specifically large language models, generative AI, and transformer-based models — is reshaping qualitative research methods.
The pipeline combines traditional bibliometric methods with LLM-assisted metadata extraction, unsupervised concept induction, deductive codebook scoring, and Kumu/Zotero synchronization to analyze a corpus of 497 papers (2010–2026).
Live outputs:
Interactive Bibliography (Kerko) — faceted search across 6 dimensions
Co-citation Network Map (Kumu) — interactive network visualization
---
Pipeline Overview
```text
Zotero Library ──► QUAL-AI.bib ──► harvest_and_enrich.py ──► JSON/item
       │                                  │
       │                                  ├──► build_bibliometrix_master.py
       │                                  │         │
       │                                  │         ▼
       │                                  │    normalize_cr.py
       │                                  │         │
       │                                  │         ▼
       │                                  │    convert_to_wos.py
       │                                  │         │
       │                                  │         ▼
       │                                  │    fix_wos_c1.py
       │                                  │         │
       │                                  │         ▼
       │                                  │  bibliometrix_master_wos_fixed.txt
       │                                  │         │
       │                                  │         ▼
       │                                  │  R / bibliometrix
       │                                  │         │
       │                                  │         ▼
       │                                  └──► build_kumu_network.py
       │
       ├──► extract_keywords_from_pdfs.py
       ├──► classify_study_type.py / map_venue_disciplines.py
       ├──► Zotero-Citation-Update.py
       ├──► fetch-abstracts.py
       │
       ▼
screen_corpus.py ◄── Inclusion / Exclusion Criteria v2
       │
       ▼
clean_kumu_keywords.py ──► Normalized Kumu dataset
       │
       ├──► run_lloom.py ──► claude_synthesis.py ──► score_corpus.py
       │
       ▼
pin_zotero_citekeys.py ──► sync_kumu_bib_zotero.py
       │                           │
       │                           ├──► Zotero (titles, tags, DOI, abstracts)
       │                           ├──► QUAL-AI.cleaned.bib
       │                           ├──► Kumu_v11_synced.xlsx
       │                           └──► Kerko (PythonAnywhere)
```
---
Replication Pipeline
Step	Research Phase	Script	LLM / API	Platform	Description
1	Corpus Construction	—	—	Zotero	Literature search across 8 databases. Import to Zotero group library. Attach PDFs. Export as `QUAL-AI.bib` via Better BibTeX.
2	Bibliometric Harvesting	`harvest_and_enrich.py`	OpenAlex, Semantic Scholar, Crossref, Unpaywall	Local	Two-phase pipeline. Phase 1 harvests cited references with an API cascade and writes one JSON per corpus item. Phase 2 enriches the JSONs with concepts/keywords, missing abstracts, OA status, citation counts, and merge passes for bibliometric downstream use.
3a	Metadata Enrichment	`extract_keywords_from_pdfs.py`	Claude Sonnet + Zotero API	Local	Downloads PDFs from Zotero cloud and extracts author-provided keywords from the first pages. Flags items with no keyword section and writes the recovered terms back into the Kumu workbook.
3b	Metadata Enrichment	`classify_study_type.py`	Claude Sonnet	Local	Classifies each corpus item as Empirical/Applied, Methodological/Technical, or Conceptual/Critical using title, abstract, and keyword metadata.
3c	Metadata Enrichment	`map_venue_disciplines.py`	—	Local	Maps publication venues to ten discipline categories with deterministic lookups and keyword-based fuzzy rules.
3d	Metadata Enrichment	`Zotero-Citation-Update.py`	OpenAlex, Semantic Scholar, Crossref, Zotero Web API	Local	Zotero maintenance utility. Repairs missing DOIs by title lookup and stores citation counts in the Zotero `Extra` field as `Citation Count: N (Source)`.
3e	Metadata Enrichment	`fetch-abstracts.py`	Semantic Scholar, OpenAlex, Elsevier, Crossref, Zotero Web API	Local	Fallback abstract completion tool. Reads a BibTeX file, finds entries missing abstracts, queries multiple APIs, and writes a `_with_abstracts.bib` output plus a report and cache.
4a	Bibliometrix Assembly	`build_bibliometrix_master.py`	—	Local	Reads `QUAL-AI.bib` and harvested JSON reference files, matches them by normalized DOI, and injects CR lists into a bibliometrix-ready BibTeX master with brace-depth-aware parsing and structural validation.
4b	Bibliometrix Assembly	`normalize_cr.py`	—	Local	Cleans CR field values into plain-text, semicolon-delimited strings that bibliometrix can safely import. Removes structural artifacts while preserving reference content.
4c	Bibliometrix Assembly	`convert_to_wos.py`	—	Local	Converts the cleaned BibTeX into Web of Science plaintext for bibliometrix import, preserving the Better BibTeX citekey in the `UT` field.
4d	Bibliometrix Assembly	`fix_wos_c1.py`	—	Local	Inserts placeholder `C1` and `RP` fields into every WoS plaintext record so bibliometrix does not crash when those columns are otherwise absent in all records.
5	Corpus Screening	`screen_corpus.py`	Claude Sonnet	Local	AI-assisted screening against seven inclusion and six exclusion criteria with per-item decisions, confidence, and reasoning.
6	Keyword Normalization	`clean_kumu_keywords.py`	—	Local	Deterministic merge rules that expand abbreviations, collapse variants, and remove off-topic or structural tags before concept induction and scoring.
7a	Concept Induction	`run_lloom.py`	GPT-4o-mini	Local	LLooM-based unsupervised concept induction on the full corpus with per-document concept scores.
7b	Concept Induction	`claude_synthesis.py`	Claude Sonnet	Local	Narrative synthesis for each LLooM concept cluster using a critical second-coder prompt strategy.
8	Deductive Codebook Scoring	`score_corpus.py`	GPT-4o-mini	Local	Scores the corpus on a 26-point binary codebook spanning methodology, dynamics, limitations, ethics, and prompting.
9a	Synchronization	`pin_zotero_citekeys.py`	Zotero Web API	Local	One-time setup script that pins Better BibTeX citation keys into the Zotero `Extra` field for stable matching.
9b	Synchronization	`sync_kumu_bib_zotero.py`	Zotero Web API	Local	Kumu-authoritative three-way sync. Cleans BibTeX, matches against pinned citation keys, updates Zotero titles/tags/DOIs/abstracts, and writes synchronized Kumu and BibTeX outputs.
9c	Synchronization	`setup_facets.py`	Zotero Web API	Local	Creates Zotero collection hierarchies used by Kerko faceted browsing and emits matching config blocks.
10	Cited Reference Extraction	`extract_cr_from_pdfs.py`	Claude Sonnet + Zotero API	Local	For items still missing cited references after harvesting, downloads PDFs and parses trailing reference lists into JSON aligned with the harvest schema.
11	Network Construction	`build_kumu_network.py`	—	Local	Builds Kumu-ready co-citation network files from `bibliometrix_master_wos_fixed.txt` and `QUAL-AI.bib`, preserving corpus-item citekeys and hashed external references.
12	Bibliometric Analysis	R / bibliometrix	—	Local (R)	Imports the fixed WoS plaintext into `bibliometrix`, clears placeholder C1/RP values, and runs co-citation, bibliographic coupling, thematic evolution, and strategic diagram analyses.
13	Web Deployment	Kerko config	—	PythonAnywhere	KerkoApp deployment with custom CSS, six facets, WSGI configuration, and PythonAnywhere logging fixes.
---
Repository Structure
```text
Qualitative-AI/
├── README.md
├── LICENSE
├── data/
│   ├── QUAL-AI.bib
│   ├── Kumu_v11.xlsx
│   ├── inclusion_exclusion_criteria.xlsx
│   └── lloom/
│       ├── lloom_concepts_unified.csv
│       ├── lloom_scores_raw_unified.csv
│       └── thematic_syntheses/
├── scripts/
│   ├── 01-harvest/
│   │   ├── harvest_and_enrich.py
│   │   └── README.md
│   ├── 02-metadata/
│   │   ├── extract_keywords_from_pdfs.py
│   │   ├── classify_study_type.py
│   │   ├── map_venue_disciplines.py
│   │   ├── Zotero-Citation-Update.py
│   │   ├── fetch-abstracts.py
│   │   └── README.md
│   ├── 03-bibliometrix/
│   │   ├── build_bibliometrix_master.py
│   │   ├── normalize_cr.py
│   │   ├── convert_to_wos.py
│   │   ├── fix_wos_c1.py
│   │   └── README.md
│   ├── 04-screening/
│   │   ├── screen_corpus.py
│   │   └── README.md
│   ├── 05-normalization/
│   │   ├── clean_kumu_keywords.py
│   │   └── README.md
│   ├── 06-concept-induction/
│   │   ├── run_lloom.py
│   │   ├── claude_synthesis.py
│   │   └── README.md
│   ├── 08-codebook/
│   │   ├── score_corpus.py
│   │   └── README.md
│   ├── 09-sync/
│   │   ├── pin_zotero_citekeys.py
│   │   ├── sync_kumu_bib_zotero.py
│   │   ├── setup_facets.py
│   │   └── README.md
│   └── 10-network/
│       ├── extract_cr_from_pdfs.py
│       ├── build_kumu_network.py
│       └── README.md
├── deployment/
│   └── kerko/
│       ├── config.toml
│       ├── wsgi.py
│       ├── kerkoapp/
│       │   ├── __init__.py
│       │   └── logging.py
│       └── README.md
└── docs/
    ├── search-strategy.md
    ├── codebook.md
    └── VERSION_AND_STATUS.md
```
---
Corpus Statistics
Metric	Value
Total items	497
Date range	2010–2026
With abstracts	497 (100%)
With author keywords	425 (85%)
With discipline classification	467 (94%)
With study type classification	467 (94%)
With 26-point codebook scores	467 (94%)
With LLooM concept scores	497 (100%)
LLooM concepts induced	14
Codebook domains	5 (26 binary criteria)
Kerko facets	6
Discipline categories	10
Inclusion types	7
---
Technology Stack
Component	Technology	Role
Reference management	Zotero + Better BibTeX	Corpus construction, PDF storage, metadata
Bibliometric APIs	OpenAlex, Semantic Scholar, Crossref, Unpaywall	Cited references, citation counts, keywords, OA status
LLM (structured extraction)	Claude Sonnet 4 (Anthropic)	Keyword extraction, study type classification, screening, CR extraction, thematic synthesis
LLM (concept induction)	GPT-4o-mini (OpenAI) via LLooM	Unsupervised concept clustering, codebook scoring
Network visualization	Kumu	Interactive co-citation and bibliographic coupling maps
Web bibliography	KerkoApp (Flask) on PythonAnywhere	Public faceted search interface
Data authority	Kumu Excel → Zotero → BibTeX	Three-way synchronization pipeline
---
Replication Requirements
```bash
pip install pyzotero anthropic openai openpyxl pypdf bibtexparser requests tqdm text_lloom python-dotenv
```
API keys needed:
Anthropic API key — for keyword extraction, study type classification, screening, CR extraction, and thematic synthesis
OpenAI API key — for LLooM concept induction and codebook scoring
Zotero API key — for citekey pinning, sync, and Zotero metadata maintenance
---
