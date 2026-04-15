### Full-Text Search on ESG Data

#### Create Full-Text Catalog

```sql
CREATE FULLTEXT CATALOG ESG_FT_Catalog;
```

#### Create Full-Text Index

We will index `ReviewText` and `SustainabilityReport` in the `RAG.ESG_TextData` table.

```sql
CREATE FULLTEXT INDEX ON RAG.ESG_TextData
(
    ReviewText LANGUAGE 1033,
    SustainabilityReport LANGUAGE 1033
)
KEY INDEX <your-primary-key-index-name>
ON ESG_FT_Catalog;
```

#### Basic Search

Search for a keyword in `ReviewText` column:
```sql
SELECT 
    CompanyName,
    ReviewText
FROM RAG.ESG_TextData
WHERE CONTAINS(ReviewText, 'emissions');
```

#### Free Text Search

Search for a concept in `SustainabilityReport` column:
```sql
SELECT 
    CompanyName,
    SustainabilityReport
FROM RAG.ESG_TextData
WHERE FREETEXT(SustainabilityReport, 'reducing pollution');
```

#### Phrase Search
Search for an exact phrase in `SustainabilityReport` column:
```sql
SELECT 
    CompanyName,
    SustainabilityReport
FROM RAG.ESG_TextData
WHERE CONTAINS(SustainabilityReport, '"renewable energy"');
```

#### Prefix Search

Search for words starting with a prefix in `SustainabilityReport` column:
```sql
SELECT 
    CompanyName,
    SustainabilityReport
FROM RAG.ESG_TextData
WHERE CONTAINS(SustainabilityReport, ' "gr*" ');
```

#### Inflectional Search

Search for different forms of a word in `SustainabilityReport` column:
```sql
SELECT 
    CompanyName,
    SustainabilityReport
FROM RAG.ESG_TextData
WHERE CONTAINS(SustainabilityReport, 'FORMSOF(INFLECTIONAL, "reduce")');
```

```sql
SELECT 
    CompanyName,
    SustainabilityReport
FROM RAG.ESG_TextData
WHERE CONTAINS(SustainabilityReport, 'FORMSOF(INFLECTIONAL, "position")');
```


#### Ranked Search

Search for a keyword and get relevance ranking:
```sql
SELECT 
    t.RecordID,
    t.CompanyName,
    t.SustainabilityReport,
    ft.RANK
FROM RAG.ESG_TextData t
INNER JOIN CONTAINSTABLE(
    RAG.ESG_TextData,
    SustainabilityReport,
    'emissions'
) ft
ON t.RecordID = ft.[KEY]
ORDER BY ft.RANK DESC;
```

#### Proximity Search
Search for two words within a certain distance in `SustainabilityReport` column:

```sql
SELECT 
    CompanyName,
    SustainabilityReport
FROM RAG.ESG_TextData
WHERE CONTAINS(
    SustainabilityReport,
    'NEAR((sustainability, reporting))'
);
```