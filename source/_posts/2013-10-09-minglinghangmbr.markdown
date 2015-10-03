---
layout: post
title: "命令行修复MBR"
date: 2013-10-09 16:22
comment: true
categorie: 
---
命令行修复MBR  

	1. shift+F10打开命令行
	2. 输入：diskpart
	3. 输入：list disk 查看磁盘信息
	4. 选择你要操作的磁盘：
	select disk 0
	5. 输入：clean,清除分区
	6. 输入：convert mbr 转换为MBR分区
	7. 退出，重新分区OK

注意：在做以上操作的时候先备份文件
