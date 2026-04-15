### Chunking + Embedding Storage (RAG Foundation)

#### Goal

From your existing table: RAG.ESG_TextData

We will:

1) Chunk SustainabilityReport
2) Generate embeddings
3) Store results in a new column

#### Create Chunk Table

```sql
CREATE TABLE RAG.ESG_Chunks (
    ChunkID INT IDENTITY PRIMARY KEY,
    
    RecordID INT NOT NULL,
    CompanyName NVARCHAR(100),
    
    ChunkText NVARCHAR(MAX),
    ChunkEmbedding VECTOR(1536), -- Adjust size based on embedding model output

    CONSTRAINT FK_ESG_Chunks_Record
        FOREIGN KEY (RecordID)
        REFERENCES RAG.ESG_TextData(RecordID)
        ON DELETE CASCADE
);
```

#### Insert all chunks

```sql
INSERT INTO RAG.ESG_Chunks (RecordID, CompanyName, ChunkText)
SELECT 
    t.RecordID,
    t.CompanyName,
    c.chunk
FROM RAG.ESG_TextData t
CROSS APPLY 
    AI_GENERATE_CHUNKS(
        SOURCE = t.SustainabilityReport,
        CHUNK_TYPE = FIXED,
        CHUNK_SIZE = 500,
        OVERLAP = 50
    ) AS c;
```

#### Generate Embeddings for Chunks

```sql
UPDATE RAG.ESG_Chunks
SET ChunkEmbedding = AI_GENERATE_EMBEDDINGS(
    ChunkText USE MODEL rag_embedding_model
);
```

#### View the results

```sql
SELECT * FROM RAG.ESG_Chunks;
```

Get chunk count per record:
```sql
SELECT 
    c.RecordID,
    t.CompanyName,
    COUNT(*) AS ChunkCount
FROM RAG.ESG_Chunks AS c
JOIN RAG.ESG_TextData AS t
    ON c.RecordID = t.RecordID
GROUP BY c.RecordID, t.CompanyName;
```

Combine with original text:
```sql
SELECT 
    t.RecordID,
    t.CompanyName,
    t.SustainabilityReport,
    c.ChunkID,
    c.ChunkText,
    c.ChunkEmbedding,
    t.ReviewText,
    t.Sentiment,
    t.Summary
FROM RAG.ESG_TextData t
JOIN RAG.ESG_Chunks c
    ON t.RecordID = c.RecordID;
```
