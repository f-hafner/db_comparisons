# db_comparisons
Compare sqlite and DuckDB for aggregation queries.

Related:
- https://github.com/f-hafner/mag_sample/issues/39
- https://github.com/NLeSC/guide/issues/316

### Steps 
1. Copy from sqlite to duckdb. Follow `create_duckdb_from_mag.md`. Need access to current database.
2. Create sqlite copy from the duckdb file created in step 1 (for exact comparison). Create indexes. TODO 
3. Compare queries: directly in Duck, directly in sqlite, and querying sqlite with duck. Compare timing; make sure results are the same. TODO


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




- if sqlite does not define types, duckdb cannot convert them
	-
	  ```sql
	   create temp table mytest  as select * from sqlite_scan("/mnt/ssd/AcademicGraph/AcademicGraph.sqlite", "author_sample");
	  ```
	- raises `Error: Conversion Error: Unimplemented type for cast (BLOB -> INTEGER)`
	- schema of `author_sample` in sqlite:
	-
	  ```sql
	  CREATE TABLE author_sample(
	    AuthorId INT,
	    YearLastPub,
	    YearFirstPub,
	    PaperCount,
	    FirstName
	  );
	  CREATE UNIQUE INDEX idx_as_AuthorId ON author_sample (AuthorId ASC) ;
	  CREATE INDEX idx_as_FirstName ON author_sample (FirstName ASC);
	  ```
		- this means when building the sqlite table, types need to be enforced with
		-
		  ```sql
		  create table flavio_test (AuthorId INT, YearLastPub INT, YearFirstPub INT, PaperCount INT, FirstName VARCHAR);
		  ```



## Thoughts and lessons learned 

- "static input data" != static database. 
    - what seems to matter is whether input data is static; I did not understand that correctly until today
- sqlite still has advantages that may be overlooked by these comparisons
    - for instance, the unique constraint on indexes is quick check that data integrity is not violated; a similar mechanism should always be used even when using duckdb
- I think Duck parallelizes across CPUs. So the gains may differ by by laptop vs cluster/server
    - are the gains proportional to the CPUs available?
    - compare both on a cluster and on a laptop

