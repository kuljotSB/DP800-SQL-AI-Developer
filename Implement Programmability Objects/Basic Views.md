### Implementing Views for ESG Data Abstraction

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG database has grown complex:

1) Multiple tables (Companies, EmissionRecords, EnergyConsumption)
2) Frequent joins
3) Repeated business logic

The analytics team complains:

“We don’t want to write complex joins every time.”

Your task is to create views to:

1) Simplify queries
2) Standardize business logic
3) Improve usability and security

#### Create a Basic View

Companies Overview:
```sql
CREATE VIEW ESG.vw_Companies
AS
SELECT 
    CompanyID,
    CompanyName,
    Industry,
    Country
FROM ESG.Companies;
```

```sql
SELECT * FROM ESG.vw_Companies;
```

#### Create View with JOIN

Emission Summary:
```sql
CREATE VIEW ESG.vw_CompanyEmissions
AS
SELECT 
    c.CompanyName,
    c.Industry,
    e.EmissionDate,
    e.Scope,
    e.CO2_Emissions
FROM ESG.Companies c
INNER JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID;
```

```sql
SELECT *
FROM ESG.vw_CompanyEmissions
WHERE CompanyName = 'GreenSteel Ltd';
```

#### View with Aggregation

Total Emissions by Company:
```sql
CREATE VIEW ESG.vw_TotalEmissions
AS
SELECT 
    c.CompanyName,
    SUM(e.CO2_Emissions) AS TotalEmissions
FROM ESG.Companies c
JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID
GROUP BY c.CompanyName;
```

```sql
SELECT * FROM ESG.vw_TotalEmissions;
```

#### View with Calculated Column

Emission Category:
```sql
CREATE VIEW ESG.vw_EmissionCategory
AS
SELECT 
    RecordID,
    CompanyID,
    CO2_Emissions,
    CASE 
        WHEN CO2_Emissions < 500 THEN 'Low'
        WHEN CO2_Emissions < 1500 THEN 'Medium'
        ELSE 'High'
    END AS EmissionLevel
FROM ESG.EmissionRecords;
```

```sql
SELECT * FROM ESG.vw_EmissionCategory;
```

#### Lab Cleanup

```sql
DROP VIEW ESG.vw_Companies;
DROP VIEW ESG.vw_CompanyEmissions;
DROP VIEW ESG.vw_TotalEmissions;
DROP VIEW ESG.vw_EmissionCategory;
```
