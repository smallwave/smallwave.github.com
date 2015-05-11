---
layout: post
title: "IDL读取Access数据库"
description: "IDL读取数据库"
category: 'work'
tags: ['Access','IDL','Code']
---

IDL的DataMiner是通用的ODBC的函数集，支持链接、查询、修改数据库，本程序实现了Access数据库查询功能（SQL语句），其中主要函数来源于[http://hesperia.gsfc.nasa.gov/ssw/goes/sxig12/idl/IDAC_source/OdbcSqlx.pro](http://hesperia.gsfc.nasa.gov/ssw/goes/sxig12/idl/IDAC_source/OdbcSqlx.pro)

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
    FUNCTION CONNECTSQL,dsource,dbObj
      ; Function connects to a SQL data source and returns
      ; 1 if successful, 0 if not successful.
      ; dsource is the ODBC data source, e.g. 'SBOX'
      ; set up using ODBC Administrator (on a PC)

      dbObj = OBJ_NEW('IDLdbDatabase')
      dbObj->CONNECT,datasource=dsource  
      dbObj->GETPROPERTY,is_connected=con
      RETURN,con
    END
    ;----------------------------------------------------------------------------
    FUNCTION DISCONNECTSQL,dbObj
      ; Check that dbObj is a valid IDLdbDatabase object
      dbObj->GETPROPERTY,is_connected=con
      IF con THEN BEGIN
        OBJ_DESTROY,dbObj
        RETURN,1
      ENDIF ELSE BEGIN
        PRINT,'Not a valid IDLdbDatabase object ',dbObj
        RETURN,0
      ENDELSE
    END
        ;----------------------------------------------------------------------------
    FUNCTION USESQLCONNECTION,dbObj,q,ptr=rp

      ; UseSQLConnection function uses IDL's Data Miner to retrieve data from
      ; a SQL database that has already been connected, e.g. using ConnectSQL().

      ; Sample calling statement:  nrec = UseSQLConnection(dbObj,q,ptr=rp)
      ; where dbObj=  IDLdbDatabase object (already connected)
      ;   q=    SQL query string
      ;   ptr=  Variable to hold pointer to data array
      ;   nrec= Number of records found (returned as value of Dminer2 function)
     ; Individual elements of the array pointed to by ptr are accessed like this:
      ;       print,(*rp)(0,1)
      ; Access complete individual records as: (*rp)(i)
      ; Access individual fields (elements of i), as (*rp)    (i,j)
      ; Can also do a range of a field as (*rp)(x:y,j), where
      ; x and y are record indices.

      ;=======================================================================

      ; Check that the database object is valid
      dbObj->GETPROPERTY,is_connected=con
      IF con THEN BEGIN
        rsObj = OBJ_NEW('IDLdbRecordset',dbObj,sql=q)
        nf = rsObj->NFIELDS()  ; This gives only the *selected* fields (q, above)

    ; Determine the number of records requested.
    ; (Movecursor(/first) returns 0 if there are no records.)

    res = rsObj->MOVECURSOR(/first)
    IF res EQ 0 THEN RETURN,0
    n1 = rsObj->CURRENTRECORD()
    res = rsObj->MOVECURSOR(/last)
    IF res EQ 0 THEN RETURN,0
    nx = rsObj->CURRENTRECORD()
    nr = nx - n1 + 1
    IF nr EQ 0 THEN RETURN,0
    ;print,'Records retrieved = ',nr

    ; Get field info (name,datatype, etc)
    rsObj->GETPROPERTY,field_info=info

    ; Retrieve the data as 2D string array

    r = STRARR(nr,nf) ; String array to hold results
    res = rsObj->MOVECURSOR(/first)
    ; Format for datetime data:
    fmt = "(I4,'-',I2.2,'-',I2.2,1x,I2.2,':',I2.2,':',I2.2,'.',I3.3)"
    FOR i = 0,nr-1 DO BEGIN
      FOR j = 0,nf-1 DO BEGIN
        val = rsObj->GETFIELD(j)
        IF info[j].TYPE_NAME EQ 'DATETIME' THEN BEGIN
          ; This j is a datetime field--form datetime string
          ; Convert fraction to milliseconds
          foo = val.FRACTION/1e9
          valStr = STRING(format=fmt, $
            val.YEAR,val.MONTH,val.DAY, $
            val.HOUR,val.MINUTE,val.SECOND,foo)
          GOTO,skip
        ENDIF
        IF info[j].TYPE_NAME EQ 'bit' THEN BEGIN
          val = FIX(val)
          valStr = STRTRIM(STRING(val),2)
          GOTO,skip
        ENDIF
        IF info[j].TYPE_NAME EQ 'tinyint' THEN BEGIN
          val = FIX(val)
          valStr = STRTRIM(STRING(val),2)
          GOTO,skip
        ENDIF
        IF info[j].TYPE_NAME EQ 'real' THEN BEGIN
          val = FLOAT(val)
          valStr = STRTRIM(STRING(val),2)
          GOTO,skip
        ENDIF
        IF info[j].TYPE_NAME EQ 'float' THEN BEGIN
          val = DOUBLE(val)
          valStr = STRTRIM(STRING(val),2)
          GOTO,skip
        ENDIF
        valStr = STRTRIM(STRING(val),2)
        skip:
        r[i,j] = valStr
      ENDFOR
      res = rsObj->MOVECURSOR(/next)
    ENDFOR

    ; Create a pointer to the 2D string array, saves tons of
    ; memory.  The function returns the number of rows, nr.
    ; The pointer is returned as the value of the 'pointer' keyword.
    ; Individual elements of the array are accessed like this:
    ;       print,(*rp)(0,1)
    ; where this function (UseSQLConnection) was called like this:
    ;   nr = UseSQLConnection(dbObj,q,ptr=rp)
    rp = PTR_NEW(r)
    RETURN,nr

      ENDIF ELSE BEGIN
        PRINT,'Not a valid IDLdbDatabase object: ',dbObj
      ENDELSE

    END

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