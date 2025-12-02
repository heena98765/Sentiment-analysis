# Sentiment Analyser — Architecture Diagram

The diagram below shows a basic end-to-end workflow for a sentiment analysis tool. Preview this file on GitHub to see the rendered Mermaid diagram.

```mermaid
flowchart TB

    %% ================
    %% INGESTION
    %% ================

    DEV([Developer])

    EXT_SOURCES([External Sources: gov.uk, Ofgem, YouGov])
    INT_SOURCES([Internal Sources: Yonder Notes/Surveys/Reports])

    DEV --> EXT_SOURCES
    DEV --> INT_SOURCES

    EXT_SOURCES -->|Extract links via Tavily| TAVILY([Tavily API])
    TAVILY -->|Download documents| RAW_DOCS_EXT([External PDF/Excel/CSV])

    INT_SOURCES -->|Upload internal files| RAW_DOCS_INT([Internal PDF/Excel/CSV])


    %% ================
    %% AZURE STORAGE
    %% ================
    subgraph BLOB[Azure Blob Storage]
        RAW_RAW[raw - Original Documents]
        META_STORE[metadata - Document and Chunk Metadata]
        PROC_STORE[processed_chunks - Processed Text with Metadata]
    end

    RAW_DOCS_EXT --> RAW_RAW
    RAW_DOCS_INT --> RAW_RAW


    %% ============================
    %% METADATA & LINEAGE LAYER
    %% ============================
    META_LAYER([Metadata Extraction and Lineage Tracking
    Assign document_id
    Capture source link or file path
    Extract page numbers and offsets
    Generate chunk metadata])

    RAW_RAW --> META_LAYER
    META_LAYER --> META_STORE


    %% ============================
    %% DATA PREPROCESSING
    %% ============================
    PREPROC([Data Preprocessing and Chunk Creation
    Cleaning and filtering
    Create text chunks with metadata])

    META_LAYER --> PREPROC
    PREPROC -->|Store processed text + metadata| PROC_STORE


    %% ============================
    %% SENTIMENT ANALYSIS
    %% ============================
    SA([Sentiment Analysis using Azure OpenAI])

    PROC_STORE -->|Processed text + metadata| SA
    META_STORE -.->|Metadata lookup for traceback| SA


    %% ============================
    %% OUTPUT
    %% ============================
    REPORT([Final Summary and Theme-wise Sentiment Analysis])
    SA --> REPORT
```

Notes:
- **API / UI**: Accepts text from users or batch ingestion, returns sentiment label and score.
- **Preprocessing**: Typical steps include cleaning, tokenization, lowercasing, stopword removal.
- **Feature Extraction**: Could be classic (TF-IDF) or modern embeddings (BERT, sentence-transformers).
- **Model**: Handles both training and inference. Models and artifacts should be versioned in a registry.
- **Monitoring & Feedback**: Collects metrics and user feedback to enable retraining and improvements.

Preview tip: open this file on GitHub and use the Markdown preview to render the Mermaid diagram.
