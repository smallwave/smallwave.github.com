---
layout: post
title: "64 bit debugging not working in IVF"
description: "Fortran"
category: 'work'
tags: ['Fortran']
---


I'm using Intel Visual Fortran Composer XE 2013 SP1 Update 3 and debugging in Visual Studio 2013 Update 2's IDE with this simple program:

----------

	program Console1 
	implicit none 
	! Variables  
	integer(4) testInt 
	! Body of Console1 
	print *, 'Hello World'  
	testInt = 7 
	end program Console1 

----------

<!--more-->

When the EXE for this program is compiled with the "IA 32" (the Win32 platform) ifort compiler, the debugging works as expected. The watch on `testInt` changes from a random value to 7.

But, when I use the "Intel (R) 64" (the x64 platform) compiler by setting the project to the x64 platform, the debugger will stop at correct break points. But the watch on `testInt` reports "Undefined address" instead of 7. 

What's going on here? How can I enable correct debugging for 64 bit Fortran compilations? This is interfering with COM Server development where the COM client and server must be 64-bit.
[问题来源](https://software.intel.com/en-us/forums/intel-visual-fortran-compiler-for-windows/topic/515943)

解决方式：
Yes, our messaging system is broken. I'm just going to attach the fix here. Unpack the attached ZIP into C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Packages\Debugger  This will be fixed in Update 4.
[FEE下载链接：](http://pan.baidu.com/s/1dDsH2CD)













