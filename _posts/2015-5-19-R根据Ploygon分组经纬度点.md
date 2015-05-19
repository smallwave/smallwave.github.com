---
layout: post
title: "R根据Ploygon分组Point"
description: "R根据Ploygon分组Point"
category: 'work'
tags: ['Soil','R','Vecter']
---

R根据shapefiles包可以读取Shapefile，并依据sp [访问] (http://cran.r-project.org/package=sp)包对空间数据处理，其中一个基本的需求是对空间点按照不同的Ploygon分组，以下Code实现了这一基本功能，主要用到Sp的over函数，函数返回Ponit所在Ploygon的ID。

<!--more-->


    
    library(shapefiles)
    library(sp)
    #Read shapefile
    shpFilePloy <- read.shp(shapeFile)  # shapeFile eq "Shp/Ploygon.shp"
    #simple Ploy
    simpleShpFormat <- convert.to.simple(shpFilePloy)
    #convert list
    simpleAsList <- by(simpleShpFormat, simpleShpFormat[,1], function(x) x)
    #get Dim
    nPloy <- attributes(simpleAsList)$dim
    listPloys <- list()
    #construct SpatialPolygons
    for (i in 1:nPloy) 
    {
      cbTmp  <- cbind(simpleAsList[[i]]$X,simpleAsList[[i]]$Y)
      plyTmp <- Polygon(cbTmp)
      id <-paste("p",i,sep='')
      plyTmps<-Polygons(list(Polygon(plyTmp)),id)
      listPloys <- append(listPloys, list(plyTmps))
    }
    spPloys = SpatialPolygons(listPloys)
    #use over
    groupID <- over(soilDataRes,spPloys)






