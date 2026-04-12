### Implementing Scalar Functions for ESG Logic

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform requires:

1) Reusable calculations
2) Standardized business rules
3) Clean query logic

Instead of repeating logic in queries, you decide to use scalar functions.

#### Create a Basic Scalar Function

Classify Emission Level:
```sql
CREATE FUNCTION ESG.fn_GetEmissionLevel
(
    @EmissionValue DECIMAL(10,2)
)
RETURNS NVARCHAR(10)
AS
BEGIN
    DECLARE @Level NVARCHAR(10);

    IF @EmissionValue < 500
        SET @Level = 'Low';
    ELSE IF @EmissionValue < 1500
        SET @Level = 'Medium';
    ELSE
        SET @Level = 'High';

    RETURN @Level;
END;
```

Use in query:
```sql
SELECT 
    CompanyID,
    CO2_Emissions,
    ESG.fn_GetEmissionLevel(CO2_Emissions) AS EmissionLevel
FROM ESG.EmissionRecords;
```

Use Function in WHERE clause:
```sql
SELECT *
FROM ESG.EmissionRecords
WHERE ESG.fn_GetEmissionLevel(CO2_Emissions) = 'High';
```

#### Function with Business Logic

Carbon Tax Calculation:
```sql
CREATE FUNCTION ESG.fn_CalculateCarbonTax
(
    @EmissionValue DECIMAL(10,2),
    @Rate DECIMAL(5,2)
)
RETURNS DECIMAL(12,2)
AS
BEGIN
    DECLARE @Tax DECIMAL(12,2);

    SET @Tax = @EmissionValue * @Rate;

    RETURN @Tax;
END;
```

Use in query:
```sql
SELECT 
    CompanyID,
    CO2_Emissions,
    ESG.fn_CalculateCarbonTax(CO2_Emissions, 0.05) AS CarbonTax
FROM ESG.EmissionRecords;
```

#### Lab Cleanup

```sql
DROP FUNCTION ESG.fn_GetEmissionLevel;
DROP FUNCTION ESG.fn_CalculateCarbonTax;
```
