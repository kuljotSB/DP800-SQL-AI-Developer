### Querying and Analyzing ESG Data for CarbonOps

#### Scenario

You are a SQL AI Developer at CarbonOps.

A client has onboarded their ESG data into your system and now wants:

1) Insights into emissions
2) Energy usage trends
3) Sustainability performance

Your job is to query and analyze the data using SQL.

#### Task 1: Retrieve Basic Data

View all companies:
```sql
SELECT * FROM ESG.Companies;
```

View Emission records:
```sql
SELECT * FROM ESG.EmissionRecords;
```

#### Task 2: Filter Data Using WHERE

Find all Scope 1 Emissions:
```sql
SELECT *
FROM ESG.EmissionRecords
WHERE Scope = 1;
```

Find emissions greater than 500:
```sql
SELECT *
FROM ESG.EmissionRecords
WHERE CO2_Emissions > 500;
```

Find companies in India:
```sql
SELECT *
FROM ESG.Companies
WHERE Country = 'IN';
```

#### Task 3: Sort and Limit Results

Top 3 highest emission records:
```sql
SELECT TOP 3 *
FROM ESG.EmissionRecords
ORDER BY CO2_Emissions DESC;
```

Lowest emissions first:
```sql
SELECT *
FROM ESG.EmissionRecords
ORDER BY CO2_Emissions ASC;
```

#### Task 4: Aggregations - Business Insights

Total emissions per company:
```sql
SELECT CompanyID, SUM(CO2_Emissions) AS TotalEmissions
FROM ESG.EmissionRecords
GROUP BY CompanyID;
```

Average emissions:
```sql
SELECT AVG(CO2_Emissions) AS AvgEmissions
FROM ESG.EmissionRecords;
```

Maximum emissions recorded:
```sql
SELECT MAX(CO2_Emissions) AS MaxEmission
FROM ESG.EmissionRecords;
```

#### Task 5: Simple Data Manipulation

Update a company name:
```sql
UPDATE ESG.Companies
SET CompanyName = 'EcoLogistics International'
WHERE CompanyID = 2;
```

Delete a record:
```sql
DELETE FROM ESG.EmissionRecords
WHERE RecordID = 8;
```