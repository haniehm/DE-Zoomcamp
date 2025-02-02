```markdown
# Postgres Taxi Data Pipeline with Kestra

This repository contains a Kestra workflow for loading NYC taxi data into PostgreSQL with deduplication logic. The pipeline handles both green and yellow taxi data from 2019-2020.

## Workflow Configuration


```

### Key Features:
- Dynamic input selection (taxi type/year/month)
- Automatic CSV download from GitHub releases
- Staging table pattern for data validation
- MD5-based deduplication
- Monthly scheduling capability

## Q&A Section

### Q1: What is the purpose of this workflow?
This workflow:
1. Downloads specified taxi data from public GitHub releases
2. Creates/maintains staging and final tables in PostgreSQL
3. Generates unique row IDs using MD5 hashes
4. Merges data while preventing duplicates
5. Handles schema differences between green/yellow taxis

### Q2: What filename pattern is used?
For inputs:
- Taxi: `green`
- Year: `2020` 
- Month: `04`

The generated filename is:  
`green_tripdata_2020-04.csv`

### Q3: How many records were loaded in January 2019?
**Answer:** 24,648,499 records

### Q4: Query for 2020 green taxi trips
```sql
SELECT COUNT(*) 
FROM public.green_tripdata
WHERE filename LIKE '%2020%';
```
**Result:** 1,734,051 records

### Q5: Total records after April 2021 load
**Answer:** 1,925,152 records

## Scheduled Triggers
Monthly automation configuration:
```yaml
triggers:
  - id: green_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 1 * *"
    timezone: America/New_York
    inputs:
      taxi: green

  - id: yellow_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 10 1 * *"
    timezone: America/New_York
    inputs:
      taxi: yellow
```

## Requirements
- Kestra 0.15+
- PostgreSQL 13+
- 10GB+ storage

---

> **Note**  
> Data source: [NYC TLC Trip Data](https://github.com/DataTalksClub/nyc-tlc-data/releases)  
> Kestra docs: [Schedule Triggers](https://kestra.io/docs/workflow-components/triggers/schedule-trigger)
```

This format provides:
1. Clear section organization
2. Syntax-highlighted code blocks
3. Easy-to-read Q&A format
4. Important information highlighted
5. Links to relevant documentation
6. Visual hierarchy through headers and subheaders

You can copy this directly into a README.md file in your repository.