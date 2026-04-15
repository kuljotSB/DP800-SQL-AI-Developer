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
        SET @payload = N'{
          "messages": [
            {
              "role": "system",
              "content": "Classify sentiment as Positive, Negative, or Neutral. Reply with one word only."
            },
            {
              "role": "user",
              "content": "' + REPLACE(@review, '"', '\"') + N'"
            }
          ],
          "model": "<YOUR-MODEL-NAME>",
          "max_completion_tokens": 20,
          "temperature": 0.4
        }';

        EXEC @returnValue = sp_invoke_external_rest_endpoint
            @url = N'https://<resource-name>.openai.azure.com/openai/v1/chat/completions',
            @method = 'POST',
            @payload = @payload,
            @credential = [https://<resource-name>.openai.azure.com/],
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

#### View Results

```sql
SELECT 
    CompanyName,
    ReviewText,
    Sentiment
FROM RAG.ESG_TextData;
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
        -- STRING_ESCAPE handles quotes, newlines, tabs, backslashes etc.
        SET @payload = N'{
          "messages": [
            {
              "role": "system",
              "content": "Summarize this ESG report in 2-3 concise lines."
            },
            {
              "role": "user",
              "content": "' + STRING_ESCAPE(@report, 'json') + N'"
            }
          ],
          "model": "<MODEL-NAME>",
          "max_completion_tokens": 200,
          "temperature": 0.4
        }';

        EXEC @returnValue = sp_invoke_external_rest_endpoint
            @url = N'https://<resource-name>.openai.azure.com/openai/v1/chat/completions',
            @method = 'POST',
            @payload = @payload,
            @credential = [https://<resource-name>.openai.azure.com/],
            @response = @response OUTPUT;

        IF @returnValue = 0
        BEGIN
            SET @summary = JSON_VALUE(@response, '$.result.choices[0].message.content');
            UPDATE RAG.ESG_TextData
            SET Summary = @summary
            WHERE RecordID = @id;
        END
        ELSE
        BEGIN
            -- Log error per row so cursor continues instead of stopping
            UPDATE RAG.ESG_TextData
            SET Summary = 'ERROR: ' + CAST(@returnValue AS NVARCHAR(10)) 
                        + ' - ' + JSON_VALUE(@response, '$.response.status.http.description')
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
    SustainabilityReport,
    Summary
FROM RAG.ESG_TextData;
```
