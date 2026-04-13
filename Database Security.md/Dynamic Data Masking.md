### Implement Dynamic Data Masking in SQL

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG system stores sensitive company data:

1) Contact emails
2) Phone numbers
3) Financial metrics
4) Compliance identifiers

Requirement: Protect sensitive data without changing application code

#### Create Masked Table

```sql
CREATE TABLE ESG.CompanySensitive (
    CompanyID INT PRIMARY KEY,
    CompanyName NVARCHAR(100),

    Email NVARCHAR(100) 
        MASKED WITH (FUNCTION = 'email()'),

    Phone NVARCHAR(20) 
        MASKED WITH (FUNCTION = 'partial(3, "XXX-XXX-", 2)'),

    CreditRating NVARCHAR(10)
        MASKED WITH (FUNCTION = 'default()'),

    Revenue DECIMAL(18,2)
        MASKED WITH (FUNCTION = 'random(100000, 1000000)')
);
```

#### Insert Sample Data

```sql
INSERT INTO ESG.CompanySensitive
VALUES
(1, 'GreenSteel Ltd', 'contact@greensteel.com', '9876543210', 'AAA', 500000),
(2, 'EcoLogistics Pvt Ltd', 'support@ecologistics.com', '9123456789', 'BBB', 750000);
```

#### Create Different Users with Varying Access

```sql
CREATE USER SupportUser WITHOUT LOGIN;
CREATE USER FinanceUser WITHOUT LOGIN;
```

Grant base access:
```sql
GRANT SELECT ON ESG.CompanySensitive TO SupportUser;
GRANT SELECT ON ESG.CompanySensitive TO FinanceUser;
```

SupportUser -> only Phone unmasked:
```sql
GRANT UNMASK ON ESG.CompanySensitive(Phone) TO SupportUser;
```

FinanceUser -> only Revenue unmasked:
```sql
GRANT UNMASK ON ESG.CompanySensitive(Revenue) TO FinanceUser;
```

Test behavior with SupportUser:
```sql
EXECUTE AS USER = 'SupportUser';

SELECT CompanyName, Email, Phone, Revenue
FROM ESG.CompanySensitive;

REVERT;
```

Test behavior with FinanceUser:
```sql
EXECUTE AS USER = 'FinanceUser';  
 
SELECT CompanyName, Email, Phone, Revenue
FROM ESG.CompanySensitive;

REVERT;
```

#### Remove Masking

```sql
ALTER TABLE ESG.CompanySensitive
ALTER COLUMN Email DROP MASKED;

ALTER TABLE ESG.CompanySensitive
ALTER COLUMN Phone DROP MASKED;

ALTER TABLE ESG.CompanySensitive
ALTER COLUMN CreditRating DROP MASKED;

ALTER TABLE ESG.CompanySensitive
ALTER COLUMN Revenue DROP MASKED;
```

