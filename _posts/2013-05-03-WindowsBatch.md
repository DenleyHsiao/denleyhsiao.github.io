---
layout: post
title:  "Windows批处理"
date:   2013-05-03 10:25:00
categories: windows
---

{{page.title}}

##1. [获取批处理文件所在路径](http://blog.csdn.net/longtrue/article/details/2499402)

##2. 批处理获取当前路径  
	@echo off  
	echo 当前盘符：%~d0  
	echo 当前盘符和路径：%~dp0  
	echo 当前批处理全路径：%~f0  
	echo 当前盘符和路径的短文件名格式：%~sdp0  
	echo 当前CMD默认目录：%cd%  
	pause  
	@echo on  

##3. [使用批处理文件设置环境变量](http://blog.csdn.net/clever101/article/details/7956308)
