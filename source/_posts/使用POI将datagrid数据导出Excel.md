---
title: 使用POI将datagrid数据导出Excel
date: 2017-09-11 20:25:28
tags: java
---
> 这个功能也是因公司项目要求，之前毕设的时候有做过一个将数据导出Excel的功能，不过时隔一年，全然忘了，也不知道是不是用的POI，这次用的POI来实现将datagrid的数据导成Excel表，其实看了下文档挺简单的。<!--more-->

### 步骤：
1.将数据从前端传到后台，或者通过后台操作数据库获取需要导出的数据
2.创建工作簿、标题、列名、数据列，还有样式（核心部分）
3.使用流来输出

### 前端操作
一开始我是将前端datagrid的数据通过ajax传递到后台，然后打印出，这个时候你会遇到一个问题：在使用流导出的时候你不能使用OutputStream来操作，只能通过FileOutputStream指定文件路径来操作，这样就有很多局限性了。查了资料，然后采用的是“location.href”的方式来传递，不过这里要注意“location.href”的传递方式是GET方式，而GET对URL的长度是有限制的，好像chrome的限制是7万多字节，这样的话如果数据量超了就会报错，这也是一个局限性，所以最后只能使用通过后台操作数据库来获取数据：
```
window.location.href="/report/XXX.htm?........."
```
后台使用Controller来捕获请求并操作数据库获取相关数据

### 核心部分：
我的思路是将这个核心部分封装成一个公用的工具类之类的，因为你指不定有多少个地方需要导出Excel，所以这样就额可以备用。然后通过实例化这个对象，再通过调用相关方法来创建Excel导出的模板。当然实例化时需要的参数有：标题，列名，数据。

#### 创建Excel的大致模板（即实例化出模板）
依次就是创建工作簿、创建一个sheet、创建表格标题行、定义标题样式和数据行样式：
```
HSSFWorkbook workbook = new HSSFWorkbook();//创建工作簿
HSSFSheet sheet = workbook.createSheet(title);//创建一个sheet
HSSFRow row = sheet.createRow(0);//创建表格标题行
HSSFCell cellTitle = row.createCell(0);
//创建单元格，并设置值表头
HSSFCellStyle columnStyle = this.getColumnTopStyle(workbook);//列名样式
HSSFCellStyle style = this.getStyle(workbook);//数据样式
```
上面代码中我通过两个方法来分别设置了标题样式和数据行的样式，设置样式代码：
```
	/**
     * 列名样式
     *<br/>
     * 加粗，字体大小，字体，居中
     * @param workbook
     * @return
     */
    public HSSFCellStyle getColumnTopStyle(HSSFWorkbook workbook){
        HSSFFont font = workbook.createFont();
        font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);//加粗
        font.setFontHeightInPoints((short) 12);//字体大小
        font.setFontName("宋体");
        HSSFCellStyle style = workbook.createCellStyle();
        style.setFont(font);
        style.setAlignment(HSSFCellStyle.ALIGN_CENTER);//水平居中
        style.setVerticalAlignment(HSSFCellStyle.ALIGN_CENTER);//垂直居中
        return style;
    }

    /**
     * 设置数据样式
     * <br/>
     * 水平居左，垂直居中
     * @param workbook
     * @return
     */
    public HSSFCellStyle getStyle(HSSFWorkbook workbook){
        HSSFCellStyle style = workbook.createCellStyle();
        style.setAlignment(HSSFCellStyle.ALIGN_RIGHT);
        style.setVerticalAlignment(HSSFCellStyle.ALIGN_CENTER);
        return style;
    }
```
#### 设置标题
设置标题时我们要将单元格合并，注意的是poi的excel单元格不管是行还是列都是从0开始的：
```
//设置标题
sheet.addMergedRegion(new CellRangeAddress(0,0,0,(rowsName.length-1)));
cellTitle.setCellStyle(columnStyle);
cellTitle.setCellValue(title);
```
#### 设置列名
这里我们要计算出我们一共需要几列，通过后台传过来实例化的“列名”数组可以计算出，然后依次将数组的值设置进去：
```
int column = rowsName.length;//所需的列数
HSSFRow rowName = sheet.createRow(1);
//设置列名
for (int i=0; i<column; i++){
    HSSFCell cellName = rowName.createCell(i);
    cellName.setCellType(HSSFCell.CELL_TYPE_STRING);//数据类型
    HSSFRichTextString text = new HSSFRichTextString(rowsName[i]);
    cellName.setCellValue(text);
    cellName.setCellStyle(columnStyle);
}
```
#### 设置数据
传过来的是一个数组项的List，所以我们通过两个for循环来分别将每一项数据设置进相应的位置：
```
//设置数据
for (int i=0; i<list.size(); i++){
    Object[] obj = list.get(i);
    HSSFRow dataRow = sheet.createRow(i+2);
    for (int j=0; j<obj.length; j++){
        HSSFCell cell = dataRow.createCell(j,HSSFCell.CELL_TYPE_STRING);
        cell.setCellValue(obj[j].toString());
        cell.setCellStyle(style);
    }
}
```
#### 设置列宽自适应
通过getBytes().length来获取长度
这个查看下面的全部代码那里，思路就是先获取出每个单元格的字节长度（有个默认的），然后每列种每个单元格数据的字节长度跟默认的长度比较，拿最长的，最后sheet.setColumnWidth(i, (colWidth+2)*256);设置整列的长度。

### Controller层
就是封装一些创建要用的数据：标题、列名、数据
```
String title= DateUtils.format(new Date(), "yyyy-MM-dd")+"股票统计";
String[] rowsName = new String[]{"代码","股票","股数","盈亏","市值"};
List<Object[]> objList = new ArrayList<>();
Object[] obj;
for (StockStat stockStat : list){
    obj = new Object[5];
    obj[0] = stockStat.getStockCode();
    obj[1] = stockStat.getStockName();
    obj[2] = stockStat.getStockVolume();
    obj[3] = stockStat.getIncome();
    obj[4] = stockStat.getPrice();
    objList.add(obj);
}
ExportReportTemplate template = new ExportReportTemplate(title, rowsName, objList);
OutputStream out = null;
try {
    //输出excel
    out = resp.getOutputStream();
    resp.reset();
    resp.setContentType("application/vnd.ms-excel;charset=UTF-8");
    String fileName = URLEncoder.encode(title,"UTF-8");
    resp.setHeader("Content-disposition", "attachment;filename="+fileName+".xls");
    template.export(out);
    out.close();
} catch (IOException e) {
    logger.error("导出excel表出错", e);
} finally {
    try {
        out.close();
    } catch (IOException e) {
        logger.error("关闭OutputStream流出错", e);
    }
}
```
这里OutputStream输出的时候要设置好一些头文件信息，然后最后一定要关闭流，不然万一中途出现异常，会导致内存溢出。

### ExportReportTemplate模板类全部代码：
```
public class ExportReportTemplate {
    private final static Logger logger = LoggerFactory.getLogger(ExportReportTemplate.class);

    //标题
    private String title;
    //列名
    private String[] rowsName;
    //数据
    private List<Object[]> list = new ArrayList<>();

    public ExportReportTemplate(String title, String[] rowsName, List<Object[]> list) {
        this.title = title;
        this.rowsName = rowsName;
        this.list = list;
    }

    public void export(OutputStream out){
        HSSFWorkbook workbook = new HSSFWorkbook();//创建工作簿
        HSSFSheet sheet = workbook.createSheet(title);//创建一个sheet
        HSSFRow row = sheet.createRow(0);//创建表格标题行
        HSSFCell cellTitle = row.createCell(0);
        //创建单元格，并设置值表头
        HSSFCellStyle columnStyle = this.getColumnTopStyle(workbook);//列名样式
        HSSFCellStyle style = this.getStyle(workbook);//数据样式

        //设置标题
        sheet.addMergedRegion(new CellRangeAddress(0,0,0,(rowsName.length-1)));
        cellTitle.setCellStyle(columnStyle);
        cellTitle.setCellValue(title);

        int column = rowsName.length;//所需的列数
        HSSFRow rowName = sheet.createRow(1);

        //设置列名
        for (int i=0; i<column; i++){
            HSSFCell cellName = rowName.createCell(i);
            cellName.setCellType(HSSFCell.CELL_TYPE_STRING);//数据类型
            HSSFRichTextString text = new HSSFRichTextString(rowsName[i]);
            cellName.setCellValue(text);
            cellName.setCellStyle(columnStyle);
        }

        //设置数据
        for (int i=0; i<list.size(); i++){
            Object[] obj = list.get(i);
            HSSFRow dataRow = sheet.createRow(i+2);
            for (int j=0; j<obj.length; j++){
                HSSFCell cell = dataRow.createCell(j,HSSFCell.CELL_TYPE_STRING);
                cell.setCellValue(obj[j].toString());
                cell.setCellStyle(style);
            }
        }

        //让列宽自适应
        for (int i=0; i<column; i++){  //列
            int colWidth = sheet.getColumnWidth(i)/256;
            //拿出每一列数据的长度
            for (int j=1; j<=sheet.getLastRowNum(); j++){
                HSSFRow currentRow;
                if(sheet.getRow(j)==null){
                    currentRow = sheet.createRow(j);
                }else {
                    currentRow = sheet.getRow(j);
                }
                if(currentRow.getCell(i)!=null){
                    HSSFCell currentCell = currentRow.getCell(i);
                    int length = currentCell.getStringCellValue().getBytes().length;
                    if (colWidth<length){
                        colWidth=length;
                    }
                }
            }
            sheet.setColumnWidth(i, (colWidth+2)*256);
        }
        try {
            workbook.write(out);
        } catch (IOException e) {
            logger.error("导出Excel表出错", e);
        }
    }

    /**
     * 列名样式
     *<br/>
     * 加粗，字体大小，字体，居中
     * @param workbook
     * @return
     */
    public HSSFCellStyle getColumnTopStyle(HSSFWorkbook workbook){
        HSSFFont font = workbook.createFont();
        font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);//加粗
        font.setFontHeightInPoints((short) 12);//字体大小
        font.setFontName("宋体");
        HSSFCellStyle style = workbook.createCellStyle();
        style.setFont(font);
        style.setAlignment(HSSFCellStyle.ALIGN_CENTER);//水平居中
        style.setVerticalAlignment(HSSFCellStyle.ALIGN_CENTER);//垂直居中
        return style;
    }

    /**
     * 设置数据样式
     * <br/>
     * 水平居左，垂直居中
     * @param workbook
     * @return
     */
    public HSSFCellStyle getStyle(HSSFWorkbook workbook){
        HSSFCellStyle style = workbook.createCellStyle();
        style.setAlignment(HSSFCellStyle.ALIGN_RIGHT);
        style.setVerticalAlignment(HSSFCellStyle.ALIGN_CENTER);
        return style;
    }
}
```