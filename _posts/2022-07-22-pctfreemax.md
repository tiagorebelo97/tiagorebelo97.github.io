---
layout: post
title: pctfreemax
date: 2022-07-22 12:00:00 -500
categories: [Scripts, Storage]
tags: [scripts,Storage,tablespaces]
---

# pctfreemax.sql

```sql
set echo off
--set lines 500
--set pages 500
col tablespace for a30
col status for a8
col total_mb for 999,999,999
col used_mb for 999,999,999
col free_mb for 999,999,999
col max_free for 999,999,999
col pct_used for 999,999,999
col max_pct_used for 999,999,999
col GRAPH for a25

select total.ts tablespace,
 DECODE(total.mb,null,'OFFLINE',dbat.status) status,
 total.mb total_mb,
 NVL(total.mb - free.mb,total.mb) used_mb,
 NVL(free.mb,0) free_mb,
 round(max.max_mb - NVL(total.mb - free.mb,total.mb),2) max_free,
 DECODE(total.mb,NULL,0,NVL(ROUND((total.mb - free.mb)/(total.mb)*100,2),100)) pct_used,
 DECODE(total.mb,NULL,0,NVL(ROUND((total.mb - free.mb)/(max.MAX_Mb)*100,2),100)) max_pct_used,
 CASE WHEN (total.mb IS NULL) THEN '['||RPAD(LPAD('OFFLINE',13,'-'),20,'-')||']'
 ELSE '['|| DECODE(free.mb,
 null,'XXXXXXXXXXXXXXXXXXXX',
 NVL(RPAD(LPAD('X',trunc((ROUND((total.mb - free.mb)/(max.MAX_Mb)*100,2))/5),'X'),20,'-'),
 '--------------------'))||']'
 END as GRAPH
from
 (select tablespace_name ts, sum(bytes)/1024/1024 mb from dba_data_files group by tablespace_name) total,
 (select tablespace_name ts, sum(bytes)/1024/1024 mb from dba_free_space group by tablespace_name) free,
 (select tablespace_name ts, sum(decode(maxbytes, '0', round(bytes/1024/1024,0), round(maxbytes/1024/1024,0))) MAX_Mb from dba_data_files df group by tablespace_name) max,
 dba_tablespaces dbat
where total.ts=free.ts(+) and
 total.ts=dbat.tablespace_name and
 total.ts=max.ts
--UNION ALL
--select sh.tablespace_name,
-- 'TEMP',
-- SUM(sh.bytes_used+sh.bytes_free)/1024/1024 total_mb,
-- SUM(sh.bytes_used)/1024/1024 used_mb,
-- SUM(sh.bytes_free)/1024/1024 free_mb,
-- ROUND(SUM(sh.bytes_used)/SUM(sh.bytes_used+sh.bytes_free)*100,2) pct_used,
-- '['||DECODE(SUM(sh.bytes_free),0,'XXXXXXXXXXXXXXXXXXXX',
-- NVL(RPAD(LPAD('X',(TRUNC(ROUND((SUM(sh.bytes_used)/SUM(sh.bytes_used+sh.bytes_free))*100,2)/5)),'X'),20,'-'),
-- '--------------------'))||']'
--FROM v$temp_space_header sh
--GROUP BY tablespace_name
order by 8 desc
/
set echo on
--Prompt @ver_datafile_por_tbs
--Prompt @segments [tablespace_name]
set echo off
```