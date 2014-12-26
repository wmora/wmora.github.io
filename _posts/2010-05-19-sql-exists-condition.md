--- 
title: "SQL: Exists Condition"
date: "2010-05-19T18:34:00.000-03:00"
author: William Mora
tags: 
- not exists
- exists
- SQL
permalink: /2010/05/sql-exists-condition.html
---

The SQL `EXISTS` condition is a boolean function that returns true if the condition is met. The syntax is pretty simple:

```sql
SELECT * FROM TABLE WHERE EXISTS(_subquery_);
```

Alternatively, you can use `NOT EXISTS(_subquery_)`. An example of the function is the following:

```sql
SELECT utc.table_name    
 FROM user_tab_cols utc     
 WHERE utc.column_name='ACCOUNTID'
 AND NOT EXISTS(SELECT uc.table_name     
   FROM user_constraints uc     
   WHERE uc.table_name=utc.table_name     
   AND uc.r_constraint_name='SYS_C00229824')     
   ORDER BY utc.table_name;
 ```

The query looks for all the tables that contain the column `ACCOUNTID` and from those tables, only get the ones that don't make reference to the constraint `SYS_C00229824`. 