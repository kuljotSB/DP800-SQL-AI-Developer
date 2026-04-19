### Lab: Creating and using Copilot Instruction Files (SQL Project)

#### Setup Repository Structure

create this structure in your VSCode workspace:
```
your-project/
│
├── .github/
│   ├── copilot-instructions.md
│   └── prompts/
│       └── create-index.prompt.md
```

#### Create a Project-Wide Instruction File

the following instructions need to go in the `.github/copilot-instructions.md` file:
```
# SQL Project Guidelines

## Naming Conventions
- Tables: PascalCase (EmissionRecords)
- Columns: PascalCase (CompanyID, EmissionDate)
- Indexes: IX_TableName_ColumnName

## Query Standards
- Avoid SELECT *
- Always specify schema (ESG.TableName)
- Use proper filtering columns

## Performance Guidelines
- Suggest indexes for frequently filtered columns
- Optimize ORDER BY queries
- Prefer indexed columns in WHERE clause

## Example

Bad:
SELECT * FROM EmissionRecords

Good:
SELECT CompanyID, EmissionDate
FROM ESG.EmissionRecords
WHERE CompanyID = @CompanyID
```

#### Create a Reusable Prompt File

The following prompt should be in `.github/prompts/create-index.prompt.md`:
```
# Create Index for Query

Given a SQL query, suggest the best index.

Query:
{{query}}

Requirements:
- Use naming convention IX_Table_Columns
- Include columns used in WHERE and ORDER BY
- Explain why index is needed
```

#### Test Copilot Behavior

Prompt:
```
Create a query to fetch latest emissions for a company
```

#### Test Index Suggestion Prompt

Use your prompt file
```
#create-index
Query:
SELECT CompanyID, EmissionDate
FROM ESG.EmissionRecords
WHERE CompanyID = 1
ORDER BY EmissionDate DESC;
```