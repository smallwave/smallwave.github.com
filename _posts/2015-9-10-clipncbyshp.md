---
layout: post
title: "NetCDF数据通过shaplefile文件裁剪"
description: "NetCDF数据通过shaplefile文件裁剪""
category: 'work'
tags: ['NetCDF','python','Code']
---

NetCDF（[http://www.unidata.ucar.edu/software/netcdf/](http://www.unidata.ucar.edu/software/netcdf/)）格式是比较常见的地学数据存储格式,读写NetCDF有很多库，如C++，java，Fortran，但是，很多NetCDF数据都是全球或大区域尺度，存储数据很浪费空间。网上也没有现成的code。所以，本文描述了通过Python调用netcdf（需要[http://code.google.com/p/netcdf4-python/](http://code.google.com/p/netcdf4-python/)支持）融入shapefile来裁剪netcdf数据。

<!--more-->

1.效果

----------

 原始示例NetCDF下载地址[http://westdc.westgis.ac.cn/data/7a35329c-c53f-4267-aa07-e0037d913a21](http://westdc.westgis.ac.cn/data/7a35329c-c53f-4267-aa07-e0037d913a21)

> 原始数据：

----------

> ![原始数据](/images/CFMDCHINA.jpg)



> clip后的数据：

----------

> ![clip后的数据](/images/CFMDQTP.jpg)



2.实现（需要PYTHON运行  支持批处理）

----------


具体思路如下：

- 计算shapefile 的 四角boundary 和所有点的boundary
- 其次根据四角boundary subset
- 在根据所有点boundary 计算netcdf mask并更新到QTP netcdf
具体code如下：

    ##########################################################################################################
	# NAME
	#    SubSetNetCDF.py
	# PURPOSE
	#   
	# PROGRAMMER(S)
	#   wuxb
	# REVISION HISTORY
	#    20150909 -- Initial version created and posted online
	#
	# REFERENCES
	##########################################################################################################
	import numpy as np
	from netCDF4 import Dataset  
	from osgeo import gdal, gdalconst, ogr
	import numpy.ma as ma
	import os  
	import formic
	import datetime
	###################################################################################
	# input data 
	###################################################################################
	strShpFilePath          = "D:\\Temp\\Shp\\Tibet_Plateau_Boundar.shp"
	strNetCDFPathInput      = "D:\\Temp\\Input\\"
	strNetCDFPathOutput     = "D:\\Temp\\Output\\"
	###################################################################################
	# Process
	# read
	# process
	# write
	###################################################################################
	#read shapefile
	shpDS        = ogr.Open(strShpFilePath)
	shpLyr       = shpDS.GetLayer()
	Envelop      = shpLyr.GetExtent() 
	xmin,ymin,xmax,ymax=[Envelop[0],Envelop[2],Envelop[1],Envelop[3]] #Your extents as given above

	mask_RES     = []
	#
	fileset  = formic.FileSet(include="**/*.nc", directory=strNetCDFPathInput)
	nFile    = 0 # print process file ID
	for file_name in fileset:
    	nFile+=1
   	    print "################################################################"
    	print "Current Process file is : " + file_name +"; it is the " + str(nFile)
    	print "################################################################"
    	#read netcdf lon lat data
    	ncInput = Dataset(file_name)
   	    #list variables
    	varList = ncInput.variables.keys()
    	try:   
       	    if len(varList) == 4:
           	 str_convert = ' '.join(varList)
           	 print "netcdf var is : " + str_convert
   	    except:
        	exit(0)

   		#get var
    	for varName in varList:
        	if not cmp(varName, 'lon'):
            	lon_Ori = ncInput.variables['lon'][:]
            	continue
       	 if not cmp(varName, 'lat'):
            	lat_Ori = ncInput.variables['lat'][:]
           	 continue
        	if not cmp(varName, 'time'):
            	time    = ncInput.variables['time'][:]  # this is only clip by spatial ,not time
           	 continue
   	    #finally var is data
    	varData     =  varName
    	var_Ori = ncInput.variables[varData][:]  # shape is time, lat, lon as shown above
    	########################################################################
   	 	#create mask
    	########################################################################
    	if len(mask_RES) == 0 :
        	#get boundary and xs ys
       	    lat_bnds, lon_bnds = [ymin, ymax], [xmin, xmax]
            lat_inds = np.where((lat_Ori > lat_bnds[0]) & (lat_Ori < lat_bnds[1]))
       	    lon_inds = np.where((lon_Ori > lon_bnds[0]) & (lon_Ori < lon_bnds[1]))
        	ncols     = len(lon_inds[0])
        	nrows     = len(lat_inds[0])
        	#create geotransform
        	xres=(xmax-xmin)/float(ncols)
        	yres=(ymax-ymin)/float(nrows)
        	geotransform=(xmin,xres,0,ymax,0, -yres)
       		#create mask
       		 mask_DS  = gdal.GetDriverByName('MEM').Create('', ncols, nrows, 1 ,gdal.GDT_Int32)
        	mask_RB  = mask_DS.GetRasterBand(1)
        	mask_RB.Fill(0) #initialise raster with zeros
        	mask_RB.SetNoDataValue(-32767)
        	mask_DS.SetGeoTransform(geotransform)
        	maskvalue = 1
        	err = gdal.RasterizeLayer(mask_DS, [maskvalue], shpLyr)
        	mask_DS.FlushCache()
        	mask_array  = mask_DS.GetRasterBand(1).ReadAsArray()    
        	mask_RES    = ma.masked_equal(mask_array, 255)          
        	ma.set_fill_value(mask_RES, -32767)  
    	########################################################################
   		 #subset
    	########################################################################
    	var_subset = var_Ori[:,min(lat_inds[0]):max(lat_inds[0])+1, min(lon_inds[0]):max(lon_inds[0])+1]
    	var_subset._set_mask(np.logical_not(np.flipud(mask_RES.mask)))  # update mask  (flipud is reverse 180)
    	lon_subset = lon_Ori[lon_inds]
    	lat_subset = lat_Ori[lat_inds]
    	###################################################################################
   	 	# Open a new NetCDF file to write the data to. For format, you can choose from
   		 # 'NETCDF3_CLASSIC', 'NETCDF3_64BIT', 'NETCDF4_CLASSIC', and 'NETCDF4'
    	###################################################################################
    	#create file path and name
    	InputFileDir,InputFile     = os.path.split(file_name);  
    	InputDir,InputFileDirName  = os.path.split(InputFileDir);  
    	OutputFileDir              = os.path.join(strNetCDFPathOutput,InputFileDirName)
    	if not os.path.exists(OutputFileDir):
        	os.makedirs(OutputFileDir)
    	strInputFileList           = InputFile.split('_')
    	strInputFileList[1]        = strInputFileList[1]+"-QTP"
    	OutfileName                = '_'.join(strInputFileList)
    	OutputFileDirName          = os.path.join(OutputFileDir,OutfileName)
    	#create file 
   		ncOutput = Dataset(OutputFileDirName, 'w', format='NETCDF3_CLASSIC')
    	ncOutput.description = "Extract Data %s from CFMD by QTP shapefile. %s" %\
                      (ncInput.variables[varData].long_name.lower(),ncInput.description)
    	# Using our previous dimension info, we can create the new time dimension
    	# Even though we know the size, we are going to set the size to unknown
   		ncOutput.createDimension('time', None)
    	ncOutput.createDimension('lon', ncols)
        ncOutput.createDimension('lat', nrows)

    	# Add lon Variable
    	var_out_lon = ncOutput.createVariable('lon', ncInput.variables['lon'].dtype,('lon',))
    	for ncattr in ncInput.variables['lon'].ncattrs():
        	var_out_lon.setncattr(ncattr, ncInput.variables['lon'].getncattr(ncattr))
    	ncOutput.variables['lon'][:] = lon_subset

    	# Add lat Variable
    	var_out_lat = ncOutput.createVariable('lat', ncInput.variables['lat'].dtype,('lat',))
    	for ncattr in ncInput.variables['lat'].ncattrs():
        	var_out_lat.setncattr(ncattr, ncInput.variables['lat'].getncattr(ncattr))
    	ncOutput.variables['lat'][:] = lat_subset

    	# Add time Variable
    	var_out_time = ncOutput.createVariable('time', ncInput.variables['time'].dtype,('time',))
    		for ncattr in ncInput.variables['time'].ncattrs():
       	 var_out_time.setncattr(ncattr, ncInput.variables['time'].getncattr(ncattr))
    	ncOutput.variables['time'][:] = time

    	# Add data Variable
    		var_out_data = ncOutput.createVariable(varData, ncInput.variables[varData].dtype, ("time","lat","lon",))
    	for ncattr in ncInput.variables[varData].ncattrs():
          var_out_data.setncattr(ncattr, ncInput.variables[varData].getncattr(ncattr))
       ncOutput.variables[varData][:] = var_subset

       # attr
       ncOutput.history = "CLIP Created datatime"+ datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S") +" by CAREERI  wuxb"
       ncOutput.source  = "netCDF4 1.1.9  python"
       ###################################################################################
       # write close
       ###################################################################################
       # close
       ncOutput.close()  # close the new file
       ncInput.close()   # close ori file

