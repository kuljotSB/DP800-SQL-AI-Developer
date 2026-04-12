### Fuzzy String Matching for ESG Data Quality

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG system has:

1) Duplicate company names
2) Misspelled entries
3) Inconsistent naming across systems

Example problems:

1) GreenSteel Ltd vs Green Steel Limited
2) EcoLogistics Pvt Ltd vs Eco Logistic Pvt. Ltd
3) SolarGrid Energy vs Solar Grid Energi

Your task: Use fuzzy matching to detect similar records


#### Create Messy Duplicate Data

```sql
INSERT INTO ESG.Companies (CompanyName, Industry, Country)
VALUES
('Green Steel Ltd', 'Manufacturing', 'IN'),
('GreenSteel Limited', 'Manufacturing', 'IN'),
('Eco Logistic Pvt Ltd', 'Transportation', 'IN'),
('EcoLogistics Pvt. Ltd', 'Transportation', 'IN'),
('Solar Grid Energi', 'Energy', 'US'),
('SolarGrid Energy', 'Energy', 'US');
```

#### Understand Edit Distance

Basic Examples:
```sql
SELECT 
    EDIT_DISTANCE('color', 'colour') AS ColorVariant,
    EDIT_DISTANCE('database', 'databaes') AS Typo,
    EDIT_DISTANCE('SQL Server', 'SQL Server') AS Exact,
    EDIT_DISTANCE('hello', 'world') AS Different;
```

#### Find Similar Company Names:
```sql
SELECT 
    c1.CompanyName AS Name1,
    c2.CompanyName AS Name2,
    EDIT_DISTANCE(c1.CompanyName, c2.CompanyName) AS Distance
FROM ESG.Companies c1
JOIN ESG.Companies c2
    ON c1.CompanyID < c2.CompanyID
WHERE EDIT_DISTANCE(c1.CompanyName, c2.CompanyName) <= 5
ORDER BY Distance;
```


#### Use Similarity Score:
```sql
SELECT 
    CompanyName,
    EDIT_DISTANCE_SIMILARITY(
        CAST('Green Steel Ltd' AS NVARCHAR(100)),
        CAST(CompanyName AS NVARCHAR(100))
    ) AS Score
FROM ESG.Companies;
```

#### Filter by Similarity Threshold:
```sql
SELECT 
    CompanyName,
    EDIT_DISTANCE_SIMILARITY(
        CAST('Eco Logistics Pvt Ltd' AS NVARCHAR(100)),
        CAST(CompanyName AS NVARCHAR(100))
    ) AS Score
FROM ESG.Companies
WHERE EDIT_DISTANCE_SIMILARITY(
        CAST('Eco Logistics Pvt Ltd' AS NVARCHAR(100)),
        CAST(CompanyName AS NVARCHAR(100))
    ) >= 70
ORDER BY Score DESC;
```

#### Jaro-Winkler Similarity:
```sql
SELECT 
    CompanyName,
    JARO_WINKLER_DISTANCE(
        CAST('GreenSteel Ltd' AS NVARCHAR(100)),
        CAST(CompanyName AS NVARCHAR(100))
    ) AS Similarity
FROM ESG.Companies
ORDER BY Similarity DESC;
```


#### Advanced Matching (First + Last Words):
```sql
SELECT 
    c.CompanyName,
    JARO_WINKLER_DISTANCE('Eco', LEFT(CAST(c.CompanyName AS VARCHAR(100)), 3)) AS PrefixScore,
    JARO_WINKLER_DISTANCE('Ltd', RIGHT(CAST(c.CompanyName AS VARCHAR(100)), 3)) AS SuffixScore
FROM ESG.Companies c;
```