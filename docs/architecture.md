# Sentiment Analyser — Architecture Diagram

The diagram below shows a basic end-to-end workflow for a sentiment analysis tool. Preview this file on GitHub to see the rendered Mermaid diagram.

```mermaid
flowchart TB
 
    %% --- DEVELOPER INPUT ---
    Developer([Developer])
 
    %% --- EXTERNAL DATA EXTRACTION ---
    ExternalExtraction([Extracts doc links from external sources using Tavily])
    Developer --> ExternalExtraction
 
    ExternalDocs([Surveys / Reports from External Sources])
    ExternalExtraction --> ExternalDocs
 
    %% --- INTERNAL DATA COLLECTION ---
    InternalCollection([Collects data from internal sources - Notes / Surveys / Reports])
    Developer --> InternalCollection
 
    InternalDocs([Surveys / Reports from Internal Sources])
    InternalCollection --> InternalDocs
 
    %% --- STORAGE OF RAW DATA ---
    StoreRaw([Store documents from links and internal sources in Azure Blob Storage])
    ExternalDocs --> StoreRaw
    InternalDocs --> StoreRaw
 
    %% --- DATA PREPROCESSING ---
    Preprocess([Data Preprocessing - Cleaning, Filtering, Collation, Text Extraction])
    StoreRaw --> Preprocess
 
    StoreProcessed([Store processed text file in Azure Blob Storage])
    Preprocess --> StoreProcessed
 
    %% --- SENTIMENT ANALYSIS ---
    Sentiment([Sentiment Analysis using Azure OpenAI + Prompt Engineering])
    Preprocess --> Sentiment
 
    Reports([External & Internal Theme-wise Summary Reports])
    Sentiment --> Reports
 
    StoreReports([Store generated reports in Azure Blob Storage])
    Reports --> StoreReports
 
    %% --- REPORT VALIDATION ---
    Validation([Report Validation using LangSmith])
    Reports --> Validation
 
    ValidationSummary([Validation Summary])
    Validation --> ValidationSummary
 
    %% --- REPORT COMPARISON ---
    Comparison([Report Comparison using Azure OpenAI + Prompt Engineering])
    Validation --> Comparison
 
    Consolidated([Consolidated Report Comparing Internal & External Sentiments])
    Comparison --> Consolidated
 
    %% --- PARALLEL PROMPT TESTING ---
    ParallelTesting([Parallel Prompt Testing using Copilot and GridGPT])
    Sentiment -.-> ParallelTesting
    Comparison -.-> ParallelTesting
```

Notes:
- **API / UI**: Accepts text from users or batch ingestion, returns sentiment label and score.
- **Preprocessing**: Typical steps include cleaning, tokenization, lowercasing, stopword removal.
- **Feature Extraction**: Could be classic (TF-IDF) or modern embeddings (BERT, sentence-transformers).
- **Model**: Handles both training and inference. Models and artifacts should be versioned in a registry.
- **Monitoring & Feedback**: Collects metrics and user feedback to enable retraining and improvements.

Preview tip: open this file on GitHub and use the Markdown preview to render the Mermaid diagram.
