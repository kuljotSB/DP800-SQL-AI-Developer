### Expose Stored Procedures via DAB

#### Scenario

At CarbonOps, your frontend now needs:

1) Aggregated ESG insights (views)
2) Controlled business logic (stored procedures)

Goal: Extend your DAB API to support analytics + controlled operations

#### Add Stored Procedures

Insert Emission Record:
```sql
CREATE OR ALTER PROCEDURE ESG.sp_AddEmission
    @CompanyID INT,
    @CO2 DECIMAL(10,2),
    @Scope INT,
    @Date DATE
AS
BEGIN
    INSERT INTO ESG.EmissionRecords
    (CompanyID, CO2_Emissions, Scope, EmissionDate)
    VALUES (@CompanyID, @CO2, @Scope, @Date);
END;
```

Get High Emission Companies:
```sql
CREATE OR ALTER PROCEDURE ESG.sp_GetHighEmitters
    @Threshold DECIMAL(10,2)
AS
BEGIN
    SELECT 
        c.CompanyName,
        SUM(e.CO2_Emissions) AS TotalEmissions
    FROM ESG.Companies c
    JOIN ESG.EmissionRecords e
        ON c.CompanyID = e.CompanyID
    GROUP BY c.CompanyName
    HAVING SUM(e.CO2_Emissions) > @Threshold;
END;
```

#### Add Stored Procedures to DAB

```bash
dab add AddEmission \
--source ESG.sp_AddEmission \
--source.type stored-procedure \
--permissions "anonymous:execute"
```

```bash
dab add HighEmitters \
--source ESG.sp_GetHighEmitters \
--source.type stored-procedure \
--permissions "anonymous:execute"
```

#### Test Stored Procedures

Get High Emitters:
```bash
POST http://localhost:5000/api/HighEmitters
Content-Type: application/json

{
  "Threshold": 1000
}
```

Insert Data:
```bash
POST http://localhost:5000/api/AddEmission
Content-Type: application/json

{
  "CompanyID": 1,
  "CO2": 500,
  "Scope": 2,
  "Date": "2025-03-01"
}
```