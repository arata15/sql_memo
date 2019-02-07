==========================
SQL メモ
==========================

DB LINK作成
====================================

::

  CREATE [PUBLIC] DATABASE LINK <リンク名> CONNECT TO <ユーザ名> IDENTIFIED BY <パスワード> USING '<DB接続名>'

例 ::

  CREATE DATABASE LINK LINK_TEST CONNECT TO testname IDENTIFIED BY testpass USING 'TEST';

DB_LINK確認
====================================

::
  
  SELECT DB_LINK FROM user_db_links;

疎通確認
====================================

::
  
  SELECT OBJECT_NAME FROM USER_OBJECTS@LINK_TEST;
  
DB LINKを削除
====================================

::

  DROP DATABASE LINK LINK_TEST

シノニム作成
====================================

::

  CREATE SYNONYM test_synonym FOR test_synonym@LINK_TEST

権限
====================================

::

  GRANT select,update,delete,insert ON test_synonym TO rivus

表領域確認
====================================

::

  select
  tablespace_name,
  to_char(nvl(total_bytes / 1024 / 1024,0),'999,999,999') as "size(MB)",
  to_char(nvl((total_bytes - free_total_bytes) / 1024 / 1024,0),'999,999,999') as "used(MB)",
  to_char(nvl(free_total_bytes/1024 / 1024,0),'999,999,999') as "free(MB)",
  round(nvl((total_bytes - free_total_bytes) / total_bytes * 100,100),2) as "rate(%)"
  from
  ( select
  tablespace_name,
  sum(bytes) total_bytes
  from
  dba_data_files
  group by
  tablespace_name
  ),
  ( select
  tablespace_name free_tablespace_name,
  sum(bytes) free_total_bytes
  from
  dba_free_space
  group by tablespace_name
  )
  where
  tablespace_name = free_tablespace_name(+)

テーブルごとの件数確認
====================================

::

  SELECT
      TABLE_NAME,
      TO_NUMBER(
        EXTRACTVALUE(
          XMLTYPE(
            DBMS_XMLGEN.GETXML('SELECT COUNT(*) C FROM '||TABLE_NAME))
            ,'/ROWSET/ROW/C')) COUNT
  FROM USER_TABLES
  WHERE TABLE_NAME NOT LIKE 'BIN$%'
  AND (IOT_TYPE != 'IOT_OVERFLOW' OR IOT_TYPE IS NULL)
  ORDER BY TABLE_NAME;





