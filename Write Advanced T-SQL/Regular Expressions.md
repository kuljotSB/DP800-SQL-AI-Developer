### Pattern Matching with Regular Expressions in SQL

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform receives messy real-world data:

1) Emails from different domains
2) Emission codes in inconsistent formats
3) Free-text notes from auditors
4) Tags stored as comma-separated values

You must use regular expressions to:

1) Validate data
2) Clean data
3) Extract insights
4) Transform text

#### Prepare Sample Data:

Add Messy Columns:
```sql
ALTER TABLE ESG.Companies
ADD Email NVARCHAR(200),
    Phone NVARCHAR(50),
    Tags NVARCHAR(200);
```

Insert messy data:
```sql
UPDATE ESG.Companies
SET 
    Email = CASE CompanyID
        WHEN 1 THEN 'contact@greensteel.com'
        WHEN 2 THEN 'support@ecologistics'
        WHEN 3 THEN 'info@solargrid.energy'
        WHEN 4 THEN 'sales@urbanretail.co.uk'
        WHEN 5 THEN 'hello@aerofly@aviation.com'
        WHEN 6 THEN 'contact.blueocean.com'
        WHEN 7 THEN 'greenfoods@gmail.com'
        WHEN 8 THEN 'futuretech.ai@corp'
    END,

    Phone = CASE CompanyID
        WHEN 1 THEN '+91-98765 43210'
        WHEN 2 THEN '9876543210'
        WHEN 3 THEN '(123) 456-7890'
        WHEN 4 THEN '123.456.7890'
        WHEN 5 THEN '123-4567-890'
        WHEN 6 THEN 'phone: 1234567890'
        WHEN 7 THEN '+1 (800) 555-1234'
        WHEN 8 THEN 'N/A'
    END,

    Tags = CASE CompanyID
        WHEN 1 THEN 'esg, steel, carbon'
        WHEN 2 THEN 'logistics;transport;ai'
        WHEN 3 THEN 'solar|energy|renewable'
        WHEN 4 THEN 'retail, sales , customer'
        WHEN 5 THEN 'aviation,AI,flight'
        WHEN 6 THEN 'shipping , ocean,trade'
        WHEN 7 THEN 'food, organic ,health'
        WHEN 8 THEN 'ai,tech, innovation'
    END;
```

#### Validate Emails using Regular Expressions

Get valid emails:
```sql
SELECT CompanyName, Email
FROM ESG.Companies
WHERE REGEXP_LIKE(Email, '^[\w\.-]+@[\w\.-]+\.\w+$');
```

Get invalid emails:
```sql
SELECT CompanyName, Email
FROM ESG.Companies
WHERE NOT REGEXP_LIKE(Email, '^[\w\.-]+@[\w\.-]+\.\w+$');
```

#### Extract Email Domain

```sql
SELECT 
    Email,
    REGEXP_SUBSTR(Email, '@(.+)$', 1, 1, '', 1) AS Domain
FROM ESG.Companies;
```

#### Clean Phone Numbers

Remove all non-digits:
```sql
SELECT 
    Phone AS Original,
    REGEXP_REPLACE(Phone, '[^\d]', '') AS CleanPhone
FROM ESG.Companies;
```
