### Window Functions for ESG Analytics

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG analytics team needs:

1) Ranking companies by emissions
2) Running totals over time
3) Trend analysis
4) Comparing current vs previous emissions

You will use window functions to solve all of these.

#### Basic Window Function

Running Total of Emissions:
```sql
SELECT 
    CompanyID,
    EmissionDate,
    CO2_Emissions,
    SUM(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
    ) AS RunningTotal
FROM ESG.EmissionRecords
ORDER BY CompanyID, EmissionDate;
```

#### Ranking Functions

Rank companies by total emissions:
```sql
SELECT 
    e.CompanyID,
    e.CO2_Emissions,
    RANK() OVER (ORDER BY CO2_Emissions DESC) AS RankValue
FROM ESG.EmissionRecords AS e
ORDER BY RankValue;
```

Rank using ROW_NUMBER:
```sql
SELECT 
    CompanyID,
    CO2_Emissions,
    ROW_NUMBER() OVER (ORDER BY CO2_Emissions DESC) AS RowNum
FROM ESG.EmissionRecords;
```

Rank using DENSE_RANK:
```sql
SELECT 
    CompanyID,
    CO2_Emissions,
    DENSE_RANK() OVER (ORDER BY CO2_Emissions DESC) AS DenseRankValue
FROM ESG.EmissionRecords;
```

#### Partitioned Ranking

Rank within each company
```sql
SELECT 
    e.CompanyID,
    c.CompanyName,
    e.EmissionDate,
    e.CO2_Emissions,
    e.Source,
    ROW_NUMBER() OVER (
        PARTITION BY e.CompanyID
        ORDER BY e.CO2_Emissions DESC
    ) AS CompanyRank
FROM ESG.EmissionRecords AS e
INNER JOIN ESG.Companies AS c
ON e.CompanyID = c.CompanyID;
```


#### Moving Averages

3-period moving average of emissions:
```sql
SELECT 
    CompanyID,
    EmissionDate,
    CO2_Emissions,
    AVG(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS MovingAvg
FROM ESG.EmissionRecords;
``` 

#### LAG and LEAD for Trend Analysis

Compare with previous emissions:
```sql
SELECT 
    CompanyID,
    Source,
    EmissionDate,
    CO2_Emissions,
    LAG(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
    ) AS PreviousEmission,
    CO2_Emissions - LAG(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
    ) AS Change
FROM ESG.EmissionRecords;
```

#### LEAD for Future Comparison

```sql
SELECT 
    CompanyID,
    Source,
    EmissionDate,
    CO2_Emissions,
    LEAD(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
    ) AS NextEmission,
    CO2_Emissions - LEAD(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
    ) AS FutureChange
FROM ESG.EmissionRecords;
```


#### FIRST_VALUE and LAST_VALUE

```sql
SELECT 
    CompanyID,
    Source,
    EmissionDate,
    CO2_Emissions,
    FIRST_VALUE(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
    ) AS FirstEmission,
    LAST_VALUE(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS LastEmission,
    CO2_Emissions/FIRST_VALUE(CO2_Emissions) OVER (
        PARTITION BY CompanyID
        ORDER BY EmissionDate
    ) AS EmissionRatio
FROM ESG.EmissionRecords;
```


#### Percentile Analysis

```sql
SELECT 
    CompanyID,
    Source,
    EmissionDate,
    CO2_Emissions,
    PERCENT_RANK() OVER (ORDER BY CO2_Emissions) AS PercentRank,
    CUME_DIST() OVER (ORDER BY CO2_Emissions) AS Distribution
FROM ESG.EmissionRecords;
```