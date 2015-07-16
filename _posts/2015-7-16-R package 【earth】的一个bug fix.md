---
layout: post
title: "R package [earth] error report"
description: "R根据Polygon分组Point"
category: 'work'
tags: ['Soil','R','Vecter']
---

今天用到了R package [earth](http://cran.r-project.org/web/packages/earth/index.html) ,该包有个数据挖掘的多元自适应回归样条法(multivariate adaptive regression splines),但是，用该方法来分类时，就会出Bug。如下面的Bug提示信息：

    Error in ylevels[apply(map.RF, 1, which1)] : 
      invalid subscript type 'list'

查找原因花了几个小时，终于发现错误的源头。

<!--more-->


1、问题描述：
-

----------
  测试Example：
   
    hv_mars <- earth(formula, data = data)
    map.RF <- predict(hv_mars, newdata = ov_data, type = "class")


   只要一运行predict 就会报上面的错误，用type = "response" 就没有错误。
查看predict的源码[https://github.com/cran/earth/edit/master/R/predict.earth.R](https://github.com/cran/earth/edit/master/R/predict.earth.R)，可以清楚的看到class是基于response的，只是class多了一个方法

    convert.predicted.response.to.class <- function(resp, ylevels, resp.name, thresh=.5)

继续追踪源码，发现convert.predicted.response.to.class里面有个函数 which1


    which1 <- function(row, thresh) # row is a scalar or a vector
    {
        if(length(row) > 1)
            which. <- which.max(row)
        else
            which. <- if(row > thresh) 2 else 1
        which.
    }


一行一行的看，问题终于找到，如果某一行row的最大值有大于等于2个时候，即返回which.的维数>2，则会出现上述错误。


2、Bug修正
-

----------

在实际工作中，一行如[1，2，5，4，5], which.max则返回3和5，其实我们只要一个最大值作为结果即可，简单修正which1如下：
    
    which1 <- function(row, thresh) # row is a scalar or a vector
    {
        if(length(row) > 1)
            which. <- which.max(row)
        else
            which. <- if(row > thresh) 2 else 1
        which.[1]
    }

经测试，目前还没发现问题。






