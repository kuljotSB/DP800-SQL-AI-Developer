### Implementing Data Partitioning for ESG Scale

#### Scenario
You are a SQL AI Developer at CarbonOps.

Your ESG platform is now handling millions of emission records over time.

The analytics team reports:

1) Queries filtering by date are slow
2) Archiving old data takes hours
3) Index rebuilds are expensive

You decide to implement table partitioning based on time (EmissionDate).

#### Create Partition Function

Partition by Date (Quarterly):
```sql
CREATE PARTITION FUNCTION PF_EmissionDate (DATETIME2)
AS RANGE RIGHT FOR VALUES
(
    '2025-01-01',
    '2025-04-01',
    '2025-07-01',
    '2025-10-01'
);
```

#### Create Partition Scheme

Apply to PRIMARY filegroup:
```sql
CREATE PARTITION SCHEME PS_EmissionDate
AS PARTITION PF_EmissionDate
ALL TO ([PRIMARY]);
```

#### Create Partitioned Table

Partition Column must be in Primary Key:
```sql
CREATE TABLE ESG.EmissionRecords_Partitioned (
    RecordID INT NOT NULL,
    CompanyID INT NOT NULL,
    EmissionDate DATETIME2 NOT NULL,
    Scope TINYINT,
    CO2_Emissions DECIMAL(10,2),

    CONSTRAINT PK_Emission_Partitioned
    PRIMARY KEY (RecordID, EmissionDate)
)
ON PS_EmissionDate(EmissionDate);
```

#### Insert Sample Data

```sql
INSERT INTO ESG.EmissionRecords_Partitioned
VALUES
(6, 2, '2024-10-10', 1, 1500),
(7, 3, '2024-11-05', 2, 900),
(8, 2, '2025-01-15', 1, 1100),
(9, 3, '2025-03-20', 3, 700),
(10, 1, '2025-04-05', 2, 950),
(11, 4, '2025-06-18', 1, 1300),
(12, 2, '2025-07-22', 2, 400),
(13, 3, '2025-09-10', 3, 300),
(14, 1, '2025-10-15', 1, 2000),
(15, 4, '2025-12-01', 2, 600),
(16, 1, '2025-02-05', 1, 1800),
(17, 1, '2025-02-25', 2, 1750),
(18, 1, '2025-03-05', 3, 1600),
(19, 5, '2025-08-01', 2, 50),
(20, 6, '2025-11-20', 3, 25);
```

#### View Partition Distribution

```sql
SELECT 
    $PARTITION.PF_EmissionDate(EmissionDate) AS PartitionNumber,
    COUNT(*) AS Rows,
    MIN(EmissionDate) AS MinDate,
    MAX(EmissionDate) AS MaxDate
FROM ESG.EmissionRecords_Partitioned
GROUP BY $PARTITION.PF_EmissionDate(EmissionDate)
ORDER BY PartitionNumber;
```

#### Query Performance

Query filtering by partition key
```sql
SELECT *
FROM ESG.EmissionRecords_Partitioned
WHERE EmissionDate BETWEEN '2025-01-01' AND '2025-03-31';
```

#### Add new Partition

```sql
ALTER PARTITION FUNCTION PF_EmissionDate()
SPLIT RANGE ('2026-01-01');
```

```sql
INSERT INTO ESG.EmissionRecords_Partitioned
VALUES
(21, 2, '2026-02-10', 1, 1200);
```

#### Create an Aligned Index on CompanyID

Create an aligned non-clustered index for faster lookups by CompanyID:
```sql
CREATE NONCLUSTERED INDEX IX_Emission_Company
ON ESG.EmissionRecords_Partitioned (CompanyID)
ON PS_EmissionDate(EmissionDate);
```

```sql
SELECT *
FROM ESG.EmissionRecords_Partitioned
WHERE CompanyID = 2
AND EmissionDate BETWEEN '2025-01-01' AND '2025-12-31';
```

#### Lab Cleanup

Drop partitioned table:
```sql
DROP TABLE ESG.EmissionRecords_Partitioned;
```

Drop partition scheme and function:
```sql
DROP PARTITION SCHEME PS_EmissionDate;
DROP PARTITION FUNCTION PF_EmissionDate;
```
