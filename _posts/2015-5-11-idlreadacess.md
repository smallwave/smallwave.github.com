---
layout: post
title: "IDL读取Access数据库"
description: "IDL读取数据库"
category: 'work'
tags: ['Access','IDL','Code']
---

用IDL的DataMiner，它是一个开放数据库连接（ ODBC ）接口，借助它IDL用户可快速访问、查询并管理ODBC兼容数据库，同时也支持Oracle、Informix、Sybase、MS SQL Server等大型商用数据库。本程序实现了访问Access数据库查询功能（基于SQL语句），其中主要函数（1、CONNECTSQL；2、USESQLCONNECTION；3、DISCONNECTSQL）来源于[http://hesperia.gsfc.nasa.gov/ssw/goes/sxig12/idl/IDAC_source/OdbcSqlx.pro](http://hesperia.gsfc.nasa.gov/ssw/goes/sxig12/idl/IDAC_source/OdbcSqlx.pro)

<!--more-->

代码如下：

    ;;******************************************************     ***************************************
    ;+
    ; AUTHOR: wuxb,CAREERI/CAS,Lanzhou,China,730000
    ; Email: wavelet2008@163.com
    ;+
    ;;NAME:
    ;;D:\workspace\Tech\Code\IDL\ENVI_IDL_Proj\SoilTextureProduce\DataPreprocessing\SoilDirllDataPreprocess.pro
    ;;
    ;; :DESCRIPTION:
    ;;    Describe the procedure.
    ;;
    ;;*********************************************************************************************
    ;+
    ;; PARAMETRAS :
    ;;
    ;;
    ;;MODIFICATION HISTORY:
    ;;   Written by : 2015-5-9 11:46:04
    ;;   Modified   : 
    ;;
    ;;*********************************************************************************************
    ;----------------------------------------------------------------------------
    ;Main Program
    ;----------------------------------------------------------------------------
    PRO SOILDRILLDATAPREPROCESS
      ;**************************************************************************
      ; 1) Initialize the Common variables
      ;**************************************************************************
      ;just for Test
      strBDPath       =    'D:\\drlwcon.mdb'
      strTableName    =    'drill'

      ;**************************************************************************
      ; 2) check db status and link db
      ;**************************************************************************
      odbcStatus  =   DB_EXISTS()
      IF (odbcStatus NE 1) THEN RETURN

      strDsource  =  'Microsoft Access Database;Dbq=' + strBDPath
      IF (CONNECTSQL(strDsource,DBobj) NE 1) THEN RETURN
      ;**************************************************************************
      ; 3) link TABLE
      ;**************************************************************************
      table       =  DBobj.GETTABLES()
      index       =  WHERE(table.NAME EQ strTableName,matchNum)
      IF (matchNum NE 1) THEN RETURN
      ;**************************************************************************
      ; 4) preProcess data
      ;**************************************************************************
      strSQL      = 'select * from '+ strTableName + ' where DrillNo=43'
      nrec = USESQLCONNECTION(DBobj,strSQL,PTR=ptr)
      IF (nrec NE 1) THEN RETURN
      PRINT,(*ptr)
  
      ;**************************************************************************
      ; 5) distory DBobj
      ;**************************************************************************
      nrec = DISCONNECTSQL(DBobj)
      IF (nrec EQ 1) THEN PRINT,'Success Quit!!!'
  
    END