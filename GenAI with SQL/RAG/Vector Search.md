### Vector Similarity Search in SQL

#### Generate Query Embedding

```sql
DECLARE @searchVector VECTOR(1536);

SELECT @searchVector = AI_GENERATE_EMBEDDINGS(
    'tell me something about the ESG Report of Petro Trans Limited'
    USE MODEL rag_embedding_model
);
```

#### Use Vector Distance - Exact Search

Find similar chunks:
```sql
SELECT TOP 5
    c.ChunkID,
    c.CompanyName,
    LEFT(c.ChunkText, 200) AS ChunkPreview,
    VECTOR_DISTANCE('cosine', @searchVector, c.ChunkEmbedding) AS Distance
FROM RAG.ESG_Chunks c
ORDER BY Distance;
```

Apply Threshold Filtering
```sql
SELECT 
    CompanyName,
    ChunkText,
    VECTOR_DISTANCE('cosine', @searchVector, ChunkEmbedding) AS Distance
FROM RAG.ESG_Chunks
WHERE VECTOR_DISTANCE('cosine', @searchVector, ChunkEmbedding) < 0.20
ORDER BY VECTOR_DISTANCE('cosine', @searchVector, ChunkEmbedding);
```

#### Create a Vector Index

Create a vector index with `cosine similarity` as the vector similarity metric and `DiskANN` arrangement of vectors in the search space

```sql
CREATE VECTOR INDEX idx_ChunkEmbedding
ON RAG.ESG_Chunks(ChunkEmbedding)
WITH (METRIC = 'cosine', TYPE = 'DiskANN');
```

#### Use Vector Index - Approximate Search

```sql
SELECT 
    c.CompanyName,
    c.ChunkText,
    s.distance
FROM VECTOR_SEARCH(
    TABLE = RAG.ESG_Chunks AS c,
    COLUMN = ChunkEmbedding,
    SIMILAR_TO = @searchVector,
    METRIC = 'cosine',
    TOP_N = 5
) AS s
ORDER BY s.distance;
```

#### Try Different Queries

```sql
1) 'What are the sustainability initiatives of GreenSteel Ltd?'
2) 'renewable energy'
3) 'carbon emissions reduction'
```
