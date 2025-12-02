# Sentiment Analyser — Architecture Diagram

The diagram below shows a basic end-to-end workflow for a sentiment analysis tool. Preview this file on GitHub to see the rendered Mermaid diagram.

```mermaid
flowchart TB

    %% ================================
    %%  INGESTION (External + Internal)
    %% ================================

    DEV([Developer])

    EXT_SOURCES([External Sources<br/>(gov.uk, Ofgem, YouGov)])
    INT_SOURCES([Internal Sources<br/>(Yonder Notes/Surveys/Reports)])

    DEV --> EXT_SOURCES
    DEV --> INT_SOURCES

    EXT_SOURCES -->|Extract links via Tavily| TAVILY([Tavily API])
    TAVILY -->|Download docs| RAW_DOCS_EXT([PDF/Excel/CSV])

    INT_SOURCES -->|Upload internal files| RAW_DOCS_INT([PDF/Excel/CSV])


    %% ================================
    %%  RAW STORAGE
    %% ================================
    subgraph BLOB[**Azure Blob Storage**]
        RAW_RAW[raw/<br/>Original Documents]
        META_STORE[metadata/<br/>Document & Chunk Metadata]
        PROC_STORE[processed_chunks/<br/>Processed Text + Metadata]
    end

    RAW_DOCS_EXT --> RAW_RAW
    RAW_DOCS_INT --> RAW_RAW


    %% ================================
    %% METADATA & LINEAGE LAYER
    %% ================================
    META_LAYER([Metadata Extraction & Lineage Tracking<br/><br/>
    - Assign document_id<br/>
    - Capture source_link / file_path<br/>
    - Extract page_number & offsets<br/>
    - Prepare chunk metadata])

    RAW_RAW --> META_LAYER
    META_LAYER --> META_STORE


    %% ================================
    %% DATA PREPROCESSING
    %% ================================
    PREPROC([Data Preprocessing + Chunk Creation<br/><br/>
    - Cleaning & filtering<br/>
    - OCR if needed<br/>
    - Text normalisation<br/>
    - Create text chunks<br/>
    - Attach chunk_id + doc_id + page_number])

    META_LAYER --> PREPROC
    PREPROC -->|Store processed text + metadata| PROC_STORE


    %% ================================
    %% SENTIMENT ANALYSIS
    %% ================================
    SA([Sentiment Analysis<br/>(Azure OpenAI + Prompt Engineering)])

    PROC_STORE -->|Processed Text + Metadata| SA
    META_STORE -.->|Metadata Lookup / Traceback| SA


    %% ================================
    %% OUTPUTS
    %% ================================
    REPORT([Final Summary + Theme-wise Sentiment Output])
    SA --> REPORT
```

Notes:
- **API / UI**: Accepts text from users or batch ingestion, returns sentiment label and score.
- **Preprocessing**: Typical steps include cleaning, tokenization, lowercasing, stopword removal.
- **Feature Extraction**: Could be classic (TF-IDF) or modern embeddings (BERT, sentence-transformers).
- **Model**: Handles both training and inference. Models and artifacts should be versioned in a registry.
- **Monitoring & Feedback**: Collects metrics and user feedback to enable retraining and improvements.

Preview tip: open this file on GitHub and use the Markdown preview to render the Mermaid diagram.
