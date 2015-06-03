---
layout: post
title: "总结一些用于土壤制图的R包"
description: "Some R packages"
category: 'work'
tags: ['Soil','R']
---


R对土壤数据的制图（digital soil mapping DSM）支持的功能还是比较全面的，本文主要整理一下最近用到的、不可或缺的R packages，主要分为：

- （1）GIS：空间数据（矢量和栅格）处理和分析，包括sp、raster、rgdal、RSAGA、maptools；
- （2）模型：土壤制图所需要的分类模型，包括caret、Cubist、C5.0、gam、nnet、gstat；
- （3）出图：探索性分析和出图，包括ggplot2；
- （4）专业：包括soilPhysical(如土壤水分特征曲线的分析)，soilTexture，GSIF。

各包具体链接如下：

----------


<!--more-->


## GIS ##


- sp: [http://cran.r-project.org/web/packages/sp/index.html](http://cran.r-project.org/web/packages/sp/index.html).
- raster: [http://cran.r-project.org/web/packages/raster/index.html](http://cran.r-project.org/web/packages/raster/index.html). 
- rgdal: [http://cran.r-project.org/web/packages/rgdal/index.html](http://cran.r-project.org/web/packages/rgdal/index.html). 
- RSAGA: [http://cran.r-project.org/web/packages/RSAGA/index.html](http://cran.r-project.org/web/packages/RSAGA/index.html). 
- maptools：[http://cran.r-project.org/web/packages/maptools/index.html](http://cran.r-project.org/web/packages/maptools/index.html)

## Modelling ##


- caret: [http://cran.r-project.org/web/packages/caret/index.html](http://cran.r-project.org/web/packages/caret/index.html). 
- Cubist: [http://cran.r-project.org/web/packages/Cubist/index.html](http://cran.r-project.org/web/packages/Cubist/index.html). 
- C5.0: [http://cran.r-project.org/web/packages/C50/index.html](http://cran.r-project.org/web/packages/C50/index.html). 
- gam: [http://cran.r-project.org/web/packages/gam/index.html](http://cran.r-project.org/web/packages/gam/index.html). 
- nnet: [http://cran.r-project.org/web/packages/nnet/index.html](http://cran.r-project.org/web/packages/nnet/index.html).
- gstat: [http://cran.r-project.org/web/packages/gstat/](http://cran.r-project.org/web/packages/gstat/). 


## Mapping and plotting ##


- ggplot2: [http://cran.r-project.org/web/packages/ggplot2/index.html](http://cran.r-project.org/web/packages/ggplot2/index.html). 


## Specific ##


- GSIF：[http://cran.r-project.org/web/packages/GSIF/index.html](http://cran.r-project.org/web/packages/GSIF/index.html). 
- soilTexture：[http://cran.r-project.org/web/packages/soilTexture/index.html](http://cran.r-project.org/web/packages/soilTexture/index.html). 
- soilPhysical：[http://cran.r-project.org/web/packages/soilTexture/index.html](http://cran.r-project.org/web/packages/soilTexture/index.html). 





