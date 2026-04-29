### Mastering SQL Joins for ESG Data Analysis

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform stores data across multiple tables:
1) Companies
2) EmissionRecords
3) EnergyConsumption
4) SustainabilityReports

To generate insights for clients, you must combine data across these tables using SQL joins.

#### Create New Table

Create a new table `ESG.CarbonTargets` 
```sql
CREATE TABLE ESG.CarbonTargets (
    TargetID INT PRIMARY KEY IDENTITY(1,1),
    CompanyID INT,
    TargetYear INT,
    TargetReductionPercent DECIMAL(5,2)
);
```

Add data into the table:
```sql
INSERT INTO ESG.CarbonTargets (CompanyID, TargetYear, TargetReductionPercent)
VALUES
-- Valid matches (existing companies)
(1, 2030, 30.00),
(1, 2035, 45.00),

(2, 2030, 25.00),
(2, 2040, 50.00),

(3, 2035, 40.00),
(3, 2045, 55.00),

(4, 2040, 35.00),

(5, 2030, 20.00),
(6, 2035, 28.00),
(7, 2030, 22.00),
(8, 2040, 18.00),

-- Orphan records (NO matching company)
(999, 2030, 60.00),
(1000, 2040, 70.00);
```

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

```sql
SELECT c.CompanyName, t.TargetReductionPercent
FROM ESG.Companies c
INNER JOIN ESG.CarbonTargets t
ON c.CompanyID = t.CompanyID;
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

```sql
SELECT c.CompanyName, t.TargetReductionPercent
FROM ESG.Companies c
LEFT JOIN ESG.CarbonTargets t
ON c.CompanyID = t.CompanyID;
```

#### Task 3: Find Missing Data using LEFT JOIN

```sql
SELECT c.CompanyName, t.TargetReductionPercent
FROM ESG.Companies c
LEFT JOIN ESG.CarbonTargets t
ON c.CompanyID = t.CompanyID;
WHERE c.CompanyID IS NULL;
```

#### Task 5: RIGHT JOIN

```sql
SELECT c.CompanyName, t.TargetReductionPercent
FROM ESG.Companies c
RIGHT JOIN ESG.CarbonTargets t
ON c.CompanyID = t.CompanyID;
```

#### Task 6: FULL OUTER JOIN

```sql
SELECT c.CompanyName, t.TargetReductionPercent
FROM ESG.Companies c
FULL OUTER JOIN ESG.CarbonTargets t
ON c.CompanyID = t.CompanyID;
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
