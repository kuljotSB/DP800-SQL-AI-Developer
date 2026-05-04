### Creating a RAG Stored Procedure

#### Create the Stored Procedure

Add the stored procedure definition code block:
```sql
CREATE OR ALTER PROCEDURE RAG.AskESGQuestion
    @Question NVARCHAR(1000),
    @modelName NVARCHAR(100)
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @questionVector VECTOR(1536);
    DECLARE @context NVARCHAR(MAX);
    DECLARE @payload NVARCHAR(MAX);
    DECLARE @response NVARCHAR(MAX);
    DECLARE @returnValue INT;
    DECLARE @Answer NVARCHAR(MAX);
```

Add the code block to generate vector embeddings for the question:
```sql
    SELECT @questionVector = AI_GENERATE_EMBEDDINGS(
        @Question USE MODEL rag_embedding_model
    );
```

Add the code block to build up the context from the most relevant chunks:
```sql
SET @context = (
    SELECT TOP 5
        CompanyName AS 'company.name',
        LEFT(ChunkText, 500) AS 'company.summary',
        VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding) 
            AS 'company.vectorDistance'
    FROM RAG.ESG_Chunks
    ORDER BY VECTOR_DISTANCE('cosine', @questionVector, ChunkEmbedding)
    FOR JSON PATH, ROOT('retrieved_context')
);
```

Add the code block to create the JSON payload for the LLM API:
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

Add the code block to call the LLM API with the augmented payload:
```sql
    EXEC @returnValue = sp_invoke_external_rest_endpoint
        @url = N'https://<resource-name>.openai.azure.com/openai/v1/chat/completions',
        @method = 'POST',
        @payload = @payload,
        @credential = [https://<resource-name>.openai.azure.com/],
        @response = @response OUTPUT;
```

Add the code block to fecth response with graceful error handling:
```sql
    IF @returnValue = 0
    BEGIN
        SET @Answer = COALESCE(
            JSON_VALUE(@response, '$.result.choices[0].message.content'),
            JSON_VALUE(@response, '$.choices[0].message.content')
        );

        SELECT @Answer AS AssistantResponse;
    END
    ELSE
    BEGIN
        SELECT 
            @returnValue AS HttpStatus,
            JSON_VALUE(@response, '$.response.status.http.description') AS ErrorMessage;
    END
END;
```

#### Execute the Stored Procedure

```sql
EXEC RAG.AskESGQuestion 
    @Question = 'What sustainability initiatives has GreenSteel Ltd taken?',
    @modelName = 'gpt-4.1';
```

some other questions to ask:
```text
1) What is GreenSteel Ltd target year for achieving carbon neutrality?
2) By how much did FutureEnergy Corp increase its renewable generation capacity?
3) Compare the Scope 3 emissions challenges faced by GreenSteel and UrbanRetail. Also compare their company performance according to their ESG reports.
4) Which company has made the most progress on emission reductions, and why?
```
