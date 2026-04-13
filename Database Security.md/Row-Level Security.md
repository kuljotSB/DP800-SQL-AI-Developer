### Implement Row-Level Security (RLS)

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your platform is multi-tenant:

1) Each company should see ONLY its own data
2) Data is stored in a shared table

You must enforce: Row-level access control without changing application code

#### Prepare Table (Tenant-Aware)

```sql
CREATE TABLE ESG.TenantData (
    RecordID INT IDENTITY PRIMARY KEY,
    CompanyID INT,
    CompanyName NVARCHAR(100),
    Emissions INT
);
```

#### Insert Sample Data 

```sql
INSERT INTO ESG.TenantData (CompanyID, CompanyName, Emissions)
VALUES
(1, 'GreenSteel Ltd', 1000),
(2, 'EcoLogistics Pvt Ltd', 800),
(3, 'SolarGrid Energy', 600);
```

#### Create Security Schema

```sql
CREATE SCHEMA Security;
```

#### Create Predicate Function

```sql
CREATE FUNCTION Security.fn_TenantPredicate(@CompanyID INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN 
    SELECT 1 AS fn_result
    WHERE @CompanyID = CAST(SESSION_CONTEXT(N'CompanyID') AS INT);
```

#### Create Security Policy

```sql
CREATE SECURITY POLICY ESG_TenantPolicy
ADD FILTER PREDICATE Security.fn_TenantPredicate(CompanyID)
    ON ESG.TenantData
WITH (STATE = ON);
```

#### Test with Session Context

Simulate Company 1:
```sql
EXEC sp_set_session_context 
    @key = N'CompanyID', 
    @value = 1;

SELECT * FROM ESG.TenantData;
```

Simulate Company 2:
```sql
EXEC sp_set_session_context 
    @key = N'CompanyID', 
    @value = 2;

SELECT * FROM ESG.TenantData;
```

#### Add BLOCK Predicate for Insert Operations

```sql
ALTER SECURITY POLICY ESG_TenantPolicy
ADD BLOCK PREDICATE Security.fn_TenantPredicate(CompanyID)
    ON ESG.TenantData AFTER INSERT;
```

#### Test Insert Protection

```sql
EXEC sp_set_session_context 
    @key = N'CompanyID', 
    @value = 1;

INSERT INTO ESG.TenantData (CompanyID, CompanyName, Emissions)
VALUES (2, 'Invalid Insert', 999);
```

#### Inspect Policies

Get list of policies and their target tables:
```sql
SELECT 
    p.name AS PolicyName,
    p.is_enabled,
    o.name AS TableName
FROM sys.security_policies p
JOIN sys.security_predicates sp 
    ON p.object_id = sp.object_id
JOIN sys.objects o 
    ON sp.target_object_id = o.object_id;
```