# Sentiment Analyser — Architecture Diagram

The diagram below shows a basic end-to-end workflow for a sentiment analysis tool. Preview this file on GitHub to see the rendered Mermaid diagram.

```mermaid
flowchart TB

    %% ============================
    %% ACTORS
    %% ============================
    DEV([Developer])

    %% ============================
    %% INPUT SOURCES
    %% ============================
    EXT_SOURCES([External Sources: gov.uk, Ofgem, YouGov])
    INT_SOURCES([Internal Sources: Yonder Notes, Surveys, Reports])

    DEV --> EXT_SOURCES
    DEV --> INT_SOURCES

    EXT_SOURCES -->|Extract links via Tavily| TAVILY([Tavily API])
    TAVILY -->|Download documents| EXT_DOCS([External Reports: PDF, Excel, CSV])

    INT_SOURCES -->|Collect internal documents| INT_DOCS([Internal Reports: PDF, Excel, CSV])


    %% ============================
    %% RAW DOCUMENT STORAGE
    %% ============================
    subgraph BLOB1[Azure Blob Storage]
        RAW_STORE[Store original documents]
    end

    EXT_DOCS --> RAW_STORE
    INT_DOCS --> RAW_STORE


    %% ============================
    %% DATA PREPROCESSING
    %% ============================
    PREPROC([Data Preprocessing
    - Cleaning and filtering
    - Collation
    - Extraction of text
    - Creation of metadata])

    RAW_STORE --> PREPROC

    PREPROC -->|Processed text + metadata| PROC_OUT([Processed Text and Metadata])


    %% ============================
    %% STORE PROCESSED OUTPUT
    %% ============================
    subgraph BLOB2[Azure Blob Storage]
        FINAL_PROC_STORE[Store processed text and metadata]
    end

    PROC_OUT --> FINAL_PROC_STORE


    %% ============================
    %% SENTIMENT ANALYSIS
    %% ============================
    SA([Sentiment Analysis
    Azure OpenAI + Prompt Engineering
    Generate theme-wise sentiment
    with citations])

    PROC_OUT --> SA
    SA --> PROMPT_USED1([Prompt Used])


    %% ============================
    %% STORE GENERATED REPORTS
    %% ============================
    subgraph BLOB3[Azure Blob Storage]
        REPORT_STORE[Store generated reports]
    end

    SA -->|External & Internal theme-wise summary reports| REPORT_STORE


    %% ============================
    %% REPORT VALIDATION
    %% ============================
    VALID([Report Validation
    LangSmith (LLM as judge)
    Validate generated reports])

    REPORT_STORE --> VALID
    VALID --> VAL_SUMMARY([Validation Summary])


    %% ============================
    %% PARALLEL PROMPT TESTING
    %% ============================
    GRIDGPT([Testing prompts in parallel
    using Copilot and GridGPT])

    PROMPT_USED1 --> GRIDGPT
    GRIDGPT --> VAL_SUMMARY


    %% ============================
    %% REPORT COMPARISON
    %% ============================
    COMPARE([Report Comparison
    Azure OpenAI + Prompt Engineering
    Compare Internal vs External Reports])

    VALID --> COMPARE
    COMPARE --> PROMPT_USED2([Prompt Used])


    %% ============================
    %% FINAL OUTPUT
    %% ============================
    FINAL_OUTPUT([Consolidated Report:
    Internal vs External Sentiment (PDF)])

    COMPARE --> FINAL_OUTPUT

```

Notes:
- **API / UI**: Accepts text from users or batch ingestion, returns sentiment label and score.
- **Preprocessing**: Typical steps include cleaning, tokenization, lowercasing, stopword removal.
- **Feature Extraction**: Could be classic (TF-IDF) or modern embeddings (BERT, sentence-transformers).
- **Model**: Handles both training and inference. Models and artifacts should be versioned in a registry.
- **Monitoring & Feedback**: Collects metrics and user feedback to enable retraining and improvements.

Preview tip: open this file on GitHub and use the Markdown preview to render the Mermaid diagram.
