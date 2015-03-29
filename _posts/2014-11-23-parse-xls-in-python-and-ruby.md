---
layout: post
title: 'Parse excel file in python and ruby'
date: 2014-11-23 14:33
comments: true
tags: [ruby, Python, Excel]
category: Script
---
個人比較習慣使用python，但是最近要搞Rails的東西，先用python轉再透過其他方式似乎也不是很方便。姑且把相關的資訊一起找一找。由於個人需要parse的檔案是xls格式，先以讀取excel為主做測試。

# python
python可使用`xlutils`來處理xls，包含讀檔(xlrd)，寫檔（xlwr）。新版的xlrd已經可以處理xlsx的檔案了。另外`openpyxl`似乎也可以處理xlsx的檔案。這次測試主要使用xlrd來讀取excel檔。

首先是library：

```python
import xlrd
```

讀取檔案：

```python
book = xlrd.open_workbook('/path/to/foo.xls')
```

利用index存取某個sheet的某個row，第一個sheet的index是0，第一個row的index是0：

```python
sheet = book.sheet_by_index(0)
print sheet.row(0)
```

sheet及row的個數：

```python
print book.nsheets
print sheet.nrows
```

讀取第3個row第2個element：

```python
print sheet.cell_value(2, 1)
```

透過index讀取第3個row開始的所有的row：

```python
for idx in range(2, sheet.nrows - 1):
    print sheet.row(idx)
```

利用sheet的name來iterate所有的sheet：
```python
for name in book.sheet_names():
    worksheet = book.sheet_by_name(name)
    print worksheet
```

利用sheet的index來iterate所有的sheet：

```python
for idx in range(0, book.nsheets - 1):
    worksheet = book.sheet_by_index(idx)
    print worksheet
```

# ruby
ruby中與excel相關的library可使用`roo`，可讀寫csv, excel, excelx, openoffice/libreoffice等格式。似乎還可以處理格式及公式的部分。
首先是輸入library的部分，roo處理excel是透過spreadsheet gem：

```ruby
require 'roo'
require 'spreadsheet'
```

讀取檔案：

```ruby
xls = Roo::Spreadsheet.open("/path/to/foo.xls")
```

利用index存取某個sheet的某個row，留意roo中第一個sheet的index是0，第一個row的index是1：

```ruby
sheet = xls.sheet(0)
p sheet.row(1)
```

最後一個row的index，可用來當作sheet裡row的個數：

```ruby
p sheet.last_row
```

透過index讀取第3個row開始的所有的row：

```ruby
(3..sheet.last_row).each do |idx|
    p sheet.row(idx)
end
```

Parse所有的sheet：

```ruby
xls.each_with_pagename do |name, sheet|
    p name
    p sheet.row(1)
end
```

# Reference
* [python excel](http://www.python-excel.org/)
* [Examples Reading Excel Documents Using Python's xlrd](http://www.youlikeprogramming.com/2012/03/examples-reading-excel-xls-documents-using-pythons-xlrd/)
* [openpyxl](https://openpyxl.readthedocs.org/en/latest/)
* [roo github](https://github.com/roo-rb/roo)
