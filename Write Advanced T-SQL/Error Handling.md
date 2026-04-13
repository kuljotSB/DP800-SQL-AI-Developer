### Error Handling with TRY...CATCH and Transactions

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG system performs:

1) Emission updates
2) Bulk inserts
3) Compliance checks

But failures can occur:

1) Invalid emission values
2) Missing company records
3) Partial updates

Your goal: Ensure data integrity, logging, and controlled failure handling

#### Create Error Log Table

```sql
CREATE TABLE ESG.ErrorLog (
    ErrorID INT IDENTITY PRIMARY KEY,
    ErrorTime DATETIME DEFAULT GETDATE(),
    ErrorNumber INT,
    ErrorSeverity INT,
    ErrorState INT,
    ErrorLine INT,
    ErrorProcedure NVARCHAR(100),
    ErrorMessage NVARCHAR(MAX)
);
```

#### Basic TRY...CATCH Example

```sql
BEGIN TRY
    SELECT 1/0;  -- error
END TRY
BEGIN CATCH
    PRINT 'Error occurred';
END CATCH;
```

#### Capture Error Details

```sql
BEGIN TRY
    INSERT INTO ESG.Companies (CompanyID)
    VALUES (1);  -- duplicate PK
END TRY
BEGIN CATCH
    INSERT INTO ESG.ErrorLog (
        ErrorNumber,
        ErrorSeverity,
        ErrorState,
        ErrorLine,
        ErrorProcedure,
        ErrorMessage
    )
    VALUES (
        ERROR_NUMBER(),
        ERROR_SEVERITY(),
        ERROR_STATE(),
        ERROR_LINE(),
        ERROR_PROCEDURE(),
        ERROR_MESSAGE()
    );

    THROW;
END CATCH;
```


#### Transaction and Error Handling (ACID - All or Nothing)

```sql
BEGIN TRY
    BEGIN TRANSACTION;

    UPDATE ESG.Companies
    SET Industry = 'Updated'
    WHERE CompanyID = 1;

    -- Step 2 (this is the error step)
    INSERT INTO ESG.Companies (CompanyID, CompanyName)
    VALUES (1, 'Carbonops Steel');  -- PK violation

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    PRINT 'Transaction rolled back';

    THROW;
END CATCH;
```

#### Custom Error with THROW

```sql
DECLARE @EmissionValue INT = -100;

IF @EmissionValue < 0
    THROW 50001, 'Emission value cannot be negative', 1;
```

Use Custom Error in a Stored Procedure:

```sql
CREATE PROCEDURE ESG.UpdateEmission(@CompanyID INT, @EmissionValue INT)
AS
BEGIN
    IF @EmissionValue < 0
        THROW 50001, 'Emission value cannot be negative', 1;
    UPDATE ESG.EmissionRecords
    SET CO2_Emissions = @EmissionValue
    WHERE CompanyID = @CompanyID;
END
```

Invoke the procedure:

```sql
EXEC ESG.UpdateEmission @CompanyID = 1, @EmissionValue = -100;
```

#### Formatted Error Messages with RAISERROR

```sql
DECLARE @CompanyName NVARCHAR(100) = 'GreenSteel';
DECLARE @Value INT = -50;

IF @Value < 0
BEGIN
    RAISERROR(
        'Invalid emission for %s. Value: %d',
        16,
        1,
        @CompanyName,
        @Value
    );
END;
```

Alter the procedure to use RAISERROR:

```sql
ALTER PROCEDURE ESG.UpdateEmission(@CompanyID INT, @EmissionValue INT)
AS
BEGIN
    IF @EmissionValue < 0
    BEGIN
        DECLARE @CompanyName NVARCHAR(100);
        SELECT @CompanyName = CompanyName FROM ESG.Companies WHERE CompanyID = @CompanyID;

        RAISERROR(
            'Invalid emission for %s. Value: %d',
            16,
            1,
            @CompanyName,
            @EmissionValue
        );
        RETURN;
    END

    UPDATE ESG.EmissionRecords
    SET CO2_Emissions = @EmissionValue
    WHERE CompanyID = @CompanyID;
END
```

Invoke the procedure again:

```sql
EXEC ESG.UpdateEmission @CompanyID = 1, @EmissionValue = -50;
```

#### Rollback Transactions with XACT_ABORT

```sql
SET XACT_ABORT ON;
    BEGIN TRANSACTION;

    UPDATE ESG.Companies SET Industry = 'Test';
    SELECT 1/0;  -- auto rollback

    COMMIT TRANSACTION;
```

