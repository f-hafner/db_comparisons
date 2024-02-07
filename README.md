# db_comparisons
Compare sqlite and DuckDB for aggregation queries

### Steps 
1. Copy from sqlite to duckdb. Follow `create_duckdb_from_mag.md`. Need access to current database.
2. Create sqlite copy from the duckdb file created in step 1 (for exact comparison). Create indexes. TODO 
3. Compare queries: directly in Duck, directly in sqlite, and querying sqlite with duck. TODO


### Queries to compare 

Citation count 
```sql
create temp table temp_citations as 
SELECT  a.PaperReferenceId, 
            b.Year, 
            b.DocType AS ReferencingDocType, 
            COUNT(DISTINCT a.PaperId) AS CitationCount 
FROM PaperReferences a 
INNER JOIN (
    SELECT PaperId, Year, DocType 
    FROM Papers 
    INNER JOIN (
        SELECT PaperId 
        FROM PaperMainFieldsOfStudy
    ) USING (PaperId)
    WHERE 
    DocType IN ('Journal', 'Conference', 'Book', 'BookChapter', 'Thesis') 
) b on a.PaperId = b.PaperId 
INNER JOIN (
    SELECT PaperId, Year, DocType 
    FROM Papers 
    INNER JOIN ( 
        SELECT PaperId 
        FROM PaperMainFieldsOfStudy
    ) USING (PaperId) 
    WHERE 
    DocType IN ('Journal', 'Conference', 'Book', 'BookChapter', 'Thesis') 
    AND 
    Year >= 1950
) c on a.PaperReferenceId = c.PaperId 
GROUP BY a.PaperReferenceId, b.Year, ReferencingDocType

```


Highest N papers by author-affiliation-year 
```sql
create temp table author_affil_year as 
SELECT AuthorId, AffiliationId, Year
FROM (
    SELECT *,
            MAX(PaperCount) OVER(PARTITION BY AuthorId, Year) AS MaxPaperCount
    FROM (
        SELECT a.AuthorId, 
                a.AffiliationId, 
                c.Year, 
                count(PaperId) AS PaperCount
        FROM PaperAuthorAffiliations a
        INNER JOIN (
            SELECT AuthorId 
            FROM author_sample 
        ) b USING (AuthorId)
        INNER JOIN (
            SELECT PaperId, Year 
            FROM Papers
            WHERE DocType IN ('Journal', 'Conference')
        ) c USING (PaperId)
        GROUP BY a.AuthorId, a.AffiliationId, c.Year 
    ) 
)   
WHERE PaperCount = MaxPaperCount ;  
```