### Implementing DML and DDL Triggers for ESG Data Integrity

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform requires:

1) Automatic audit logging
2) Data validation
3) Protection against unauthorized schema changes

You will implement triggers to enforce business rules automatically

#### Create Audit Table

```sql
CREATE TABLE ESG.EmissionAuditLog (
    AuditID INT IDENTITY PRIMARY KEY,
    CompanyID INT,
    OperationType NVARCHAR(10),
    OldValue DECIMAL(10,2),
    NewValue DECIMAL(10,2),
    ChangeDate DATETIME DEFAULT GETDATE()
);
```

#### Create DML Trigger (AFTER INSERT)

Log new emissions:
```sql
CREATE TRIGGER ESG.tr_LogEmissionInsert
ON ESG.EmissionRecords
AFTER INSERT
AS
BEGIN
    INSERT INTO ESG.EmissionAuditLog (CompanyID, OperationType, NewValue)
    SELECT 
        CompanyID,
        'INSERT',
        CO2_Emissions
    FROM inserted;
END;
```

Test Insert:
```sql
INSERT INTO ESG.EmissionRecords(CompanyID, EmissionDate, Scope, CO2_Emissions, Source)
VALUES (3, GETDATE(), 1, 1200, 'Internal');
```

#### Create DML Trigger (AFTER UPDATE)

Track emission changes:
```sql
CREATE TRIGGER ESG.tr_LogEmissionUpdate
ON ESG.EmissionRecords
AFTER UPDATE
AS
BEGIN
    INSERT INTO ESG.EmissionAuditLog 
    (CompanyID, OperationType, OldValue, NewValue)
    SELECT 
        d.CompanyID,
        'UPDATE',
        d.CO2_Emissions,
        i.CO2_Emissions
    FROM deleted d
    JOIN inserted i
    ON d.RecordID = i.RecordID
    WHERE d.CO2_Emissions <> i.CO2_Emissions;
END;
```

Test Update:
```sql
UPDATE ESG.EmissionRecords
SET CO2_Emissions = 1100
WHERE CompanyID = 3 AND EmissionDate = CAST(GETDATE() AS DATE) AND Scope = 1 AND Source = 'Internal';
```

#### Create a Data Validation Trigger 

Prevent invalid emissions:
```sql
CREATE TRIGGER ESG.tr_ValidateEmission
ON ESG.EmissionRecords
AFTER INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT * FROM inserted WHERE CO2_Emissions < 0
    )
    BEGIN
        THROW 50001, 'Emission value cannot be negative', 1;
    END;
END;
```

Try inserting invalid data:
```sql
INSERT INTO ESG.EmissionRecords(CompanyID, EmissionDate, Scope, CO2_Emissions, Source)
VALUES (4, GETDATE(), 1, -500, 'Internal');
```

#### Create a DDL Trigger to Prevent Schema Changes

Prevent table drop:
```sql
CREATE TRIGGER tr_PreventTableDrop
ON DATABASE
FOR DROP_TABLE
AS
BEGIN
    PRINT 'Table drop is not allowed in ESG system';
    ROLLBACK;
END;
```

Test:
```sql
DROP TABLE ESG.EmissionAuditLog;
```

#### Lab Cleanup

```sql
DROP TRIGGER ESG.tr_LogEmissionInsert;
DROP TRIGGER ESG.tr_LogEmissionUpdate;
DROP TRIGGER ESG.tr_ValidateEmission;
DROP TRIGGER tr_PreventTableDrop ON DATABASE;
DROP TABLE ESG.EmissionAuditLog;
```


