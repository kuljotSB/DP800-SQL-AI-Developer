### Designing and Populating an ESG Database for CarbonOps

#### Scenario

You are a SQL AI Developer at CarbonOps.

CarbonOps is onboarding enterprise clients and needs a scalable and efficient SQL database to manage:
1) Company information
2) Carbon emissions (Scope 1, 2, 3)
3) Energy consumption
4) Sustainability reports

Before building AI-powered solutions, you must:
1) Design a structured schema
2) Create optimized tables
3) Insert realistic ESG data
4) Validate the system


#### Create ESG Schema
```sql
CREATE SCHEMA ESG;
```

#### Create Tables

Companies Table
```sql
CREATE TABLE ESG.Companies (
    CompanyID INT PRIMARY KEY IDENTITY(1,1),
    CompanyName NVARCHAR(100) NOT NULL,
    Industry NVARCHAR(50) NOT NULL,
    Country CHAR(2) NOT NULL,
    CreatedDate DATETIME2 DEFAULT GETUTCDATE()
);
```

EmissionRecords Table
```sql
CREATE TABLE ESG.EmissionRecords (
    RecordID INT PRIMARY KEY IDENTITY(1,1),
    CompanyID INT NOT NULL,
    EmissionDate DATE NOT NULL,
    Scope TINYINT NOT NULL,
    CO2_Emissions DECIMAL(10,2) NOT NULL,
    Source NVARCHAR(50),

    CONSTRAINT FK_Emission_Company 
    FOREIGN KEY (CompanyID) REFERENCES ESG.Companies(CompanyID)
);
```

EnergyConsumption Table
```sql
CREATE TABLE ESG.EnergyConsumption (
    EnergyID INT PRIMARY KEY IDENTITY(1,1),
    CompanyID INT NOT NULL,
    EnergyType NVARCHAR(50) NOT NULL,
    Consumption DECIMAL(12,2) NOT NULL,
    Unit VARCHAR(10) NOT NULL,
    RecordedAt DATETIME2 NOT NULL,

    CONSTRAINT FK_Energy_Company
    FOREIGN KEY (CompanyID) REFERENCES ESG.Companies(CompanyID)
);
```

SustainabilityReports Table
```sql
CREATE TABLE ESG.SustainabilityReports (
    ReportID INT PRIMARY KEY IDENTITY(1,1),
    CompanyID INT NOT NULL,
    ReportText NVARCHAR(MAX),
    GeneratedDate DATETIME2 DEFAULT GETUTCDATE(),

    CONSTRAINT FK_Report_Company
    FOREIGN KEY (CompanyID) REFERENCES ESG.Companies(CompanyID)
);
```


#### Insert Sample Data

Companies
```sql
INSERT INTO ESG.Companies (CompanyName, Industry, Country)
VALUES
('GreenSteel Ltd', 'Manufacturing', 'IN'),
('EcoLogistics Pvt Ltd', 'Transportation', 'IN'),
('SolarGrid Energy', 'Energy', 'US'),
('UrbanRetail Corp', 'Retail', 'UK');
```

Emission Records
```sql
INSERT INTO ESG.EmissionRecords (CompanyID, EmissionDate, Scope, CO2_Emissions, Source)
VALUES
(1, '2025-01-15', 1, 1200.50, 'Furnace Combustion'),
(1, '2025-01-20', 2, 800.75, 'Electricity Usage'),

(2, '2025-01-10', 1, 450.25, 'Fleet Fuel'),
(2, '2025-01-12', 3, 300.40, 'Third-party Transport'),

(3, '2025-01-05', 2, 150.10, 'Grid Electricity'),
(3, '2025-01-18', 3, 90.00, 'Supply Chain'),

(4, '2025-01-08', 2, 600.60, 'Store Electricity'),
(4, '2025-01-22', 3, 250.30, 'Logistics');
```

Energy Consumption
```sql
INSERT INTO ESG.EnergyConsumption (CompanyID, EnergyType, Consumption, Unit, RecordedAt)
VALUES
(1, 'Coal', 5000.00, 'kWh', '2025-01-15 08:00:00'),
(1, 'Electricity', 3200.50, 'kWh', '2025-01-20 09:00:00'),

(2, 'Diesel', 1200.75, 'liters', '2025-01-10 07:30:00'),
(2, 'Petrol', 800.20, 'liters', '2025-01-12 08:15:00'),

(3, 'Solar', 7000.00, 'kWh', '2025-01-05 10:00:00'),
(3, 'Grid', 1500.00, 'kWh', '2025-01-18 11:00:00'),

(4, 'Electricity', 2500.00, 'kWh', '2025-01-08 09:30:00'),
(4, 'Gas', 900.00, 'kWh', '2025-01-22 10:45:00');
```

Sustainability Reports
```sql
INSERT INTO ESG.SustainabilityReports (CompanyID, ReportText)
VALUES
(1, 'GreenSteel reduced Scope 1 emissions through furnace optimization.'),
(2, 'EcoLogistics is transitioning to electric fleets to reduce emissions.'),
(3, 'SolarGrid is expanding renewable energy capacity.'),
(4, 'UrbanRetail is optimizing supply chain sustainability.');
```

