Steps to create the duckdb file from the existing sqlite database



In DuckDB

```sql
CREATE TABLE PaperAuthorAffiliations(PaperId BIGINT, AuthorId BIGINT, AffiliationId BIGINT);
CREATE TABLE Papers(PaperId BIGINT, DocType VARCHAR, Year INTEGER);
CREATE TABLE author_sample(AuthorId BIGINT, YearLastPub INTEGER, YearFirstPub INTEGER, PaperCount INTEGER, FirstName VARCHAR);

install sqlite;
load sqlite;
```

### Table `author_sample`

In sqlite 

```sql
create table flavio_test (AuthorId INT, YearLastPub INT, YearFirstPub INT, PaperCount INT, FirstName VARCHAR);
insert into flavio_test
select * from author_sample;
```

In DuckDB

```sql
insert into author_sample 
select * from sqlite_scan("/mnt/ssd/AcademicGraph/AcademicGraph.sqlite", "flavio_test");

```


### Table `Papers`

In sqlite 
```sql
drop table flavio_test;
create table flavio_test(PaperId INT, DocType TEXT, Year INT);
insert into flavio_test 
select PaperId, DocType, Year from Papers 
where DocType != "" and Year is not NULL and Year != "";
```

In DuckDB
```sql
insert into Papers
select * from sqlite_scan("/mnt/ssd/AcademicGraph/AcademicGraph.sqlite", "flavio_test");
```


### Table `PaperAuthorAffiliations`

In sqlite 
```sql
drop table flavio_test;
create table flavio_test(PaperId INT, AuthorId INT, AffiliationId INT);
insert into flavio_test 
select PaperId, AuthorId, AffiliationId
from PaperAuthorAffiliations
where AffiliationId != "";

```

In DuckDB
```sql
insert into PaperAuthorAffiliations
select * from sqlite_scan("/mnt/ssd/AcademicGraph/AcademicGraph.sqlite", "flavio_test");
```


### Table `PaperReferences`
Types match, can copy directly


DuckDB

```sql
create table PaperReferences as select * from sqlite_scan("/mnt/ssd/AcademicGraph/AcademicGraph.sqlite", "PaperReferences");
```

### Table `PaperMainFieldsOfStudy`
Types match, can copy directly into Duck. 

In DuckDB
```sql
create table PaperMainFieldsOfStudy as select * from sqlite_scan("/mnt/ssd/AcademicGraph/AcademicGraph.sqlite", "PaperMainFieldsOfStudy");
```




