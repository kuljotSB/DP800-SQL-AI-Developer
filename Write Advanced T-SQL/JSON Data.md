### Processing JSON Data with T-SQL

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform now:

1) Receives data from APIs in JSON format
2) Stores semi-structured ESG metadata
3) Needs to generate JSON responses for frontend + AI systems

You must process JSON directly in SQL


#### Store JSON Data

Add JSON Column to Companies:
```sql
ALTER TABLE ESG.Companies
ADD ESG_Metadata NVARCHAR(MAX);
```

Insert JSON Data:
```sql
UPDATE ESG.Companies
SET ESG_Metadata = CASE CompanyID
    WHEN 1 THEN '{"rating": "A", "esgScore": 85, "audits": {"lastAuditYear": 2024, "passed": true}}'
    WHEN 2 THEN '{"rating": "B", "esgScore": 72, "audits": {"lastAuditYear": 2023, "passed": true}}'
    WHEN 3 THEN '{"rating": "C", "esgScore": 60, "audits": {"lastAuditYear": 2022, "passed": false}'
    WHEN 4 THEN '{"rating": "A", "esgScore": 90, "audits": {"lastAuditYear": 2024, "passed": true}}'
    WHEN 5 THEN '{"rating": "B", "esgScore": 78, "audits": {"lastAuditYear": 2023, "passed": true}}'
    WHEN 6 THEN '{"rating": "C", "esgScore": 55, "audits": {"lastAuditYear": 2022, "passed": false}}'
    WHEN 7 THEN '{"rating": "A", "esgScore": 88, "audits": {"lastAuditYear": 2024, "passed": true}}'
    WHEN 8 THEN '{"rating": "B", "esgScore": 80, "audits": {"lastAuditYear": 2023, "passed": true}}'
    ELSE NULL
END
WHERE CompanyID IN (1, 2, 3, 4, 5, 6, 7, 8);
```

#### Extract Data from JSON

Use JSON_VALUE to extract scalar values:
```sql
SELECT 
    CompanyID,
    CompanyName,
    JSON_VALUE(ESG_Metadata, '$.rating') AS ESG_Rating,
    JSON_VALUE(ESG_Metadata, '$.esgScore') AS ESG_Score
FROM ESG.Companies;
```

Use JSON_QUERY to extract nested objects:
```sql
SELECT 
    CompanyID,
    CompanyName,
    JSON_QUERY(ESG_Metadata, '$.audits') AS AuditInfo
FROM ESG.Companies;
```

#### Parse JSON Arrays

Use OPENJSON to parse arrays:
```sql
DECLARE @json NVARCHAR(MAX) = '
[
    {"year": 2023, "emissions": 1000},
    {"year": 2024, "emissions": 800}
]';
```

Parse into rows:
```sql
SELECT *
FROM OPENJSON(@json)
WITH (
    Year INT '$.year',
    Emissions INT '$.emissions'
);
```

#### Combine JSON with Tables

Simulate JSON Column in EmissionRecords:
```sql
ALTER TABLE ESG.EmissionRecords
ADD ExtraData NVARCHAR(MAX);
```

```sql
UPDATE ESG.EmissionRecords
SET ExtraData = '
{
    "source": "sensor",
    "verified": true
}';
```

Parse with CROSS APPLY:
```sql
SELECT 
    e.CompanyID,
    e.CO2_Emissions,
    j.source,
    j.verified
FROM ESG.EmissionRecords e
CROSS APPLY OPENJSON(e.ExtraData)
WITH (
    source NVARCHAR(50) '$.source',
    verified BIT '$.verified'
) j;
```

#### Construct JSON

JSON_OBJECT to create JSON from columns:
```sql
SELECT JSON_OBJECT(
    'company': CompanyName,
    'country': Country
) AS CompanyJson
FROM ESG.Companies;
```

JSON_ARRAY
```sql
SELECT JSON_ARRAY('ESG', 'AI', 'Sustainability') AS Tags;
```

#### Aggregate JSON

```sql
SELECT 
    CompanyID,
    JSON_ARRAYAGG(CO2_Emissions) AS EmissionHistory
FROM ESG.EmissionRecords
GROUP BY CompanyID;
```

#### Convert SQL to JSON

```sql
SELECT 
    CompanyID,
    CompanyName,
    Country
FROM ESG.Companies
FOR JSON PATH, ROOT('companies');
```
