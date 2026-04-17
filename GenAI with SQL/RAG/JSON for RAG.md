### Preparing RAG Context (SQL -> JSON)

#### Define User Query

```sql
DECLARE @userQuestion NVARCHAR(1000) = 
'talk about the ESG Report of GreenSteel Ltd.';
```

#### Generate Query Embeeding

```sql
DECLARE @questionVector VECTOR(1536);

SELECT @questionVector = AI_GENERATE_EMBEDDINGS(
    @userQuestion USE MODEL rag_embedding_model
);
```

#### Retrieve Relevant Chunks

```sql
SELECT TOP 3
    CompanyName,
    ChunkText,
    VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding) AS Distance
FROM RAG.ESG_Chunks
ORDER BY Distance;
```

#### Convert to JSON

```sql
DECLARE @context NVARCHAR(MAX);

SET @context = (
    SELECT TOP 3
        CompanyName AS 'company.name',
        ChunkText AS 'company.context'
    FROM RAG.ESG_Chunks
    ORDER BY VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding)
    FOR JSON PATH
);

SELECT @context AS RAG_Context;
```

#### Improve Context Structure

```sql
DECLARE @context NVARCHAR(MAX);

SET @context = (
    SELECT TOP 3
        CompanyName AS 'company.name',
        LEFT(ChunkText, 500) AS 'company.summary',
        VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding) 
            AS 'company.relevanceScore'
    FROM RAG.ESG_Chunks
    ORDER BY VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding)
    FOR JSON PATH, ROOT('retrieved_context')
);

SELECT @context AS RAG_Context;
```
