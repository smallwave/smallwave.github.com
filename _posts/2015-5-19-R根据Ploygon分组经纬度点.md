---
layout: post
title: "R根据Ploygon分组Point"
description: "R根据Ploygon分组Point"
category: 'work'
tags: ['Soil','R','Vecter']
---

R根据shapefiles包可以读取Shapefile，并依据sp [Link ](http://cran.r-project.org/package=sp)包对空间数据处理，其中一个基本的需求是对空间点按照不同的Ploygon分组，要实现了这一基本功能，主要用到Sp的over函数，该函数返回Ponit所在Ploygon的ID。

修改记录（2015.5.20）: 如果Polygon比较复杂（如MutilPloygon），用shapefiles包出现了问题，可能原因是convert.to.simple简化后，Ploygon的上点的顺序发生变化，所以在取点的位置时，有的点明明在Ploygon里面而没有找到。但可以用mapTools [Link ](http://cran.r-project.org/web/packages/maptools/index.html)包readShapeSpatial函数，该函数使shapefiles一下子就转换成SpatialPolygonsDataFrame对象了，该对象也可以直接用于Over函数的参数，这种方法可以说更简单，更合理，更高效，推荐用这种方法。


<!--more-->

1、用shapefiles包构建SpatialPolygons对象（慎用）
- 

----------

    library(shapefiles)
    library(sp)
    #Read shapefile
    shpFilePloy <- read.shp(shapeFile)  # eg "Shp/Ploygon.shp"
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
    spatialPoint<- SpatialPoints(dt[c("Lon","Lat")])
    groupID <- over(spatialPoint,spPloys)

2、用mapTools包构建SpatialPolygonsDataFrame对象（推荐）
- 

----------

    #library(maptools)
    #library(sp)
    #Read shapefile
    shapefile <- readShapeSpatial(shapeFile)  # eg "Shp/Ploygon.shp"
    #use over
    spatialPoint<- SpatialPoints(dt[c("Lon","Lat")])
    groupID <- over(spatialPoint,shapefile)







