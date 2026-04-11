### Mastering SQL Joins for ESG Data Analysis

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform stores data across multiple tables:
1) Companies
2) EmissionRecords
3) EnergyConsumption
4) SustainabilityReports

To generate insights for clients, you must combine data across these tables using SQL joins.

#### Add More Data

Add more companies:
```sql
INSERT INTO ESG.Companies (CompanyName, Industry, Country)
VALUES
('AeroFly Aviation', 'Aviation', 'US'),
('BlueOcean Shipping', 'Logistics', 'NL'),
('GreenFoods Ltd', 'Food', 'IN'),
('FutureTech AI', 'Technology', 'US');
```

Add more emission records:
```sql
INSERT INTO ESG.EmissionRecords (CompanyID, EmissionDate, Scope, CO2_Emissions, Source)
VALUES
-- Aviation (high emissions)
(5, '2025-01-10', 1, 5000.00, 'Jet Fuel'),
(5, '2025-01-12', 3, 2000.00, 'Supply Chain'),

-- Shipping
(6, '2025-01-11', 1, 3000.00, 'Marine Fuel'),

-- Food industry (moderate)
(7, '2025-01-15', 2, 700.00, 'Electricity'),

-- Tech company (low emissions)
(8, '2025-01-18', 2, 100.00, 'Data Centers');
```

#### Add Energy Consumption Data
```sql
INSERT INTO ESG.EnergyConsumption (CompanyID, EnergyType, Consumption, Unit, RecordedAt)
VALUES
(5, 'Jet Fuel', 8000.00, 'liters', '2025-01-10 08:00:00'),
(6, 'Marine Diesel', 6000.00, 'liters', '2025-01-11 09:00:00'),
(7, 'Electricity', 1200.00, 'kWh', '2025-01-15 10:00:00'),
(8, 'Cloud Compute', 300.00, 'kWh', '2025-01-18 11:00:00');
```

#### Add Reports
```sql
INSERT INTO ESG.SustainabilityReports (CompanyID, ReportText)
VALUES
(5, 'AeroFly is exploring sustainable aviation fuel.'),
(7, 'GreenFoods is optimizing supply chain emissions.');
```

#### Task 1: INNER JOIN (Most Common)

Get company names with emission records
```sql
SELECT c.CompanyName, e.CO2_Emissions, e.Scope
FROM ESG.Companies c
INNER JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID;
```

#### Task 2: INNER JOIN with Aggregation

Total emissions per company
```sql
SELECT c.CompanyName, SUM(e.CO2_Emissions) AS TotalEmissions
FROM ESG.Companies c
INNER JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID
GROUP BY c.CompanyName;
```

#### Task 3: LEFT JOIN (Important for Analytics)

Get all companies + emissions (including missing)
```sql
SELECT c.CompanyName, e.CO2_Emissions
FROM ESG.Companies c
LEFT JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID;
```

#### Task 3: Find Missing Data using LEFT JOIN

Companies with NO emission records
```sql
SELECT c.CompanyName
FROM ESG.Companies c
LEFT JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID
WHERE e.CompanyID IS NULL;
```

#### Task 5: RIGHT JOIN

Get all emission records with company info
```sql
SELECT c.CompanyName, e.CO2_Emissions
FROM ESG.Companies c
RIGHT JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID;
```

#### Task 6: FULL OUTER JOIN

Combine all companies and emissions, showing all intersecting and non-intersecting records
```sql
SELECT c.CompanyName, e.CO2_Emissions
FROM ESG.Companies c
FULL OUTER JOIN ESG.EmissionRecords e
ON c.CompanyID = e.CompanyID;
```

#### Task 7: Multi-Table JOIN (Real Business Scenario)

Combine companies, emissions and energy data
```sql
SELECT 
    c.CompanyName,
    e.CO2_Emissions,
    ec.Consumption,
    e.Source
FROM ESG.Companies c
INNER JOIN ESG.EmissionRecords e
    ON c.CompanyID = e.CompanyID
INNER JOIN ESG.EnergyConsumption ec
    ON c.CompanyID = ec.CompanyID;
```

#### Task 8: JOIN with Aggregation

Total emissions + total energy per company
```sql
SELECT 
    c.CompanyName,
    SUM(e.CO2_Emissions) AS TotalEmissions,
    SUM(ec.Consumption) AS TotalEnergy
FROM ESG.Companies c
LEFT JOIN ESG.EmissionRecords e
    ON c.CompanyID = e.CompanyID
LEFT JOIN ESG.EnergyConsumption ec
    ON c.CompanyID = ec.CompanyID
GROUP BY c.CompanyName;
```
