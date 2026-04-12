### Implementing Stored Procedures for ESG Workloads

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform needs:

1) Controlled access to data
2) Reusable business logic
3) Optimized query execution

The frontend team wants:

“No direct table access — everything via stored procedures.”

You will design stored procedures for:

1) Data retrieval
2) Parameterized filtering
3) Insert operations
4) Error handling

#### Create a Basic Stored Procedure

Get all companies:
```sql
CREATE PROCEDURE ESG.GetCompanies
AS
BEGIN
    SET NOCOUNT ON;

    SELECT 
        CompanyID,
        CompanyName,
        Industry,
        Country
    FROM ESG.Companies;
END;
```

```sql
EXEC ESG.GetCompanies;
```

#### Stored Procedure with Parameters

Get emissions by company:
```sql
CREATE PROCEDURE ESG.GetEmissionsByCompany
    @CompanyID INT
AS
BEGIN
    SET NOCOUNT ON;

    SELECT 
        CompanyID,
        EmissionDate,
        CO2_Emissions,
        Scope
    FROM ESG.EmissionRecords
    WHERE CompanyID = @CompanyID
    ORDER BY EmissionDate DESC;
END;
```

```sql
EXEC ESG.GetEmissionsByCompany @CompanyID = 1;
```

#### Stored Procedure with Optional Parameters

Filter by Date Range:
```sql
CREATE PROCEDURE ESG.GetEmissionsByDate
    @CompanyID INT,
    @StartDate DATETIME = NULL,
    @EndDate DATETIME = NULL
AS
BEGIN
    SET NOCOUNT ON;

    SELECT 
        CompanyID,
        EmissionDate,
        CO2_Emissions
    FROM ESG.EmissionRecords
    WHERE CompanyID = @CompanyID
      AND (@StartDate IS NULL OR EmissionDate >= @StartDate)
      AND (@EndDate IS NULL OR EmissionDate <= @EndDate);
END;
```

```sql
EXEC ESG.GetEmissionsByDate 
    @CompanyID = 1,
    @StartDate = '2025-01-01',
    @EndDate = '2025-03-31';
```

#### Stored Procedure with Output Parameters

Calculate Total Emissions:
```sql
CREATE PROCEDURE ESG.CalculateTotalEmissions
    @CompanyID INT,
    @TotalEmissions DECIMAL(12,2) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    SELECT @TotalEmissions = SUM(CO2_Emissions)
    FROM ESG.EmissionRecords
    WHERE CompanyID = @CompanyID;
END;
```

```sql
DECLARE @Total DECIMAL(12,2);

EXEC ESG.CalculateTotalEmissions 
    @CompanyID = 1,
    @TotalEmissions = @Total OUTPUT;

SELECT @Total AS TotalEmissions;
```

#### Insert Procedure with Validation

Insert emission record
```sql
CREATE PROCEDURE ESG.InsertEmissionRecord
    @CompanyID INT,
    @EmissionDate DATETIME,
    @CO2_Emissions DECIMAL(10,2),
    @Scope TINYINT
AS
BEGIN
    SET NOCOUNT ON;

    -- Validation
    IF @CompanyID IS NULL OR @CompanyID <= 0
    BEGIN
        RAISERROR('Invalid CompanyID.', 16, 1);
        RETURN;
    END;

    INSERT INTO ESG.EmissionRecords
    (
        CompanyID,
        EmissionDate,
        CO2_Emissions,
        Scope
    )
    VALUES
    (
        @CompanyID,
        @EmissionDate,
        @CO2_Emissions,
        @Scope
    );
END;
```


```sql
EXEC ESG.InsertEmissionRecord 
    @CompanyID = 1,
    @EmissionDate = '2025-05-01',
    @CO2_Emissions = 1200,
    @Scope = 1;
```

#### Error Handling with TRY...CATCH

```sql
CREATE PROCEDURE ESG.SafeInsertEmission
    @CompanyID INT,
    @EmissionDate DATETIME,
    @CO2_Emissions DECIMAL(10,2)
AS
BEGIN
    SET NOCOUNT ON;

    BEGIN TRY
        BEGIN TRANSACTION;

        IF NOT EXISTS (
            SELECT 1 FROM ESG.Companies WHERE CompanyID = @CompanyID
        )
        BEGIN
            RAISERROR('Company does not exist.', 16, 1);
        END

        INSERT INTO ESG.EmissionRecords
        (CompanyID, EmissionDate, CO2_Emissions)
        VALUES
        (@CompanyID, @EmissionDate, @CO2_Emissions);

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        DECLARE @ErrMsg NVARCHAR(4000) = ERROR_MESSAGE();

        RAISERROR(@ErrMsg, 16, 1);
    END CATCH
END;
```

```sql
EXEC ESG.SafeInsertEmission 
    @CompanyID = 999, -- Non-existent company
    @EmissionDate = '2025-05-01',
    @CO2_Emissions = 1200;
```

#### Lab Cleanup - Delete Stored Procedures

```sql
DROP PROCEDURE IF EXISTS ESG.GetCompanies;
DROP PROCEDURE IF EXISTS ESG.GetEmissionsByCompany;
DROP PROCEDURE IF EXISTS ESG.GetEmissionsByDate;
DROP PROCEDURE IF EXISTS ESG.CalculateTotalEmissions;
DROP PROCEDURE IF EXISTS ESG.InsertEmissionRecord;
DROP PROCEDURE IF EXISTS ESG.SafeInsertEmission;
```
