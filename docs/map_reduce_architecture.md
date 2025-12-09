graph TD
    A[PDF Input] -->|Extract Text| B[Pages with Metadata]
    B -->|Chunk Text| C[Chunks with IDs<br/>file, page, chunk_id]
    
    C -->|Generate Embeddings| D[Vector Embeddings]
    D -->|Build FAISS Index| E[FAISS Index]
    
    F[Topic Query] -->|Embed Topic| G[Topic Vector]
    G -->|Semantic Search| E
    E -->|Return Top K<br/>Relevant Chunks| H[Selected Chunks<br/>TOP_K=40]
    
    H -->|MAP Step| I["map_summarize()<br/>Summarize Each Chunk<br/>~120 words per chunk"]
    I -->|Filter Empty| J[Partial Summaries]
    
    J -->|REDUCE Step| K["reduce_summarize()<br/>Merge Partials<br/>Generate Key Insights"]
    K -->|Output| L[Final Summary<br/>1. High-Level Overview<br/>2. 6-12 Key Bullets<br/>3. Conflicts/Divergences]
    
    style A fill:#e1f5ff
    style B fill:#f3e5f5
    style C fill:#f3e5f5
    style D fill:#fff3e0
    style E fill:#fff3e0
    style F fill:#e8f5e9
    style H fill:#ffe0b2
    style I fill:#ffccbc
    style J fill:#ffccbc
    style K fill:#c8e6c9
    style L fill:#a5d6a7
