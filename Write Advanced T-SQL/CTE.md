### Implementing Common Table Expressions (CTEs)

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG system now has:

1) Complex joins
2) Repeated subqueries
3) Need for hierarchical analysis

You decide to use CTEs to:

1) Simplify queries
2) Improve readability
3) Build layered logic

#### Build a Basic Non-Recursive CTE

Total Emissions per Company:
```sql
WITH EmissionSummary AS
(
    SELECT 
        CompanyID,
        SUM(CO2_Emissions) AS TotalEmissions
    FROM ESG.EmissionRecords
    GROUP BY CompanyID
)
SELECT 
    c.CompanyName,
    es.TotalEmissions
FROM ESG.Companies c
JOIN EmissionSummary es
ON c.CompanyID = es.CompanyID;
```

#### Multiple CTEs for Layered Logic

Ranking Companies by Emissions:
```sql 
WITH EmissionSummary AS
(
    SELECT 
        CompanyID,
        SUM(CO2_Emissions) AS TotalEmissions
    FROM ESG.EmissionRecords
    GROUP BY CompanyID
),
RankedCompanies AS
(
    SELECT 
        CompanyID,
        TotalEmissions,
        RANK() OVER (ORDER BY TotalEmissions DESC) AS EmissionRank
    FROM EmissionSummary
)
SELECT 
    c.CompanyName,
    rc.TotalEmissions,
    rc.EmissionRank
FROM RankedCompanies rc
JOIN ESG.Companies c
ON rc.CompanyID = c.CompanyID;
```

#### Filtering with CTE

High Emission Companies:
```sql
WITH HighEmitters AS
(
    SELECT 
        CompanyID,
        SUM(CO2_Emissions) AS TotalEmissions
    FROM ESG.EmissionRecords
    GROUP BY CompanyID
)
SELECT *
FROM HighEmitters
WHERE TotalEmissions > 2000;
```

