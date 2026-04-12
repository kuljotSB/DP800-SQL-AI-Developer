### Creating and Working with Specialized Tables

#### Scenario

You are a SQL AI Developer at CarbonOps.

As your ESG platform scales, different workloads require specialized storage strategies:

1) High-speed ingestion → In-memory tables
2) Historical tracking → Temporal tables
3) Compliance → Ledger tables
4) Relationship modeling → Graph tables

Your task is to implement these specialized table types and understand their use cases.

#### Task 1: Create Temporal Table for Historical Tracking

Track Historical changes:
```sql
CREATE TABLE ESG.CompanyTargets (
    TargetID INT PRIMARY KEY IDENTITY,
    CompanyID INT,
    TargetEmission DECIMAL(10,2),

    SysStartTime DATETIME2 GENERATED ALWAYS AS ROW START,
    SysEndTime DATETIME2 GENERATED ALWAYS AS ROW END,
    PERIOD FOR SYSTEM_TIME (SysStartTime, SysEndTime)
)
WITH (SYSTEM_VERSIONING = ON);
```

Insert + Update data:
```sql
INSERT INTO ESG.CompanyTargets (CompanyID, TargetEmission)
VALUES (1, 1000);

UPDATE ESG.CompanyTargets
SET TargetEmission = 900
WHERE CompanyID = 1;
```

Query History:
```sql
SELECT *
FROM ESG.CompanyTargets
FOR SYSTEM_TIME ALL;
```

#### Task 2: Create Graph Table for Relationship Modeling

Create Node Table:
```sql
CREATE TABLE ESG.CompanyNode (
    CompanyID INT,
    CompanyName NVARCHAR(100)
) AS NODE;
```

Create Edge Table:
```sql
CREATE TABLE ESG.SuppliesTo AS EDGE;
```

Insert Nodes:
```sql
INSERT INTO ESG.CompanyNode VALUES
(1, 'GreenSteel'),
(2, 'EcoLogistics'),
(3, 'UrbanRetail');
```

Insert Relationships:
```sql
INSERT INTO ESG.SuppliesTo VALUES
((SELECT $node_id FROM ESG.CompanyNode WHERE CompanyID = 1),
 (SELECT $node_id FROM ESG.CompanyNode WHERE CompanyID = 2));
```

Query Graph:
```sql
SELECT c1.CompanyName, c2.CompanyName
FROM ESG.CompanyNode c1,
     ESG.SuppliesTo s,
     ESG.CompanyNode c2
WHERE MATCH (c1-(s)->c2);
```


#### Lab Cleanup

Drop tables:
```sql
ALTER TABLE ESG.CompanyTargets
SET (SYSTEM_VERSIONING = OFF);

DROP TABLE ESG.CompanyTargets;

DROP TABLE ESG.CompanyNode;
DROP TABLE ESG.SuppliesTo;
```
