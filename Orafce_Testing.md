
    #!div class="important" style="border: 2pt solid; text-align: center" 
    '''''DEPRECATED, NO LONGER USED''''' 
# Orafce Testing

## Overview




The Motive of the testing is to make sure that Orafce "Oracle Function " features is compatible with Postgres and also used by spacewalk.

The minimum supported version of Postgres is PG 8.1+.

Targeted version of Orafce is 3.0.0.
### About Orafce Features



Orafce: Oracle support functions http://www.postgres.cz/index.php/Oracle_functionality_(en))
### Oracle Compatible Function's of Orafce being used in Spacewalk





| *Function Name* | *Remarks* |  |
| --- | --- |
| DBMS_OUTPUT |  To see the data we must have to enable the serveroutput on also we need to Use select statement before DBMS_OUTPUT,we can also use in RETURN statement of Plpgsql fuctions  |
| NVL |  It is almost Compatible with oracle but  it give  Error due to its PG limitation for ex:-`select nvl('a','b') from dual;ERROR:  could not determine anyarray/anyelement type because input has type "unknown"`.Comments which i have got from Itagaki: "Yeah, PostgreSQL 8.2 has a limitation in treating anyelement.Please use 8.3 or add explicit type casts".so this is applicable for 8.1 too  |
| NVL2 |  same commets which has given for NVL  |
| ROUND |  spacewalk is using this function like RETURN and IF statement  |
| TRUNC |  spacwalk is using this function for passing to variable i.e target_date := trunc(add_months(sysdate,-months_ago)-(7*weeks_ago)) |
| SUBSTR |  It is better to use oracle.substr instead of substr in PG because by default it will search in pg_catalog.substr which is not oracle compatible  |
| INSTR |  It is using in IF,inside Substr function  |
| DECODE |  Same comments which has given for NVL2  |
| LENGTH |  it is commanly using in DBMS_OUTPUT,SUBSTR,IF,FOR Loop |
| LAST_DAY | using this function for passing to variable i.e target_date := last_day(target_date) |
| ADD_MONTHS | one of the sql statment using by spacewalk is add_months(sysdate,-5)-(7*5) , if we changed sysdate to current_timestamp and try to run this query, it will throw an error saying:-ERROR:  function add_months(timestamp with time zone, integer) does not exist,to avoid this error ,either we can use explicit type casting OR instead of using Current_timestamp ,we can use current_date. |
| BITAND |  Being used in the function adler32  |
### Oracle Compatible Function's of Orafce *NOT* being used in Spacewalk




||*Function Name*||
||MONTHS_BETWEEN|| 
||NEXT_DAY|| 
||DBMS_ALERT|| 
||DBMS_PIPE|| 
||UTL_FILE|| 
||RESERVE|| 
||CONCAT|| 
||SINH||	
||COSH||
||TANH  	

| NANVL |
| --- |
### Tracking



*About the Bugs and Issue's raised so far ...*

Reported in January http://lists.pgfoundry.org/pipermail/orafce-general/2009-January/thread.html.

Reported in Feburary http://lists.pgfoundry.org/pipermail/orafce-general/2009-February/thread.html.
### Test Cases



The Test cases file has been uploaded in this link ,the sql file name is "orafce_testcases_version0.1.sql"
### Limitation



1.Oracle grouping function is not supported by Orafce.


    ./schema/util/scripts/hwreport.sql:--    DECODE(GROUPING(model), 1, 'All Models', model) "CPU Model",
    ./schema/util/scripts/hwreport.sql:--    DECODE(GROUPING(rhnCpuSpeedRangeFunc(mhz)), 1, 'All Speeds',

*NOTE:*

{{{ 
It is Commented so we should not worry for this.
}}}

2.In Most of the function ,we have to use  explicit type casting  because PostgreSQL 8.2/8.1 has a limitation in treating anyelement-

{{{  
Example:-select decode(1,1,null::int)
}}} 

 for more detail pls review  test-case file(orafce_testcases_version0.1.sql)
### Sql File's Location



[Sql File Location for Oracle(Supported by Orafce) Built-in functions using by spacewalk](Sql_Files_Location)
### Conclusion



* This is all about Orafce Feature's apart from this, there are other Single-row functions which are Using by spacewalk which has been taken care in other wiki page.

* Grouping is not a part of  Orafce Feature but still mentioning here .

* Orafce 3.0.0 testing done aganist PG8.1 . currently one has to update the orafce 3.0.0 with CVS to run against PG8.1



