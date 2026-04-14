### Build ESG APIs using Data API Builder (DAB)

#### Scenario

You are building APIs for CarbonOps.

Your frontend team needs:

1) REST APIs for dashboards
2) GraphQL APIs for analytics

Goal: Expose ESG data via Data API Builder (DAB)

#### Install DAB CLI

Install the Data API Builder CLI using .NET
```bash
dotnet tool install --global Microsoft.DataApiBuilder
```

Verify the Installation
```bash
dab --version
```

#### Create the Project

```bash
mkdir carbonops-esg-api
cd carbonops-esg-api
```

#### Initialize Config

```bash
dab init --database-type mssql \
--connection-string "@env('DATABASE_CONNECTION_STRING')" \
--host-mode development
```

#### Add ESG Entities

Add companies:
```bash
dab add Company --source ESG.Companies --permissions "anonymous:read"
```

Add EmissionRecords:
```bash
dab add EmissionRecord --source ESG.EmissionRecords --permissions "anonymous:read"
```

Add EnergyConsumption:
```bash
dab add EnergyConsumption --source ESG.EnergyConsumption --permissions "anonymous:read"
```

Add SustainabilityReports:
```bash
dab add SustainabilityReport --source ESG.SustainabilityReports --permissions "anonymous:read"
```

#### Configure Field Mappings and Relationships

EmissionRecord -> Company (many-to-one)
```json
"relationships": {
  "company": {
    "cardinality": "one",
    "target.entity": "Company",
    "source.fields": ["CompanyID"],
    "target.fields": ["CompanyID"]
  }
}
```

Company -> EmissionRecord (one-to-many)
```json
"relationships": {
  "emissions": {
    "cardinality": "many",
    "target.entity": "EmissionRecord",
    "source.fields": ["CompanyID"],
    "target.fields": ["CompanyID"]
  }
}
```

Field Mappings for EmissionRecord:
```json
"mappings": {
  "EmissionID": "id",
  "CompanyID": "companyId",
  "EmissionDate": "date",
  "CO2_Emissions": "co2",
  "Scope": "scope"
}
```

#### Run DAB

Set connection string in environment variable
```bash
export DATABASE_CONNECTION_STRING="Server=your-server.database.windows.net;Database=ProductCatalog;User Id=your-user;Password=your-password;TrustServerCertificate=true"
```

Start the API:
```bash
dab start
```

#### Try some BasicREST API Queries

get all companies:
```bash
GET http://localhost:5000/api/Company
```

Get all emission records:
```bash
GET http://localhost:5000/api/EmissionRecord
```

Filter by CompanyID:
```bash
GET http://localhost:5000/api/EmissionRecord?$filter=companyId eq 1
```

Filter high emissions:
```bash
GET http://localhost:5000/api/EmissionRecord?$filter=co2 gt 500
```

Combine filters:
```bash
GET http://localhost:5000/api/EmissionRecord?$filter=companyId eq 1 and co2 gt 300
```

Projection - select specific fields:
```bash
GET http://localhost:5000/api/EmissionRecord?$select=companyId,co2,date
```

Sort by emissions descending:
```bash
GET http://localhost:5000/api/EmissionRecord?$orderby=co2 desc
```

Pagination (first 5 records):
```bash
GET http://localhost:5000/api/EmissionRecord?$top=5
```



#### Try some GraphQL Queries

Use the base URL for GraphQL:
```bash
http://localhost:5000/graphql
```

Get companies with their emissions:
```graphql
query {
  companies {
    items {
      CompanyName
      emissions {
        items {
          CO2_Emissions
          EmissionDate
        }
      }
    }
  }
}
```

Filter emissions by company:
```graphql
query {
  emissionRecords(filter: { CompanyID: { eq: 1 } }) {
    items {
      CO2_Emissions
      Scope
    }
  }
}
```

Traverse relationships:
```graphql
query {
  companies {
    items {
      CompanyName
      emissions {
        items {
          CO2_Emissions
        }
      }
    }
  }
}
```
