### ESG Sentiment & Summarization using Stored Procedures

#### Scenario

At CarbonOps, you want to analyze ESG text using LLMs.

You have:

1) Reviews → sentiment
2) Sustainability reports → summaries

Goal: Build reusable stored procedures for:

1) Sentiment analysis
2) Summarization

#### Add output columns

Add Sentiment and Summary columns:
```sql
ALTER TABLE RAG.ESG_TextData
ADD Sentiment NVARCHAR(50),
    Summary NVARCHAR(MAX);
```

#### Create a Stored Procedure for Sentiment Analysis

```sql
CREATE OR ALTER PROCEDURE RAG.sp_AnalyzeSentiment
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @id INT, @review NVARCHAR(MAX);
    DECLARE @payload NVARCHAR(MAX);
    DECLARE @response NVARCHAR(MAX);
    DECLARE @returnValue INT;
    DECLARE @sentiment NVARCHAR(50);

    DECLARE review_cursor CURSOR FOR
    SELECT RecordID, ReviewText
    FROM RAG.ESG_TextData;

    OPEN review_cursor;

    FETCH NEXT FROM review_cursor INTO @id, @review;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- Build payload
        SET @payload = N'{
          "messages": [
            {
              "role": "system",
              "content": "Classify sentiment as Positive, Negative, or Neutral."
            },
            {
              "role": "user",
              "content": "' + REPLACE(@review, '"', '\"') + N'"
            }
          ],
          "max_tokens": 20
        }';

        -- Call LLM
        EXEC @returnValue = sp_invoke_external_rest_endpoint
            @url = N'https://<endpoint>.openai.azure.com/openai/deployments/<chat-model>/chat/completions?api-version=2024-02-15-preview',
            @method = 'POST',
            @payload = @payload,
            @credential = [https://<endpoint>.cognitiveservices.azure.com/],
            @response = @response OUTPUT;

        IF @returnValue = 0
        BEGIN
            SET @sentiment = JSON_VALUE(@response, '$.result.choices[0].message.content');

            UPDATE RAG.ESG_TextData
            SET Sentiment = @sentiment
            WHERE RecordID = @id;
        END

        FETCH NEXT FROM review_cursor INTO @id, @review;
    END

    CLOSE review_cursor;
    DEALLOCATE review_cursor;
END;
```

#### Run Sentiment Analysis

```sql
EXEC RAG.sp_AnalyzeSentiment;
```

#### Create a Stored Procedure for Summarization

```sql
CREATE OR ALTER PROCEDURE RAG.sp_SummarizeReports
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @id INT, @report NVARCHAR(MAX);
    DECLARE @payload NVARCHAR(MAX);
    DECLARE @response NVARCHAR(MAX);
    DECLARE @returnValue INT;
    DECLARE @summary NVARCHAR(MAX);

    DECLARE report_cursor CURSOR FOR
    SELECT RecordID, SustainabilityReport
    FROM RAG.ESG_TextData;

    OPEN report_cursor;

    FETCH NEXT FROM report_cursor INTO @id, @report;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- Build payload
        SET @payload = N'{
          "messages": [
            {
              "role": "system",
              "content": "Summarize this ESG report in 2-3 concise lines."
            },
            {
              "role": "user",
              "content": "' + REPLACE(@report, '"', '\"') + N'"
            }
          ],
          "max_tokens": 100
        }';

        -- Call LLM
        EXEC @returnValue = sp_invoke_external_rest_endpoint
            @url = N'https://<endpoint>.openai.azure.com/openai/deployments/<chat-model>/chat/completions?api-version=2024-02-15-preview',
            @method = 'POST',
            @payload = @payload,
            @credential = [https://<endpoint>.cognitiveservices.azure.com/],
            @response = @response OUTPUT;

        IF @returnValue = 0
        BEGIN
            SET @summary = JSON_VALUE(@response, '$.result.choices[0].message.content');

            UPDATE RAG.ESG_TextData
            SET Summary = @summary
            WHERE RecordID = @id;
        END

        FETCH NEXT FROM report_cursor INTO @id, @report;
    END

    CLOSE report_cursor;
    DEALLOCATE report_cursor;
END;
```

#### Run Summarization

```sql
EXEC RAG.sp_SummarizeReports;
```

#### View Results

```sql
SELECT 
    CompanyName,
    ReviewText,
    Sentiment,
    Summary
FROM RAG.ESG_TextData;
```
