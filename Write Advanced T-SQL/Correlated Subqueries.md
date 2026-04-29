### Compare Rows with Correlated Subqueries

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG analytics team wants to:

1) Compare company emissions against their own benchmarks
2) Detect outliers within each company
3) Identify anomalous emission records
4) Build contextual insights

You will use correlated subqueries to perform row-by-row comparisons

#### Basic Correlated Subquery

Find emission records above company average:
```sql
SELECT 
    e.CompanyID,
    e.EmissionDate,
    e.Source,
    e.CO2_Emissions
FROM ESG.EmissionRecords e
WHERE e.CO2_Emissions > (
    SELECT AVG(e2.CO2_Emissions)
    FROM ESG.EmissionRecords e2
    WHERE e2.CompanyID = e.CompanyID
);
```

#### JOIN with Correlated Subquery

Get company names for records above average:
```sql
SELECT 
    c.CompanyName,
    e.CO2_Emissions,
    e.EmissionDate,
    e.Source
FROM ESG.Companies c
JOIN ESG.EmissionRecords e
    ON c.CompanyID = e.CompanyID
WHERE e.CO2_Emissions > (
    SELECT AVG(e2.CO2_Emissions)
    FROM ESG.EmissionRecords e2
    WHERE e2.CompanyID = e.CompanyID
);
```

#### Use EXISTS with Correlated Subquery

Get companies that have atleast one emission record
```sql
SELECT 
    CompanyID,
    CompanyName
FROM ESG.Companies c
WHERE EXISTS (
    SELECT 1
    FROM ESG.EmissionRecords e
    WHERE e.CompanyID = c.CompanyID
);
```

#### Top-N per Company

Top 2 emissions per company:
```sql
SELECT 
    e.CompanyID,
    e.CO2_Emissions,
    e.EmissionDate,
    e.Source
FROM ESG.EmissionRecords e
WHERE (
    SELECT COUNT(*)
    FROM ESG.EmissionRecords e2
    WHERE e2.CompanyID = e.CompanyID
      AND e2.CO2_Emissions > e.CO2_Emissions
) < 2;
```

#### Detect Outlier

Detect unusually high emissions (> 150% of company average):
```sql
SELECT 
    e.CompanyID,
    e.CO2_Emissions,
    e.EmissionDate,
    e.Source
FROM ESG.EmissionRecords e
WHERE e.CO2_Emissions > (
    SELECT AVG(e2.CO2_Emissions) * 1.5
    FROM ESG.EmissionRecords e2
    WHERE e2.CompanyID = e.CompanyID
);
```
