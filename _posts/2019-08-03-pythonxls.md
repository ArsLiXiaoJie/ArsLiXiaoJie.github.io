---
layout: post
title:  "python 操作xlsx"
date:   2019-08-03 18:31:01 +0800
categories: python
tag: python
typora-root-url: ../../arslixiaojie.github.io
---

接到这么一个需求：根据给出的一些key值，在某个xlsx文件中找出这些key值对应的行，放到另一个xlsx中，从原xlsx文件中删除。

```python
# -*- coding: utf-8 -*-

import sys

reload(sys) 
sys.setdefaultencoding('utf-8')

#加上一些相对路径，因为下面需要import的一些模块在不在当前路径下
sys.path.append("../")
sys.path.append("../def/")

import config
import xlrd, xlwt		#pip install xlrd / xlwt
import codecs, os, json
import win32com.client  # pip install pywin32
import pywintypes

project_path = "E:/project/xxx/"

#移动到的目标xls，这里用xls格式是因为xlwt库不支持xlsx，用xlsx格式会导致生成的文件打不开
deprecated_file = "xxx.xls"

#copyExcel，在这里用来删除原xlsx文件中的某行数据
class copyExcel:
    def __init__(self,filename = None):
        self.workApp = win32com.client.Dispatch('Excel.Application')
        if filename:
            self.filename = filename
            self.workbook = self.workApp.Workbooks.Open(filename)
        else:
            self.workbook = self.workApp.Workbooks.Add()
            self.filename = ''
 
    def deleteRow(self,sheet,row):
        sht = self.workbook.Worksheets(sheet)
        sht.Rows(row).Delete()
 
    def save(self, newfile=None):
        if newfile:
            self.filename = newfile
            self.workbook.SaveAs(newfile)
        else:
            self.workbook.Save()
 
    def close(self):
        self.workbook.Close(SaveChanges = 0)
        del self.workApp
 
    def getCell(self,sheet,row,col):
        sht = self.workbook.Worksheets(sheet)
        return sht.Cells(row,col).Value
 
    def setCell(self,sheet,row,col,value):
        sht = self.xlBook.Worksheets(sheet)
        sht.Cells(row, col).Value = value
    def getRowNum(self,sheet):
        sht = self.workbook.Worksheets(sheet)
        return sht.usedrange.rows.count


#查找和移动
def find_and_move( deprecated_map ):
    #def_module是我们用来存放xlsx中sheet的配置
	def_module = __import__("xxx", globals(), locals(), [])
	if def_module is None:
		raise Exception("no def file %s" % def_name)

	#输入源文件，用xlrd来读取原xlsx文件的内容
	input_file = "source.xlsx"
	infile = xlrd.open_workbook(input_file)

	#目标xls文件
	deprecated_workbook = None
	if os.path.exists(deprecated_file):
        #如果本来就存在这个文件，需要先把这个文件的内容读出来
		deprecated_workbook = xlrd.open_workbook(deprecated_file)
	workbook = xlwt.Workbook(encoding='utf-8')

	#用来查找和删除，原xlsx文件，需要删除行
	temp = copyExcel("source.xlsx")

	#全部sheet
	sheet_names = def_module.sheet_names

	#移动的总条数
	del_total_count = 0

	try:
		for k in sheet_names:
			sheetname = sheet_names[k]
			sheet = infile.sheet_by_name(sheetname)

			if sheet == None:
				print("sheet %s is not exist!" % k)
				continue

			print(u"\n正在检查sheet：%s" % sheetname)
			#新excel也按原来的sheet区分

			booksheet = workbook.add_sheet(sheetname, cell_overwrite_ok=True)
			write_index = 1
			#把旧的数据拷过来，因为xlwt并不能修改xls文件，只能重写
			if deprecated_workbook:
				print(u"\n正在拷贝旧数据...\n")
				tempsheet = deprecated_workbook.sheet_by_name(sheetname)
				tempRow = tempsheet.nrows
				write_index = tempRow
				for i in xrange(0,tempRow):
					tempdata = tempsheet.row_values(i)
					for j in range(len(tempdata)):
						booksheet.write(i, j, tempdata[j])
				print(u"旧数据拷贝结束...\n")
			else:
				write_index = 1
				colname = ["简称","简体中文","繁体中文","English","Russian","Japanese","Spanish","Thai","Hindi","Português","Turkish","Korean"]
				for i in range(len(colname)):
					booksheet.write(0, i, colname[i])

			usheetname = u"" + sheetname
			rowNum = temp.getRowNum(usheetname)
			print(u"[[%s]] 删除前条数 : %d" % (usheetname, rowNum-1))
            #copyExcel里面用到的库，下标是从1开始的，而xlrd和xlwt下标是从0开始，第一行只是列名
			row = 2
			del_count = 0
			while row <= rowNum:
				lab_key = temp.getCell(usheetname,row,1)
				if deprecated_map.get(lab_key) == 1:
					print(u"---正在移动：%s" % lab_key)
                    #row-1+del_count ，源文件在不断地删除行，行下标会发生变化，而xlrd读出来的内容下标不变，所以需要+del_count
					rowdata = sheet.row_values(row-1+del_count)

					#写到新excel
					for j in range(len(rowdata)):
						booksheet.write(write_index, j, rowdata[j])

					write_index += 1
					del_count += 1

					print(u"---正在删除：%s" % lab_key)
					temp.deleteRow(usheetname,row)
					row -= 1
					rowNum -=1
				row += 1

			del_total_count += del_count
			print(u"[[%s]] 删除 %d 条" % (usheetname, del_count))
			print(u"[[%s]] 删除后条数 : %d" % (usheetname, temp.getRowNum(usheetname)-1))
			#xlwt只能保存xls格式的文件，xlsx会报错
			workbook.save(deprecated_file)
			temp.save()
			
	except pywintypes.com_error as e:
		pass
	finally:
		temp.close()

	print(u"\n查找移动结束，一共移动了 %d 条！！\n" % del_total_count)


if __name__ == '__main__':
    #读取出需要移动的key值
	f = codecs.open("json.txt","r","utf-8")
	content = f.read()
	f.close()
	deprecated = json.loads(content)

	print(u"全部文案总数量：%d" % deprecated["count"])

	if deprecated["count"] == 0:
		print(u"没有弃用，直接结束")

	else:
		deprecated_map = {}
		for lab in deprecated["list"]:
			deprecated_map[lab] = 1

		print(u"读取json文件，弃用总条数 = %d" % len(deprecated["list"]))
		find_and_move(deprecated_map)

		print(u"结束！！")
```

