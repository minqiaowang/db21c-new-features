# SQL Macros

## Introduction

This lab you can learn how to use the SQL Macros  (SQM) to increase developer productivity, simplify collaborative development, and improve code quality.

Estimated Lab Time: n minutes

### About SQL Macros
You can create SQL Macros (SQM) to factor out common SQL expressions and statements into reusable, parameterized constructs that can be used in other SQL statements. SQL macros can either be scalar expressions, typically used in `SELECT` lists, `WHERE`, `GROUP` `BY` and `HAVING` clauses, to encapsulate calculations and business logic or can be table expressions, typically used in a `FROM` clause.

### Objectives

*List objectives for the lab - if this is the intro lab, list objectives for the workshop*

In this lab, you will:
* Objective 1
* Objective 2
* Objective 3

### Prerequisites

The prerequisites for the lab

* Oracle database 20c or up

## **STEP 1**: Prepare the environment

Step 1 opening paragraph.

1. Connect to the PDB1.

  ```
  $ sqlplus / as sysdba         
  
  SQL*Plus: Release 20.0.0.0.0 - Production on Tue Oct 6 06:35:04 2020
  Version 20.3.0.0.0
  
  Copyright (c) 1982, 2020, Oracle.  All rights reserved.
  
  
  Connected to:
  Oracle Database 20c EE Extreme Perf Release 20.0.0.0.0 - Production
  Version 20.3.0.0.0
  
  SQL> show pdbs
  
      CON_ID CON_NAME			  OPEN MODE  RESTRICTED
  ---------- ------------------------------ ---------- ----------
  	 2 PDB$SEED			  READ ONLY  NO
  	 3 PDB1 			  READ WRITE NO
  SQL> alter session set container=pdb1;
  
  Session altered.
  ```

  

2. Create HR schema and sample tables.

    ```
    SQL> @$ORACLE_HOME/demo/schema/human_resources/hr_main.sql WelcomePTS_123# users temp /home/oracle /home/oracle
    
    specify password for HR as parameter 1:
    
    specify default tablespeace for HR as parameter 2:
    
    specify temporary tablespace for HR as parameter 3:
    
    specify log path as parameter 4:
    
    
    PL/SQL procedure successfully completed.
    
    
    User created.
    
    
    User altered.
    
    
    User altered.
    
    ...
    ...
    ...
    
    
    Comment created.
    
    
    Commit complete.
    
    
    PL/SQL procedure successfully completed.
    
    SQL>
    ```

    

3. Enable auto trace.

    ```
    SQL> @$ORACLE_HOME/sqlplus/admin/plustrce.sql;
    SQL> 
    SQL> drop role plustrace;
    drop role plustrace
              *
    ERROR at line 1:
    ORA-01919: role 'PLUSTRACE' does not exist
    
    
    SQL> create role plustrace;
    
    Role created.
    
    SQL> 
    SQL> grant select on v_$sesstat to plustrace;
    
    Grant succeeded.
    
    SQL> grant select on v_$statname to plustrace;
    
    Grant succeeded.
    
    SQL> grant select on v_$mystat to plustrace;
    
    Grant succeeded.
    
    SQL> grant plustrace to dba with admin option;
    
    Grant succeeded.
    
    SQL> 
    ```

    

4. Grant *plustrace* role to HR user.

    ```
    SQL> grant plustrace to hr;
    
    Grant succeeded.
    
    SQL> exit
    Disconnected from Oracle Database 20c EE Extreme Perf Release 20.0.0.0.0 - Production
    Version 20.3.0.0.0
    [oracle@db20c ~]$ 
    ```

    

5. Connect to the HR user.

    ```
    [oracle@db20c ~]$ sqlplus hr/WelcomePTS_123#@localhost:1521/pdb1.sub11160748350.myvcnseoul.oraclevcn.com
    
    SQL*Plus: Release 20.0.0.0.0 - Production on Tue Oct 6 06:48:33 2020
    Version 20.3.0.0.0
    
    Copyright (c) 1982, 2020, Oracle.  All rights reserved.
    
    
    Connected to:
    Oracle Database 20c EE Extreme Perf Release 20.0.0.0.0 - Production
    Version 20.3.0.0.0
    
    SQL> select tname from tab;                    
    
    TNAME
    --------------------------------------------------------------------------------
    REGIONS
    COUNTRIES
    LOCATIONS
    DEPARTMENTS
    JOBS
    EMPLOYEES
    JOB_HISTORY
    EMP_DETAILS_VIEW
    
    8 rows selected.
    
    SQL>  
    ```

    

6. E

## **STEP 2:** Create the SQM as a scalar expression.

1. Create a function using the SQL_MACRO(SCALAR) keyword.

  ```
  SQL> <copy>create or replace function m_full_name(p_first_name varchar2,p_last_name varchar2)
  return varchar2 SQL_MACRO(SCALAR)
  as
  begin
    return q'[p_first_name||' '||p_last_name]';
  end;
  /</copy>  2    3    4    5    6    7  
  
  Function created.
  
  SQL> 
  ```

  

2. Use the SQM to query the table and display the full employees names.

    ```
    SQL> select m_full_name(first_name,last_name) from employees where rownum<10;
    
    FULL_NAME(FIRST_NAME,LAST_NAME)
    ----------------------------------------------
    Ellen Abel
    Sundar Ande
    Mozhe Atkinson
    David Austin
    Hermann Baer
    Shelli Baida
    Amit Banda
    Elizabeth Bates
    Sarah Bell
    
    9 rows selected.
    
    SQL> 
    ```

    

3. Compare the normal function.

    ```
    SQL> <copy>create or replace function f_full_name(p_first_name varchar2,p_last_name varchar2)
    return varchar2 as
    begin
      return p_first_name||' '||p_last_name;
    end;
    /</copy>  2    3    4    5    6  
    
    Function created.
    
    SQL> 
    ```

4. Using the normal function to display the full employees names.

  ```
  SQL> select f_full_name(first_name,last_name) from employees where rownum<10;
  
  F_FULL_NAME(FIRST_NAME,LAST_NAME)
  --------------------------------------------------------------------------------
  Ellen Abel
  Sundar Ande
  Mozhe Atkinson
  David Austin
  Hermann Baer
  Shelli Baida
  Amit Banda
  Elizabeth Bates
  Sarah Bell
  
  9 rows selected.
  
  SQL> 
  ```

  You can see the result is the same.

5. Create another SQL Macros which calculate the commission of the employees.

    ```
    SQL> create or replace function m_commission(p_salary number,p_commission_pct number)
    return varchar2 SQL_MACRO(SCALAR)
    as
    begin
      if p_commission_pct is null
      then
      	return	'p_salary*0.1';
      else
        return 'p_salary*p_commission_pct';
       end if;
    end;
    /  2    3    4    5    6    7    8    9   10   11   12  
    
    Function created.
    
    SQL> 
    ```

    

6. Query the commission of an employee. This time we enable the auto trace.

    ```
    SQL> set linesize 120
    SQL> set autotrace on
    SQL> select m_full_name(first_name,last_name) Employee, m_commission(salary,commission_pct) Commission from employees
    where m_full_name(first_name,last_name) like 'Steven King';  2  
    
    EMPLOYEE				       COMMISSION
    ---------------------------------------------- ----------
    Steven King					     2400
    
    
    Execution Plan
    ----------------------------------------------------------
    Plan hash value: 1445457117
    
    -------------------------------------------------------------------------------
    | Id  | Operation	  | Name      | Rows  | Bytes | Cost (%CPU)| Time     |
    -------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT  |	      |     1 |    19 |     3	(0)| 00:00:01 |
    |*  1 |  TABLE ACCESS FULL| EMPLOYEES |     1 |    19 |     3	(0)| 00:00:01 |
    -------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
       1 - filter("FIRST_NAME"||' '||"LAST_NAME"='Steven King')
    
    
    Statistics
    ----------------------------------------------------------
    	 62  recursive calls
    	  0  db block gets
    	189  consistent gets
    	  0  physical reads
    	  0  redo size
    	660  bytes sent via SQL*Net to client
    	533  bytes received via SQL*Net from client
    	  2  SQL*Net roundtrips to/from client
    	  0  sorts (memory)
    	  0  sorts (disk)
    	  1  rows processed
    
    SQL> 
    ```

    

7. Create a normal function to calculate the commission.

    ```
    SQL> create or replace function f_commission(p_salary number,p_commission_pct number)
    return number
    as
    begin
      if p_commission_pct is null
      then
      	return	p_salary*0.1;
      else
        return p_salary*p_commission_pct;
       end if;
    end;
    /  2    3    4    5    6    7    8    9   10   11   12  
    
    Function created.
    
    SQL> 
    ```

    

8. Query the commission using the normal function.

    ```
    SQL> set linesize 120
    SQL> set autotrace on
    SQL> select f_full_name(first_name,last_name) Employee, f_commission(salary,commission_pct) Commission from employees
    where f_full_name(first_name,last_name) like 'Steven King';  2  
    
    EMPLOYEE
    ------------------------------------------------------------------------------------------------------------------------
    COMMISSION
    ----------
    Steven King
          2400
    
    
    
    Execution Plan
    ----------------------------------------------------------
    Plan hash value: 1445457117
    
    -------------------------------------------------------------------------------
    | Id  | Operation	  | Name      | Rows  | Bytes | Cost (%CPU)| Time     |
    -------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT  |	      |     1 |    21 |     3	(0)| 00:00:01 |
    |*  1 |  TABLE ACCESS FULL| EMPLOYEES |     1 |    21 |     3	(0)| 00:00:01 |
    -------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
       1 - filter("F_FULL_NAME"("FIRST_NAME","LAST_NAME")='Steven King')
    
    
    Statistics
    ----------------------------------------------------------
    	 47  recursive calls
    	  0  db block gets
    	 13  consistent gets
    	  0  physical reads
    	  0  redo size
    	660  bytes sent via SQL*Net to client
    	533  bytes received via SQL*Net from client
    	  2  SQL*Net roundtrips to/from client
    	  0  sorts (memory)
    	  0  sorts (disk)
    	  1  rows processed
    
    SQL> 
    ```

    Compare the Predicate Information with the SQL Macros, What's the different you found?

9. Now, we can create an index on employees table.

    ```
    SQL> create index emp_full_name on employees(first_name||' '||last_name);
    
    Index created.
    ```

    

10. Run with the SQM again.

    ```
    SQL> select m_full_name(first_name,last_name) Employee, m_commission(salary,commission_pct) Commission from employees
    where m_full_name(first_name,last_name) like 'Steven King';  2  
    
    EMPLOYEE				       COMMISSION
    ---------------------------------------------- ----------
    Steven King					     2400
    
    
    Execution Plan
    ----------------------------------------------------------
    Plan hash value: 1578404883
    
    -----------------------------------------------------------------------------------------------------
    | Id  | Operation			    | Name	    | Rows  | Bytes | Cost (%CPU)| Time     |
    -----------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT		    |		    |	  1 |	 44 |	  2   (0)| 00:00:01 |
    |   1 |  TABLE ACCESS BY INDEX ROWID BATCHED| EMPLOYEES     |	  1 |	 44 |	  2   (0)| 00:00:01 |
    |*  2 |   INDEX RANGE SCAN		    | EMP_FULL_NAME |	  1 |	    |	  1   (0)| 00:00:01 |
    -----------------------------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
       2 - access("FIRST_NAME"||' '||"LAST_NAME"='Steven King')
    
    
    Statistics
    ----------------------------------------------------------
    	 67  recursive calls
    	  0  db block gets
    	187  consistent gets
    	  0  physical reads
    	  0  redo size
    	660  bytes sent via SQL*Net to client
    	754  bytes received via SQL*Net from client
    	  2  SQL*Net roundtrips to/from client
    	  0  sorts (memory)
    	  0  sorts (disk)
    	  1  rows processed
    
    SQL> 
    ```

    

11. Run with the normal function again. Compere the Execution Plan with SQM.

     ```
     SQL> select f_full_name(first_name,last_name) Employee, f_commission(salary,commission_pct) Commission from employees
     where f_full_name(first_name,last_name) like 'Steven King';  2  
     
     EMPLOYEE
     ------------------------------------------------------------------------------------------------------------------------
     COMMISSION
     ----------
     Steven King
           2400
     
     
     
     Execution Plan
     ----------------------------------------------------------
     Plan hash value: 1445457117
     
     -------------------------------------------------------------------------------
     | Id  | Operation	  | Name      | Rows  | Bytes | Cost (%CPU)| Time     |
     -------------------------------------------------------------------------------
     |   0 | SELECT STATEMENT  |	      |     1 |    21 |     3	(0)| 00:00:01 |
     |*  1 |  TABLE ACCESS FULL| EMPLOYEES |     1 |    21 |     3	(0)| 00:00:01 |
     -------------------------------------------------------------------------------
     
     Predicate Information (identified by operation id):
     ---------------------------------------------------
     
        1 - filter("F_FULL_NAME"("FIRST_NAME","LAST_NAME")='Steven King')
     
     
     Statistics
     ----------------------------------------------------------
     	 47  recursive calls
     	  0  db block gets
     	 13  consistent gets
     	  0  physical reads
     	  0  redo size
     	660  bytes sent via SQL*Net to client
     	533  bytes received via SQL*Net from client
     	  2  SQL*Net roundtrips to/from client
     	  0  sorts (memory)
     	  0  sorts (disk)
     	  1  rows processed
     
     SQL> 
     ```

     It's can not use the index and still full table scan.

12. sadf

## **Step 3:**  Use a SQM as a table expression

1. Create a SQM to calculate the budget of a department. If `SCALAR` or `TABLE` is not specified, `TABLE` is the default.

   ```
   SQL> CREATE OR REPLACE FUNCTION budget
   return varchar2 SQL_MACRO
   IS
   BEGIN
     RETURN q'( select department_id, sum(salary) budget
                from hr.employees 
                group by department_id )';
   END;
   /  2    3    4    5    6    7    8    9  
   
   Function created.
   
   SQL> 
   ```

   

2. Use the SQM to display the result for the departments 10 and 50.

   ```
   SQL> SELECT * FROM budget() WHERE department_id IN (10,50);
   
   DEPARTMENT_ID	  BUDGET
   ------------- ----------
   	   50	  156400
   	   10	    4400
   
   SQL> 
   ```

   

3. Compare with a simple view. First, create a simple view to display the sum of the salaries per department.

   ```
   SQL> CREATE VIEW v_budget 
    AS SELECT department_id, sum(salary) v_budget 
       FROM hr.employees
       GROUP BY department_id;  2    3    4  
   
   View created.
   
   SQL> 
   ```

   

4. Query the result from the view.

   ```
   SQL> SELECT * FROM v_budget WHERE department_id IN (10,50);
   
   DEPARTMENT_ID	V_BUDGET
   ------------- ----------
   	   50	  156400
   	   10	    4400
   
   SQL> 
   ```

   

5. Use an SQM as a table expression to display sum of the salaries per department for a particular job.

   ```
   SQL> CREATE OR REPLACE FUNCTION budget_per_job(job_id varchar2)
   return varchar2 SQL_MACRO
   IS
   BEGIN
     RETURN q'( select department_id, sum(salary) budget
                from hr.employees 
                where job_id = budget_per_job.job_id
                group by department_id )';
   END;
   /  2    3    4    5    6    7    8    9   10  
   
   Function created.
   
   SQL>
   ```

   

6. Use the SQM to display the result for the `SH_CLERK` job in department 50.

   ```
   SQL> set autotrace on
   SQL> SELECT * FROM budget_per_job('SH_CLERK') WHERE department_id=50;
   
   DEPARTMENT_ID	  BUDGET
   ------------- ----------
   	   50	   64300
   
   
   Execution Plan
   ----------------------------------------------------------
   Plan hash value: 1116566935
   
   -------------------------------------------------------------------------------------------
   | Id  | Operation		     | Name	  | Rows  | Bytes | Cost (%CPU)| Time	  |
   -------------------------------------------------------------------------------------------
   |   0 | SELECT STATEMENT	     |		  |	1 |    16 |	3   (0)| 00:00:01 |
   |   1 |  SORT GROUP BY NOSORT	     |		  |	1 |    16 |	3   (0)| 00:00:01 |
   |*  2 |   TABLE ACCESS BY INDEX ROWID| EMPLOYEES  |	8 |   128 |	3   (0)| 00:00:01 |
   |*  3 |    INDEX RANGE SCAN	     | EMP_JOB_IX |    20 |	  |	1   (0)| 00:00:01 |
   -------------------------------------------------------------------------------------------
   
   Predicate Information (identified by operation id):
   ---------------------------------------------------
   
      2 - filter("DEPARTMENT_ID"=50)
      3 - access("JOB_ID"='SH_CLERK')
   
   
   Statistics
   ----------------------------------------------------------
   	  0  recursive calls
   	  0  db block gets
   	  3  consistent gets
   	  0  physical reads
   	  0  redo size
   	653  bytes sent via SQL*Net to client
   	425  bytes received via SQL*Net from client
   	  2  SQL*Net roundtrips to/from client
   	  0  sorts (memory)
   	  0  sorts (disk)
   	  1  rows processed
   
   SQL> 
   ```

   

7. asdf

8. sadf

## **Step 4:** Display the new data dictionary

1. Use the `USER_PROCEDURES` view to display the new values of the `SQL_MACRO` column.

   ```
   SQL> COL object_name FORMAT A30
   SQL> SELECT object_name, sql_macro, object_type FROM user_procedures;
   
   OBJECT_NAME		       SQL_MA OBJECT_TYPE
   ------------------------------ ------ -------------
   SECURE_DML		       NULL   PROCEDURE
   ADD_JOB_HISTORY 	       NULL   PROCEDURE
   M_COMMISSION		       SCALAR FUNCTION
   F_FULL_NAME		       NULL   FUNCTION
   M_FULL_NAME		       SCALAR FUNCTION
   F_COMMISSION		       NULL   FUNCTION
   BUDGET			       TABLE  FUNCTION
   SECURE_EMPLOYEES		      TRIGGER
   UPDATE_JOB_HISTORY		      TRIGGER
   
   9 rows selected.
   
   SQL> 
   ```

   

2. asdf

3. adsf




You may proceed to the next lab.

## Learn More

*(optional - include links to docs, white papers, blogs, etc)*

* [URL text 1](http://docs.oracle.com)
* [URL text 2](http://docs.oracle.com)

## Acknowledgements
* **Author** - <Name, Title, Group>
* **Contributors** -  <Name, Group> -- optional
* **Last Updated By/Date** - <Name, Group, Month Year>
* **Workshop (or Lab) Expiry Date** - <Month Year> -- optional, use this when you are using a Pre-Authorized Request (PAR) URL to an object in Oracle Object Store.

## See an issue?
Please submit feedback using this [form](https://apexapps.oracle.com/pls/apex/f?p=133:1:::::P1_FEEDBACK:1). Please include the *workshop name*, *lab* and *step* in your request.  If you don't see the workshop name listed, please enter it manually. If you would like us to follow up with you, enter your email in the *Feedback Comments* section.
