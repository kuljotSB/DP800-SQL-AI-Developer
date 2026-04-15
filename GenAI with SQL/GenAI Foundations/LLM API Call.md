### Call LLM from SQL (Hello World Example)

#### Create Managed Identity Credentials for LLM API Access

Create a master key for credential/identity encryption purposes:
```sql
CREATE MASTER KEY 
ENCRYPTION BY PASSWORD = 'StrongPassword@123!';
```

Grant invoke permissions on the stored procedure:
```sql
GRANT EXECUTE ANY EXTERNAL ENDPOINT TO [PUBLIC];
```

Create a database scoped credential to allow SQL to call the LLM API securely using a managed identity. Replace `<resource-name>` with the name of your Azure Cognitive Services resource.

**Note**: Grant the managed identity `Cognitive Services OpenAI User` role access to the Azure Cognitive Services resource in the Azure portal.

```sql
CREATE DATABASE SCOPED CREDENTIAL [https://<resource-name>.openai.azure.com/]
    WITH IDENTITY = 'Managed Identity',
    SECRET = '{"resourceid":"https://cognitiveservices.azure.com"}';
```

Alternatively, you can even use API Key based authentication, but that is less secure and not recommended for production scenarios.
```sql
CREATE DATABASE SCOPED CREDENTIAL [https://<resource-name>.openai.azure.com/]
    WITH IDENTITY = 'HTTPEndpointHeaders',
    SECRET = '{"api-key":"<your-api-key>"}';
```

Check that the credential was created successfully:
```sql
SELECT * FROM sys.database_scoped_credentials;
```

#### Call LLM API from SQL

Create a simple prompt payload:
```sql
DECLARE @payload NVARCHAR(MAX);
SET @payload = N'{
    "messages": [
        {
            "role": "system",
            "content": "You are Batman, the Dark Knight of Gotham City. Answer questions in character."
        },
        {
            "role": "user",
            "content": "How is Gotham City doing today?"
        }
    ],
    "model": "YOUR_MODEL_NAME",
    "max_completion_tokens": 2000,
    "temperature": 0.7
}';
```

Call the LLM API using the `sp_invoke_external_rest_endpoint` stored procedure:
```sql
DECLARE @response NVARCHAR(MAX);
DECLARE @returnValue INT;

EXEC @returnValue = sp_invoke_external_rest_endpoint
    @url = N'https://<resource-name>.openai.azure.com/openai/v1/chat/completions',
    @method = 'POST',
    @payload = @payload,
    @credential = [https://<resource-name>.openai.azure.com/],
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

