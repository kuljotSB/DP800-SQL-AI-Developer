### Views - Logical vs Materialized (Indexed Views)

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG analytics system has:

1) Frequently used queries
2) Heavy aggregations (SUM, GROUP BY)
3) Performance bottlenecks

You decide to:

1) Use normal views for abstraction
2) Use indexed views for performance

#### Create Non-Materialized View

```sql
CREATE VIEW ESG.vw_TotalEmissions_Normal
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
SELECT * FROM ESG.vw_TotalEmissions_Normal;
```

#### Create Indexed (Materialized View)

Before creating:
```sql
SET ANSI_NULLS ON;
SET QUOTED_IDENTIFIER ON;
```

Create view with SCHEMABINDING:
```sql
CREATE VIEW ESG.vw_TotalEmissions_Indexed
WITH SCHEMABINDING
AS
SELECT 
    c.CompanyID,
    c.CompanyName,
    SUM(e.CO2_Emissions) AS TotalEmissions,
    COUNT_BIG(*) AS TotalRows
FROM ESG.Companies c
JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID
GROUP BY c.CompanyID, c.CompanyName;
```

Create unique clustered index to materialize:
```sql
CREATE UNIQUE CLUSTERED INDEX IX_vw_TotalEmissions
ON ESG.vw_TotalEmissions_Indexed (CompanyID);
```

```sql
SELECT * FROM ESG.vw_TotalEmissions_Indexed;
```

#### Lab Cleanup

```sql
DROP VIEW ESG.vw_TotalEmissions_Normal;
DROP VIEW ESG.vw_TotalEmissions_Indexed;
```
