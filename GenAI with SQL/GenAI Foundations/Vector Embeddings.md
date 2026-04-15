### Generate Vector Embeddings in SQL

#### Scenario

You are a SQL AI Developer at CarbonOps. You want to generate vector embeddings for ESG text data using LLMs.

#### Create External Model (Vector Embedding Engine)

```sql
CREATE EXTERNAL MODEL rag_embedding_model
WITH (
    LOCATION = 'https://<your-resource-name>.openai.azure.com/openai/deployments/<your-deployment-name>/embeddings?api-version=2024-02-15-preview',
    API_FORMAT = 'Azure OpenAI',
    MODEL_TYPE = EMBEDDINGS,
    MODEL = '<embedding-model-deployment-type>',
    CREDENTIAL = [https://<your-resource-name>.openai.azure.com/],
);
```

#### Generate Embeddings for ESG Text Data

Generate for Reviews:
```sql
SELECT 
    CompanyName,
    ReviewText,
    AI_GENERATE_EMBEDDINGS(
        ReviewText USE MODEL rag_embedding_model
    ) AS ReviewEmbedding
FROM RAG.ESG_TextData;
```

Generate for Sustainability Reports:
```sql
SELECT 
    CompanyName,
    SustainabilityReport,
    AI_GENERATE_EMBEDDINGS(
        SustainabilityReport USE MODEL rag_embedding_model
    ) AS ReportEmbedding
FROM RAG.ESG_TextData;
```

Generate for both at once:
```sql
SELECT 
    CompanyName,
    ReviewText,
    SustainabilityReport,
    AI_GENERATE_EMBEDDINGS(
        ReviewText USE MODEL rag_embedding_model
    ) AS ReviewEmbedding,
    AI_GENERATE_EMBEDDINGS(
        SustainabilityReport USE MODEL rag_embedding_model
    ) AS ReportEmbedding
FROM RAG.ESG_TextData;
```
