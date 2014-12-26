--- 
title: Duplicate Oracle Constraints
date: "2010-07-06T22:35:00.000-03:00"
author: William Mora
tags: 
- foreign key
- PL/SQL
- DBA
- primary key
- not null
- constraints
- Oracle
- SQL
permalink: /2010/07/duplicate-oracle-constraints.html
---

If you have ever used the Oracle import utility, your database has system generated constraint names (ex: `SYS_C00641321`), and you did not specify `CONSTRAINTS=N` during the import then chances are you have duplicate constraints for pretty much every constraint except for any `PRIMARY KEY`, `FOREIGN KEY` and user generated constraint names.  I recently stumbled upon a database that had performance issues and one of the things I noticed was that almost every single constraint was repeated up to 10 times. Most of the duplicates were `NOT NULL` constraints, which is kind of expected since people tend to create `NOT NULL` constraints anonymously such as the following:  

```sql
CREATE TABLE tab1
(col1  VARCHAR(2) NOT NULL,
col2  VARCHAR(30) NOT NULL);
```

That statement will create 2 `NOT NULL` constraints with a system generated name (ex: `SYS_C00641322` and `SYS_C00641323`). If the schema that owns that table is exported and then imported to another schema, it will create two additional `NOT NULL` constraints since it recreates the table running the exact same command shown above. That could be prevented by having user generated constraint names:  

<!--more-->
```sql
CREATE TABLE tab1
(col1  VARCHAR(2) CONSTRAINT COL1_NN NOT NULL,
col2  VARCHAR(30) CONSTRAINT COL2_NN NOT NULL);
``` 

If the schema that owns that table is exported and then imported to another schema, it does not create duplicate any constraints since the system is forced to specify the constraint name at table creation.  Here is my script to identify the duplicate constraints and then generate a script to drop those unnecessary constraints. The logic is to create two temporary tables: One that lists all the constraints (except for `PRIMARY KEY` and `FOREIGN KEY` constraints) that should be in the database and another table where I put all the necessary information for the constraints that need to be dropped. Once the table `DUMP_CONSTRAINTS` is populated, a script is generated to drop all those constraints. Finally, I dropped the temporary tables that I used: 

```sql
CREATE GLOBAL TEMPORARY TABLE temp_constraints
(table_name       varchar2(30),
constraint_name  varchar2(30),
search_condition varchar2(2000),
CONSTRAINT temp_constraints_pk PRIMARY KEY (table_name, search_condition))
/
CREATE GLOBAL TEMPORARY TABLE dump_constraints
(table_name       varchar2(30),
constraint_name  varchar2(30),
search_condition varchar2(2000))
/
DECLARE
tabName   VARCHAR2(30)    := NULL;
conName   VARCHAR2(30)    := NULL;
seaCond   VARCHAR2(2000)  := NULL;
CURSOR c_cursor IS
   SELECT table_name,constraint_name,search_condition
   FROM user_constraints
   WHERE constraint_type NOT IN ('P','R')
   AND search_condition IS NOT NULL
   ORDER BY table_name;
BEGIN
  dbms_output.enable(null); -- To prevent a buffer overflow
  FOR c IN c_cursor LOOP
  BEGIN
     tabName := c.table_name;
     conName := c.constraint_name;
     seaCond := c.search_condition;
     INSERT INTO temp_constraints values(tabName,conName,seaCond);
     EXCEPTION
     WHEN DUP_VAL_ON_INDEX THEN
      BEGIN
       INSERT INTO dump_constraints values(tabName,conName,seaCond);
 END;
     WHEN OTHERS THEN dbms_output.put_line('Another exception: '||tabName
||' Search condition: '||seaCond);  -- For debugging purposes
   END;
END LOOP;
END;
/
SPOOL 'C:\dump_duplicate_constraints'.sql
SET verify OFF
SET pages 0
SET serveroutput ON
SELECT 'ALTER TABLE '||table_name||' DROP CONSTRAINT '||constraint_name||';'
FROM dump_constraints;
/
SPOOL OFF
SET pages 10000
DROP TABLE temp_constraints CASCADE CONSTRAINTS;
/
DROP TABLE dump_constraints CASCADE CONSTRAINTS;
/
```

So, how to prevent this from ever happening again? You could either: 1) Make sure that an import does not import any constraints (just set `CONSTRAINTS=N` when importing) 2) Make sure you properly name ALL constraints. Do not let the system generate a constraint name for you (like previously mentioned)  I hope this helps anyone out there. Feel free to leave a comment if you have any questions.