### Traverse Relationships with Graph Queries

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG system now needs to model:

1) Company relationships
2) Supply chains
3) ESG influence networks

Instead of complex joins, you will use Graph Tables + MATCH

#### Create Node Tables

```sql
CREATE TABLE ESG.CompanyNode (
    CompanyID INT PRIMARY KEY,
    CompanyName NVARCHAR(100),
    Industry NVARCHAR(50)
) AS NODE;
```

Insert Data:
```sql
INSERT INTO ESG.CompanyNode (CompanyID, CompanyName, Industry)
SELECT CompanyID, CompanyName, Industry
FROM ESG.Companies;
```

#### Create Edge Table (Relationships)

```sql
CREATE TABLE ESG.SuppliesTo (
    SupplyType NVARCHAR(50),
    ContractValue DECIMAL(10,2)
) AS EDGE;
```

#### Insert Relationships

```sql
-- Company 1 supplies to Company 2
INSERT INTO ESG.SuppliesTo ($from_id, $to_id, SupplyType, ContractValue)
SELECT 
    (SELECT $node_id FROM ESG.CompanyNode WHERE CompanyID = 1),
    (SELECT $node_id FROM ESG.CompanyNode WHERE CompanyID = 2),
    'Raw Materials',
    50000;

-- Company 2 supplies to Company 3
INSERT INTO ESG.SuppliesTo ($from_id, $to_id, SupplyType, ContractValue)
SELECT 
    (SELECT $node_id FROM ESG.CompanyNode WHERE CompanyID = 2),
    (SELECT $node_id FROM ESG.CompanyNode WHERE CompanyID = 3),
    'Logistics',
    30000;
```

#### Basic Graph Query with MATCH

```sql
SELECT 
    c1.CompanyName AS Supplier,
    c2.CompanyName AS Customer,
    s.SupplyType,
    s.ContractValue
FROM ESG.CompanyNode c1,
     ESG.SuppliesTo s,
     ESG.CompanyNode c2
WHERE MATCH(c1-(s)->c2);
```

#### Multi-Hop Traversal

```sql
SELECT 
    c1.CompanyName AS StartCompany,
    c3.CompanyName AS FinalCompany
FROM ESG.CompanyNode c1,
     ESG.SuppliesTo s1,
     ESG.CompanyNode c2,
     ESG.SuppliesTo s2,
     ESG.CompanyNode c3
WHERE MATCH(c1-(s1)->c2-(s2)->c3);
```

#### Filter Graph Query

```sql
SELECT 
    c1.CompanyName,
    c2.CompanyName,
    s.ContractValue
FROM ESG.CompanyNode c1,
     ESG.SuppliesTo s,
     ESG.CompanyNode c2
WHERE MATCH(c1-(s)->c2)
  AND s.ContractValue > 40000;
```

#### Shortest Path Query

```sql
SELECT 
    StartCompany.CompanyName,
    LAST_VALUE(TargetCompany.CompanyName) 
        WITHIN GROUP (GRAPH PATH) AS ReachableCompany,
    COUNT(TargetCompany.CompanyName) 
        WITHIN GROUP (GRAPH PATH) AS Distance
FROM ESG.CompanyNode AS StartCompany,
     ESG.SuppliesTo FOR PATH AS s,
     ESG.CompanyNode FOR PATH AS TargetCompany
WHERE MATCH(SHORTEST_PATH(StartCompany(-(s)->TargetCompany){1,3}))
  AND StartCompany.CompanyID = 1;
```