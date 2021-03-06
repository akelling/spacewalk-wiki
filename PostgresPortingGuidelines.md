
For up-to-date porting guidelines, see [[PostgreSQLPortingGuide]] This page is now obsolete.

For up-to-date information about the PostgreSQL port in general, see [[PostgreSQL]].

----
## Open questions relevant to porting efforts

1. Which versions of Oracle are to be supported by Spacewalk?

  Ans: The minimum supported version of Oracle is Oracle 10g. 

2. Which versions of Postgres are to be supported by Spacewalk?
  Ans: Targeted version of PostgreSQL is 8.1+ 
# Generic (query) porting issues



1. Calling code should not use quotes around identifiers.
 This is not a limitation, but just a coding convention note so as to increase portabilty of the application code. If the application should use quotes, it should do so all across the code.

2. AS keyword necessary for column aliases in Postgres (fixed in 8.4).
 Suggestion is to modify even the Oracle code to use this convention, because this convention, being standard, is supported across very many databases, and will cause less pain to anybody trying to fix a bug in both versions (Oracle and Postgres) of code.

3. Do not use keywords as identifiers.
 This is again a coding convention suggestion that we should abstain from using keywords such as 'date', 'time', 'timestamp' as column or object names.

4. Set operator MINUS is not supported; use SQL standard EXCEPT instead.
 occurances in schema: 2; find ./ -name "*.sql" | xargs grep -iw minus

5. Use Orafce, which helps mitigate many porting issues. (TODO: more docs needed on this)

6. Oracle treats {quote}{quote} (empty string) as null, but PG does not.
 So either make the application aware of it, or attach a BEFORE EACH ROW trigger on every table which has a char/varchar column, and set 'if new.char_col = {quote}{quote}then new.char_col = null'.

 Triggers will not help in cases where app uses results of dynamically computed strings.

 occurrences in schema: 17; find ./ -name "*.sql" | xargs grep \'\' | grep -iv values | grep -v rhnFAQ_satdata.sql

7. Convert DECODE calls to CASE expressions.
 Although Orafce provides DECODE support, suggestion here is again to modify the Oracle code to use CASE expressions for the reason that this construct is in SQL standard, and Oracle also supports it.

8. Make the app use standard Outer Join syntax instead of legacy Oracle syntax (+).
 Again, it is recommended that Oracle code also be modified to use the standard syntax.

 occurances in schema: 8 ; find ./ -name "*.sql" | xargs grep \(+\) 

9. Optimizer Hints are not supported by Postgres, but wouldn't cause a problem as they are technically just comments.
 It is suggested to keep them as they wouldn't hurt at all, instead they may aide in some way for debugging performance problem etc. Moreover, if the deployment is ever moved to Postgres compatible Postgres Plus Advanced Server, then they may again come handy as it supports Optimizer Hints.

10. Beware of date+int operations in the application; they mean different things in Oracle and PG.

11. ROWID is not supported by Postgres.
 Fortunately, ROWID keyword is used in very few places across the code, so it can be easily ported to Postgres by using the result of expression 'tableoid::varchar || ctid::varchar' as a row identifier. Better still, using a Primary or Unique Key instead is recommended.

    Consensus: It has been agreed that we should try to use Primary Key instead of ROWID as much as possible.

12. ROWNUM is not supported by Postgres.
 But LIMIT ... OFFSET clause can be used to achieve the same result.

13. Subquery in SET clause of UPDATE command which updates multiple columns is not supported in Postgres.
 Syntax supported:

    UPDATE tbl SET col = (SELECT col from tbl2);

 Syntax not supported:

    UPDATE tbl SET ( col1, col2 ) = (SELECT col1, col2 from tbl2);

This can be done in Postgres by using a FROM clause that contains a subquery:

    UPDATE accounts
    SET     contact_last_name = x.last_name,
            set contact_first_name = x.first_name
    FROM (SELECT last_name, first_name 
          FROM salesmen 
          WHERE salesmen.id = accounts.sales_id) x

14. TODO : Mention port from sysdate to CURRENT_TIMESTAMP, and research their differences (formatting etc.).

15.SELECT DISTINCT, ORDER BY expressions must appear in select list in Postgres but not required in oracle,hence we have write the query in this manner

create table t( a int );
select distinct a from t order by upper( a || '-' ); -- fails on PG
select a from ( select distinct a, upper( a || '-' ) from t order by upper( a || '-' ) ) s; -- works on both Ora and PG


[Tagging examples for queries](QueryTaggingExamples)
# Tablespaces



 Talespaces are supported in Postgres, but they do not mean exactly the same thing as in Oracle. There is a GUC variable in Postgres that allows us to set a global tablespace (at system or session level), and any object being created that does not have a TABLESPACE clause gets created in that {quote}global{quote} tablespace.

 So we have two options.
  1. If having per-object tablespace is important, then we can keep these usages intact for Postgres.
  2. Or, strip off the TABLESPACE clause from all table and index creation scripts, and use the default_tablespace Postgres GUC variable.

Note: Tablespace feature was introduced in Postgres 8.0, and hence not available in Postgres version 7.4.

    Consensus: It has been agreed that since Postgres 8.1 supports tablespace clause, we will keep the tablespace clauses.
# Datatype handling



 Special care needs to be taken while mapping datatypes. Following generic mappings are recommended, but we might need a few iterations to achieve optimal performance while retaining application behavior and portability.

 * varchar2, nvarchar2, clob, long -> varchar (or text)

 * nchar(p) -> char(n)

 * number -> numeric (alternatives: bigint, int, smallint, float, double, real)

 * date -> timestamp (because 'date' in PG stores time component as 00:00:00);
 Choose carefully, what does the application need? date, time or date+time?

 * blob, raw, longraw -> bytea (+application change needed)
# Porting tables



 * Port the triggers to pl/pgsql language (created in the same script as the table).

 * Change DEFAULT clause of 'date' columns from

    defualt (sysdate)
 to

    default(CURRENT_TIMESTAMP)

 * (obsolete) TABLESPACE clause in PRIMARY KEY clause needs to be changed if porting to 7.4 because 7.4 does not support tablespaces.

 * Strip out NOLOGGING clause. (incompatible)
 * Strip out ENABLE ROW MOVEMENT. (incompatible)
 * Strip out STORAGE clause. (incompatible)
# Porting Sequences



 Sequences are available in both Oracle and Postgres, with minor differences.

 * Instead of seq_name.nextval, in Postgres we should do nextval( 'seqname' ).

 * Same applies to curval too.

 * MAXVAL clause of CREATE SEQUENCE command in Postgres is sensitive to the underlying datatype.
# Porting Views



 * Oracle style Outer Join syntax should be changed to ANSI:

    (+) -> {LEFT|RIGHT} [OUTER] JOIN
 Beware when there are many tables in the FROM clause.

 * Some occurances of a function multiset() are seen, which accepts a SELECT query; needs more research. TODO
# Porting Packages



 * Break up packages into individual functions.

 * CURSOR package variables.
 Most package variables are CURSORs; see if they can be replaced with inline cursors in stored procedures. Porting these cursor variables will be problematic if these cursors are opened, executed and closed in different calls to database.

 * Non-CURSOR package variables.
 There is some usage of package-variables that are not CURSORs. These are mostly varchars like 'version'. It is recommended that a database table be created with package_name, variable_name, type, value as columns, and be used in the code. Hopefully, these variables are accesses only in the PL/SQL so the main application can be left alone.

 * Private variables.
 There seems to be some usage of package private-variables, but these are just declarations, with no code using them. Remove them after consensus from community.
# Porting Procedures/Functions/Trigger Bodies



 * Default values for parameters. ( TODO, needs more research )

 * COMMIT/ROLLBACK inside PL/PGSQL.
 Transaction boundaries inside PL/PGSQL are not supported; need to handle this on per-case basis.

 COMMIT/ROLLBACK inside PL/SQL code means that the calling application relies heavily on the fact that the DB can do that. Get the developer's POV on getting rid of such use (even on Oracle PL/SQL).

 Note: SAVEPOINTs in PL/PGSQL 'are' supported using EXCEPTION handling feature.

 * Port DETERMINISTIC pragma to Postgres as IMMUTABLE/STABLE/VOLATILE.

 * RAISE mechanism of raisng exception needs to be migrated to use RAISE EXCEPTION.

 * Autonomous transactions... A BIG TODO.
 Use dblink in conjunction with views to get over this.

 * CURSOR variables are supported in PL/PGSQL, with a slight syntax difference.

 * BULK COLLECT: No direct mapping. TODO
 Posiibily convert into a slow non-BULK operation. The usage that has been noticed till now can be easily modified to not use this feature.

 * Implicit FOR .. LOOP variable declarations.
 PL/SQL does not require explicit declaration of variables used for LOOPing. We have to declare those variables explicitly in PL/PGSQL. The datatype for the CURSOR-FOR LOOP should be RECORD.

 * TODO Get alternative for DUP_VAL_ON_INDEX (one use in new_user_postop.sql)

 * %TYPE usage in parameter datatypes is supported.

 * PRAGMA RESTRICT_REFERENCES(org_channel_setting, WNDS, RNPS, WNPS);
 This is clearly not supported, understand it's necessity and remove if not needed.

 * RAISE_APPLICATION_ERROR() being used.
 Understand the need for it, and port it over to Postgres using RAISE so that the app feels minimal pain.
# Porting Objects



 There's only one OBJECT type in the schema, evr_t. The only requirement that has been noticed of this object is that, that it has to have a ORDER member
function, so that the queries usin this type can perform ORDER BY based on this datatype.

 This simple usage of this feature can be easily ported to Postgres by using a combination of CREATE TYPE and CREATE OPERATOR <=(evr_t, evr_t).
# Porting Synonyms



 Synonyms are not supported in Postgres.

 It seems that synonyms are being used as a convenience (after all, thats what they are!), so we propose that the calling code be changed to reference the objects directly. (Need to vet it out with the community).

    Consensus: It has been agreed that we'll get rid of all the synonyms.
# Porting Types



 There are 4 TYPE objects in schema, and all of them are used only in stored procedures, and two views (using multiset()). So, hopefully it wil be easy to migrate that PL/SQL code, without afecting the rest of the code. TODO

occurances in schema: 16; find ./ -type f | grep -v .html | xargs grep -iwE 'channel_name_t|user_group_id_tuser_group_id_t|user_group_label_t|user_group_name_t'

 Note: Its seems that these types are only being used as arrays (seen from behavior of functions channel_name_join(), ID_JOIN() and LABEL_JOIN() ). In that case, we can easily replace them by postgres ARRAYs.
# Porting Exceptions



 * Port the RHN_Exception package using RAISE EXCEPTION.

 * (obsolete) Postgres version 7.4's PL/PGSQL does not have EXCEPTION support.
# Anonymous Blocks

  Have noticed some anonymous block usage in Python code (search for autonomous_transaction). Anonymous blocks are not supported in Postgres; one has to CREATE FUNCTION for any programmatic interface. So the proposal here is to either reduce these anon-blocks to single line queries that achieve the same affect or create function for each of these and call those functions from Python.

# TODO



 * What would channel_name_join() return if the first IF condition succeeds?
  {quote}{quote} or NULL?

 * One time anonymous block usage in new_user_postop.sql; handle it.

 * "RANK() over partition by" function not supported in PG8.1.
   eg:-SELECT CP.package_id, CP.channel_id,
               RANK() OVER (PARTITION BY P.name_id, P.package_arch_id ORDER BY PE.evr DESC) AS DEPTH