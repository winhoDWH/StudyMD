##使用Jakarta POI,即HSSFWorkbook操作、导出excel
1. 常用的java两种操作excel的技术有：Jakarta POI 和 java Excel
2. Jakarta POI是一套微软格式文档的java API.由很多组件组成，其中有操作excel文件的HSSF和操作Word的HWPF；

####HSSFWorkbook
1. 对象：
    * HSSFWorkbook:     excel的工作薄
    * HSSFSheet:        excel工作表
    * HSSFRow:          excel的行
    * HSSFCell:         单元格
    * HSSFFont:         字体
    * HSSFDataFormat:   日期格式
    * HSSFHeader:       sheet（表）头
    * HSSFCellStyle:    单元格样式
2. 对象之间的关系：一个excel文件就是一个工作簿，一个工作簿有多个工作表，一个工作表由多行组成，一行又有多个单元格组成；
