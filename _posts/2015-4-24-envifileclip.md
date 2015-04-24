---
layout: post
title: "ENVI标准格式通过Shapefile批量裁剪"
description: "Envi格式裁剪"
category: 'work'
tags: ['ShapeFile','IDL','Code']
---

ENVI标准格式通过ShapeFile裁剪是数据处理的基本功能，本文实现了其批处理，源代码代码修改来自于[http://www.cnblogs.com/gisoracle/p/3663707.html](http://www.cnblogs.com/gisoracle/p/3663707.html)

<!--more-->

1.原始数据和裁剪后对比

> ![](/images/EnviClip.jpg)


2.代码（需要ENVI+IDL运行  支持批处理）


    ;+
    ; ;**********************************************************
    ; Author: wuxb
    ; Email: wavelet2008@163.com
    ; 
    ; NAME:
    ;    D:\workspace\Tech\Code\IDL\ENVI_IDL4.7\SoilTextureProduce\DataPreprocessing\CLIP_ENVIFILE_BY_SHP.pro
    ; PARAMETERS:
    ; Some Path
    ; Write by :
    ;    2015-4-24 13:25:53
    ;;MODIFICATION HISTORY:
    ;    Modified BY Source :　http://www.cnblogs.com/gisoracle/p/3663707.html
    ;;   Modified  :  
    ;
    ;***********************************************************
    ;-
    PRO CLIP_ENVIFILE_BY_SHP,EnviDirectory = EnviDirectory, ClipResDirectory = ClipResDirectory, ShpFilePath = ShpFilePath
      ;**************************************************************************
      ;
      ; 1) Initialize the Common variables
      ;
      ;**************************************************************************
      ;just for Test
      EnviDirectory     = 'D:\worktemp\SoilTextureProduce\DataProcess\Output\'
      ClipResDirectory  = 'D:\worktemp\SoilTextureProduce\DataProcess\ClipOutput\'
      ShpFilePath       = 'D:\worktemp\SoilTextureProduce\DataProcess\Boundary\Tibet_Plateau_Boundar.shp'
      ;**************************************************************************
      ;
      ; 2) Get the files to be clip
      ;
      ;**************************************************************************
      ;Find Data
      ENVIFilePaths = FILE_SEARCH(EnviDirectory,'*.HDR', COUNT = nCount, /TEST_READ, /FULLY_QUALIFY_PATH)
      IF(nCount LE 0) THEN RETURN
      ;Process
      FOR i = 0,nCount-1 DO BEGIN
        ;**************************************************************************
        ; 1) open HDR Data get Infomation
        ;**************************************************************************
        EnviFile = ENVIFilePaths[i]
        IF STRLEN(EnviFile) EQ 0 THEN RETURN
        ENVI_OPEN_FILE, EnviFile, r_fid=fid, /no_interactive_query, /no_realize
        IF fid EQ -1 THEN  RETURN
        ENVI_FILE_QUERY, fid, file_type=file_type, nl=nl, ns=ns,dims=dims,nb=nb
        ;**************************************************************************
        ; 2) open shp data get Infomation
        ;**************************************************************************
        IF STRLEN(ShpFilePath) EQ 0 THEN RETURN
        oshp = OBJ_NEW('IDLffshape',ShpFilePath)
        oshp->GETPROPERTY,n_entities=n_ent,Attribute_info=attr_info,$
          n_attributes=n_attr,Entity_type=ent_type
        roi_shp = LONARR(n_ent)
        FOR ishp = 0,n_ent-1 DO BEGIN
          entitie = oshp->GETENTITY(ishp)
          IF entitie.SHAPE_TYPE EQ 5 THEN BEGIN
            record = *(entitie.VERTICES)
            ;Convert map Coordinates
            ENVI_CONVERT_FILE_COORDINATES,fid,xmap,ymap,record[0,*],record[1,*]
            ;Creat ROI
            roi_shp[ishp] = ENVI_CREATE_ROI(ns=ns,nl=nl)
            ENVI_DEFINE_ROI,roi_shp[ishp],/polygon,xpts=REFORM(xmap),ypts=REFORM(ymap)
            ;记录X,Y的区间，裁剪用
            IF ishp EQ 0 THEN BEGIN
              xMin = ROUND(MIN(xMap,max = xMax))
              yMin = ROUND(MIN(yMap,max = yMax))
            ENDIF ELSE BEGIN
              xMin = xMin < ROUND(MIN(xMap))
              xMax = xMax > ROUND(MAX(xMap))
              yMin = yMin < ROUND(MIN(yMap))
              yMax = yMax > ROUND(MAX(yMap))
            ENDELSE
          ENDIF
          oshp->DESTROYENTITY,entitie
        ENDFOR;ishp
        xMin = xMin >0
        xMax = xMax < ns-1
        yMin = yMin >0
        yMax = yMax < nl-1
        ;**************************************************************************
        ; 3) output file setting
        ;**************************************************************************
        OutFilename = ClipResDirectory + FILE_BASENAME(EnviFile,'.HDR')+'_Clip'
        out_dims    = [-1,xMin,xMax,yMin,yMax]
        pos = INDGEN(nb)
        ENVI_DOIT,'ENVI_SUBSET_VIA_ROI_DOIT',background=0,fid=fid,dims=out_dims,out_name=OutFilename,$
      ns = ns, nl = nl,pos=pos,roi_ids=roi_shp
      ENDFOR
