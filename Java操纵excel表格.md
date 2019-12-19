### JavaMaven项目操纵Excel表格

CeoAccount对象装一行数据

List<CeoAccount>存储数据的

#### 1.添加jar包依赖 poi-ppxml

```xml
<dependency>   
    <groupId>org.apache.poi</groupId>    
    <artifactId>poi-ooxml</artifactId>    
    <version>4.0.1</version>
</dependency>
```

#### 2.从excel导出数据

##### 1.创建io流,WorkBook

```java
InputStream is=null;
Workbook workBook=null;
```

##### 2.判断excel表格类型(xlsx,xls)

```java
is=new FileInputStream(inputPath);
if(type.equalsIgnoreCase("xlsx")){ 
    workBook=new XSSFWorkbook(is);}
else {    workBook=new HSSFWorkbook(is);}
```

##### 3.获取表格数(sheet)并分层遍历,获取行(row)单元格(ceo)的数据

```java
for(int numSheet=0;numSheet<workBook.getNumberOfSheets();numSheet++)
{    
    Sheet sheet=workBook.getSheetAt(numSheet);    
    if(sheet==null){        
        continue;    }    
    System.out.println(sheet.getLastRowNum());    
    //第零行数据不读取,列名    
    for(int rowNum=1;
        rowNum<=sheet.getLastRowNum();rowNum++){     
        Row row=sheet.getRow(rowNum);       
        account=new CeoAccount();        
        account.setSymbol(row.getCell(0).toString().toUpperCase());        
        account.setAccount(row.getCell(1).toString());       
        account.setApiKey(row.getCell(2).toString());        
        account.setSecKey(row.getCell(3).toString());       
        list.add(account);    }}
```

#### 3.将数据导出到excel中

##### 1.建立文件流

```JAVA
File file;
OutputStream outputStream=null;
XSSFWorkbook wb=null;
//创建XSSFBook对象
 wb=new XSSFWorkbook();
```

##### 2写入第一行的列名

```java
//建立新的sheet对象
XSSFSheet xssfSheet = wb.createSheet("CEO账户资金表");
//在sheet创建第一行,参数为行索引(excel的行)
XSSFRow row = xssfSheet.createRow(0);
row.createCell(0).setCellValue("账户");
row.createCell(1).setCellValue("交易对");
row.createCell(2).setCellValue("可用");
row.createCell(3).setCellValue("冻结");
row.createCell(4).setCellValue("总额");
```

##### 3.正式填入数据

```java
int rowNum=1;
for(int i=0;i<list.size();i++){   
CeoAccount account = list.get(i);   
    List<Map<String, Double>> capitalList = account.getCapitalList();   
    for (int j=0;j<capitalList.size();j++){       
        XSSFRow row1 = xssfSheet.createRow(rowNum);       
        Map<String, Double> stringDoubleMap = capitalList.get(j);       
        row1.createCell(0).setCellValue( account.getAccount());       
        row1.createCell(1).setCellValue(account.getSymbol());       
        row1.createCell(2).setCellValue(stringDoubleMap.get("balance"));       
        row1.createCell(3).setCellValue(stringDoubleMap.get("lock"));        
    row1.createCell(4).setCellValue(stringDoubleMap.get("balance")+stringDoubleMap.get("lock"));        
        rowNum++;    }}
file=new File(outputPath);
```

##### 4.填入file

```java
 file=new File(outputPath);   
if(!file.exists()){        
    file.createNewFile();    }    
outputStream=new FileOutputStream(file);   
if(wb!=null){        
    wb.write(outputStream);    }} 
catch (Exception e) {    
    e.printStackTrace();}finally {    
    System.out.println("写入成功");    
    try {       
        if(wb!=null){            
            wb.close();        }       
        if(outputStream!=null){          
            outputStream.close();        }    
    } catch (IOException e) {      
        e.printStackTrace();    }}
```