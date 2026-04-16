### Hybrid Search - Full-Text, Vector and RRF (Reciprocal Rank Fusion)

#### Create a Full-Text Index on ChunkText

```sql
CREATE FULLTEXT INDEX ON RAG.ESG_Chunks
(
    ChunkText LANGUAGE 1033
)
KEY INDEX <your-primary-key-index-name>
ON ESG_FT_Catalog;
```

#### Define Search Input Parameters

```sql
DECLARE @searchText NVARCHAR(1000) = 'reducing carbon emissions';
DECLARE @searchVector VECTOR(1536);
DECLARE @topN INT = 20;
DECLARE @rrfK INT = 60;
```

#### Generate Query Embedding

```sql
SELECT @searchVector = AI_GENERATE_EMBEDDINGS(
    @searchText USE MODEL rag_embedding_model
);
```

#### Hybrid Search Query

```sql
WITH keyword_search AS (
    SELECT TOP(@topN)
        c.ChunkID,
        RANK() OVER (ORDER BY ftt.[RANK] DESC) AS keyword_rank
    FROM RAG.ESG_Chunks c
    INNER JOIN FREETEXTTABLE(
        RAG.ESG_Chunks,
        ChunkText,
        @searchText
    ) ftt
        ON c.ChunkID = ftt.[KEY]
),

vector_search AS (
    SELECT TOP(@topN)
        ChunkID,
        RANK() OVER (
            ORDER BY VECTOR_DISTANCE(
                'cosine',
                @searchVector,
                ChunkEmbedding
            )
        ) AS vector_rank
    FROM RAG.ESG_Chunks
),

combined AS (
    SELECT 
        COALESCE(ks.ChunkID, vs.ChunkID) AS ChunkID,
        ks.keyword_rank,
        vs.vector_rank,

        -- RRF Formula
        COALESCE(1.0 / (@rrfK + ks.keyword_rank), 0.0) +
        COALESCE(1.0 / (@rrfK + vs.vector_rank), 0.0) 
        AS rrf_score

    FROM keyword_search ks
    FULL OUTER JOIN vector_search vs
        ON ks.ChunkID = vs.ChunkID
)

SELECT 
    c.CompanyName,
    LEFT(c.ChunkText, 200) AS ChunkPreview,
    combined.keyword_rank,
    combined.vector_rank,
    combined.rrf_score
FROM combined
JOIN RAG.ESG_Chunks c
    ON combined.ChunkID = c.ChunkID
ORDER BY combined.rrf_score DESC;
```