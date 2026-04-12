### Implementing Table Valued Functions (TVFs)

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform needs:

1) Reusable query logic
2) Parameterized datasets
3) Ability to JOIN function outputs

You decide to use Table-Valued Functions.

#### Inline Table-Valued Function 

Get emissions for a company:
```sql
CREATE FUNCTION ESG.fn_GetEmissionsByCompany
(
    @CompanyID INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        CompanyID,
        EmissionDate,
        CO2_Emissions,
        Scope
    FROM ESG.EmissionRecords
    WHERE CompanyID = @CompanyID
);
```

Use in query:
```sql
SELECT *
FROM ESG.fn_GetEmissionsByCompany(1);
```

#### Use CROSS APPLY with TVF

```sql
SELECT 
    c.CompanyName,
    e.CO2_Emissions
FROM ESG.Companies c
CROSS APPLY ESG.fn_GetEmissionsByCompany(c.CompanyID) e;
```

#### Create Multi-Statement Table-Valued Function

Aggregate Emissions:
```sql
CREATE FUNCTION ESG.fn_GetEmissionSummary
(
    @StartDate DATE,
    @EndDate DATE
)
RETURNS @Summary TABLE
(
    CompanyID INT,
    TotalEmissions DECIMAL(12,2),
    AvgEmissions DECIMAL(12,2)
)
AS
BEGIN
    INSERT INTO @Summary
    SELECT 
        CompanyID,
        SUM(CO2_Emissions),
        AVG(CO2_Emissions)
    FROM ESG.EmissionRecords
    WHERE EmissionDate BETWEEN @StartDate AND @EndDate
    GROUP BY CompanyID;

    RETURN;
END;
```

Use in Query:
```sql
SELECT *
FROM ESG.fn_GetEmissionSummary('2025-01-01', '2025-12-31');
```

#### Join Multi-TVF with Tables

```sql
SELECT 
    c.CompanyName,
    s.TotalEmissions
FROM ESG.Companies c
JOIN ESG.fn_GetEmissionSummary('2025-01-01', '2025-12-31') s
ON c.CompanyID = s.CompanyID;
```


#### Lab Cleanup

```sql
DROP FUNCTION ESG.fn_GetEmissionsByCompany;
DROP FUNCTION ESG.fn_GetEmissionSummary;
```
