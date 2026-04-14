### Call LLM from SQL (Hello World Example)

#### Create Managed Identity Credentials for LLM API Access

Create a database scoped credential to allow SQL to call the LLM API securely using a managed identity. Replace `<resource-name>` with the name of your Azure Cognitive Services resource.

**Note**: Grant the managed identity `Cognitive Services OpenAI User` role access to the Azure Cognitive Services resource in the Azure portal.

```sql
CREATE DATABASE SCOPED CREDENTIAL [https://<resource-name>.cognitiveservices.azure.com/]
    WITH IDENTITY = 'Managed Identity',
    SECRET = '{"resourceid":"https://cognitiveservices.azure.com"}';
```

Alternatively, you can even use API Key based authentication, but that is less secure and not recommended for production scenarios.
```sql
CREATE DATABASE SCOPED CREDENTIAL [https://<resource-name>.cognitiveservices.azure.com/]
    WITH IDENTITY = 'HTTPEndpointHeaders',
    SECRET = '{"api-key":"<your-api-key>"}';
```

#### Call LLM API from SQL

Create a simple prompt payload:
```sql
DECLARE @payload NVARCHAR(MAX);

SET @payload = N'{
  "messages": [
  {
      "role": "system",
      "content": "You are Batman - The Dark Knight of Gotham City. Answer questions in a brooding, mysterious manner."
  },
  {
      "role": "user",
      "content": "How is Gotham city doing today?"
  }
  ],
  "temperature": 0.7,
}';
```

Call the LLM API using the `sp_invoke_external_rest_endpoint` stored procedure:
```sql
DECLARE @response NVARCHAR(MAX);
DECLARE @returnValue INT;

EXEC @returnValue = sp_invoke_external_rest_endpoint
    @url = N'https://<endpoint>.openai.azure.com/openai/deployments/<chat-model>/chat/completions?api-version=2024-02-15-preview',
    @method = 'POST',
    @payload = @payload,
    @credential = [https://<endpoint>.cognitiveservices.azure.com/],
    @response = @response OUTPUT;
```

#### See Raw Response

```sql
SELECT @response AS RawResponse;
```

#### Parse Response (Extract Generated Text)

```sql
SELECT 
    JSON_VALUE(@response, '$.result.choices[0].message.content') 
    AS Answer;
```

#### Handle Errors

```sql
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
        AS Error;
END;
```

