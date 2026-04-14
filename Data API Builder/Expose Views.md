### Expose Views via DAB

#### Scenario
At CarbonOps, your frontend now needs:

1) Aggregated ESG insights (views)
2) Controlled business logic (stored procedures)

Goal: Extend your DAB API to support analytics + controlled operations

#### Create ESG Views

View 1: Company Emissions Summary
```sql
CREATE OR ALTER VIEW ESG.vw_CompanyEmissions AS
SELECT 
    c.CompanyID,
    c.CompanyName,
    SUM(e.CO2_Emissions) AS TotalEmissions,
    COUNT(*) AS RecordCount
FROM ESG.Companies c
JOIN ESG.EmissionRecords e
    ON c.CompanyID = e.CompanyID
GROUP BY c.CompanyID, c.CompanyName;
```

Create a view for Energy Efficiency:
```sql
CREATE OR ALTER VIEW ESG.vw_EnergyEfficiency AS
SELECT 
    ec.CompanyID,
    SUM(ec.Consumption) AS TotalEnergy,
    SUM(e.CO2_Emissions) AS TotalEmissions,
    CASE 
        WHEN SUM(ec.Consumption) = 0 THEN NULL
        ELSE SUM(e.CO2_Emissions) / SUM(ec.Consumption)
    END AS EmissionPerUnitEnergy
FROM ESG.EnergyConsumption ec
JOIN ESG.EmissionRecords e
    ON ec.CompanyID = e.CompanyID
GROUP BY ec.CompanyID;
```

#### Add Views to DAB

```bash
dab add CompanyEmissions \
--source ESG.vw_CompanyEmissions \
--source.type view \
--source.key-fields "CompanyID" \
--permissions "anonymous:read"
```

```bash
dab add EnergyEfficiency \
--source ESG.vw_EnergyEfficiency \
--source.type view \
--source.key-fields "CompanyID" \
--permissions "anonymous:read"
```

#### Test Views

```bash
GET http://localhost:5000/api/CompanyEmissions
GET http://localhost:5000/api/EnergyEfficiency
```