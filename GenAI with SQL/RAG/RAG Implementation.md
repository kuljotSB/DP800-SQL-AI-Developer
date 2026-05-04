### Generate RAG Responses (End-to-End in SQL)

#### Declare Variables

```sql
DECLARE @Question NVARCHAR(1000) = 
'talk about the ESG Report of GreenSteel Ltd.';

DECLARE @modelName NVARCHAR(100) = 'YOUR_MODEL_NAME_HERE';
DECLARE @Answer NVARCHAR(MAX);
DECLARE @questionVector VECTOR(1536);
DECLARE @context NVARCHAR(MAX);
DECLARE @payload NVARCHAR(MAX);
DECLARE @response NVARCHAR(MAX);
DECLARE @returnValue INT;
```

#### Convert Question to Vector Embedding

```sql
SELECT @questionVector = AI_GENERATE_EMBEDDINGS(
    @Question USE MODEL rag_embedding_model
);
```

#### Retrieve Relevant Context Chunks

```sql
SET @context = (
    SELECT TOP 3
        CompanyName AS 'company.name',
        LEFT(ChunkText, 500) AS 'company.summary',
        VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding) 
            AS 'company.vectorDistance'
    FROM RAG.ESG_Chunks
    ORDER BY VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding)
    FOR JSON PATH, ROOT('retrieved_context')
);
```


#### Build Prompt Payload for LLM API

```sql
SET @payload = JSON_OBJECT(
    'messages': JSON_ARRAY(
        JSON_OBJECT(
            'role': 'system',
            'content': 
'You are an ESG assistant. Answer ONLY using the provided context. Do not hallucinate.'
        ),
        JSON_OBJECT(
            'role': 'user',
            'content': 
'Context: ' + @context + 
' Question: ' + @Question
        )
    ),
    'model': @modelName,
    'temperature': 0.3
);
```

#### Call LLM API

```sql
EXEC @returnValue = sp_invoke_external_rest_endpoint
    @url = N'https://<resource-name>.openai.azure.com/openai/v1/chat/completions',
    @method = 'POST',
    @payload = @payload,
    @credential = [https://<resource-name>.openai.azure.com/],
    @response = @response OUTPUT;
```

#### Extract Answer from LLM Response

```sql
IF @returnValue = 0
BEGIN
    SET @Answer = JSON_VALUE(
        @response,
        '$.result.choices[0].message.content'
    );

    SELECT @Answer AS AssistantResponse;
END
```

#### Try other queries

some other questions to ask:
```text
1) What is GreenSteel Ltd target year for achieving carbon neutrality?
2) By how much did FutureEnergy Corp increase its renewable generation capacity?
3) Compare the Scope 3 emissions challenges faced by GreenSteel and UrbanRetail. Also compare their company performance according to their ESG reports.
4) Which company has made the most progress on emission reductions, and why?
```
