### Encapsulate LLM Logic in Stored Procedures

#### Encapsulate in Stored Procedure for Reusability

```sql
CREATE OR ALTER PROCEDURE RAG.sp_AskLLM
    @question NVARCHAR(MAX)
    @systemPrompt NVARCHAR(MAX) = N'You are a helpful assistant that answers user queries'
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @payload NVARCHAR(MAX);
    DECLARE @response NVARCHAR(MAX);
    DECLARE @returnValue INT;

    -- Step 1: Build payload dynamically
    SET @payload = N'{
      "messages": [
        {
          "role": "system",
          "content": "' + REPLACE(@systemPrompt, '"', '\"') + N'"
        },
        {
          "role": "user",
          "content": "' + REPLACE(@question, '"', '\"') + N'"
        }
      ],
      "max_tokens": 100
    }';

    -- Step 2: Call LLM
    EXEC @returnValue = sp_invoke_external_rest_endpoint
        @url = N'https://<endpoint>.openai.azure.com/openai/deployments/<chat-model>/chat/completions?api-version=2024-02-15-preview',
        @method = 'POST',
        @payload = @payload,
        @credential = [https://<endpoint>.cognitiveservices.azure.com/],
        @response = @response OUTPUT;

    -- Step 3: Return result
    IF @returnValue = 0
    BEGIN
        SELECT 
            JSON_VALUE(@response, '$.result.choices[0].message.content') 
            AS Answer;
    END
    ELSE
    BEGIN
        SELECT 
            @returnValue AS HttpStatus,
            JSON_VALUE(@response, '$.response.status.http.description') 
            AS ErrorMessage;
    END
END;
```

#### Try different queries

```sql
EXEC RAG.sp_AskLLM 
    @question = N'What is the capital of France?',
    @systemPrompt = N'You are a geography expert.';
```

```sql
EXEC RAG.sp_AskLLM 
    @question = N'Help me plan a trip to India',
    @systemPrompt = N'You are a travel expert';
```

```sql
EXEC RAG.sp_AskLLM 
    @question = N'Explain the concept of ESG reporting and scope 1, 2, and 3 emissions',
    @systemPrompt = N'You are an ESG consultant providing detailed explanations.';
```