**第一步，导入poi的Maven依赖**
```
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>5.2.3</version>
        </dependency>
```
**第二步，实现自定义注解**

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface ExcelColumn {
    String value() default "";

}
```
**第三步，编写导出的工具类**

```

/**
 * 类上面的字段 -> 加上  @ExcelColumn("表头") 的注解
 * @author 木池
 */
@Slf4j
public class ExcelUtil {

    public static <T> void write(List<T> objectList, OutputStream fileOut) {

        Workbook workbook = new XSSFWorkbook();
        try {
            Sheet sheet = workbook.createSheet("Sheet1");
            createHeader(sheet, objectList.get(0).getClass());
            createRows(sheet, objectList);
            workbook.write(fileOut);
        } catch (IOException e) {
            log.error("Error writing to Excel file: {}", e.getMessage());
        } finally {
            try {
                fileOut.close();
                workbook.close();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }

    /**
     * 写入Excel文件 Sheet1 中
     * @param objectList 存储对象的集合
     * @param filePath 文件路径 为空 则是桌面路径
     * @return 文件路径
     * @param <T> 泛型
     */
    public static <T> String writeToExcel(List<T> objectList, String filePath) {
        if (filePath == null || filePath.isEmpty()) {
            filePath = getDefaultDesktopPath() + "\\" + objectList.get(0).getClass().getSimpleName() + ".xlsx";
        }
        byte[] utf8Bytes = filePath.getBytes(StandardCharsets.UTF_8);
        String newFileName = new String(utf8Bytes, StandardCharsets.UTF_8);
        try (Workbook workbook = new XSSFWorkbook(); FileOutputStream fileOut = new FileOutputStream(newFileName)) {
            Sheet sheet = workbook.createSheet("Sheet1");
            Font font = getFont(sheet);
            createHeader(sheet, objectList.get(0).getClass(), font);
            createRows(sheet, objectList);
//            autoSizeColumns(sheet);
            workbook.write(fileOut);
        } catch (IOException e) {
            log.error("Error writing to Excel file: {}", e.getMessage());
        }
        return newFileName;
    }

    /**
     * 创建表头
     * @param sheet 表单对象
     * @param clazz 类对象
     * @param <T> 泛型
     */
    private static <T> void createHeader(Sheet sheet, Class<T> clazz) {
        createHeader(sheet, clazz, null);
    }

    private static <T> void createHeader(Sheet sheet, Class<T> clazz, Font headerFont) {

        CellStyle headerCellStyle = sheet.getWorkbook().createCellStyle();
        if (null != headerFont) {
            headerCellStyle.setFont(headerFont);
        }

        Row headerRow = sheet.createRow(0);
        List<String> propertyNames = new ArrayList<>();
        for (Field field : clazz.getDeclaredFields()) {
            ExcelColumn excelColumnAnnotation = field.getAnnotation(ExcelColumn.class);
            if (excelColumnAnnotation != null) {
                propertyNames.add(excelColumnAnnotation.value());
            }
        }
        int cellNum = 0;
        for (String propertyName : propertyNames) {
            Cell headerCell = headerRow.createCell(cellNum++);
            headerCell.setCellValue(propertyName);
            headerCell.setCellStyle(headerCellStyle);
        }
    }

    private static @NotNull Font getFont(Sheet sheet) {
        Font headerFont = sheet.getWorkbook().createFont();
        headerFont.setBold(true);
        headerFont.setFontHeightInPoints((short) 14);
        return headerFont;
    }

    /**
     * 创建每行的数据
     * @param sheet 表单
     * @param objectList 对象集合
     * @param <T> 泛型
     */
    private static <T> void createRows(Sheet sheet, List<T> objectList) {
        int rowNum = 1;
        for (T object : objectList) {
            Row row = sheet.createRow(rowNum++);
            int cellNum = 0;
            for (Field field : object.getClass().getDeclaredFields()) {
                ExcelColumn excelColumnAnnotation = field.getAnnotation(ExcelColumn.class);
                if (excelColumnAnnotation != null) {
                    field.setAccessible(true);
                    try {
                        Object value = field.get(object);
                        Cell cell = row.createCell(cellNum++);
                        if (value != null) {
                            if (value instanceof String) {
                                cell.setCellValue((String) value);
                            } else if (value instanceof Number) {
                                cell.setCellValue(((Number) value).doubleValue());
                            } else if (value instanceof Boolean) {
                                cell.setCellValue((Boolean) value);
                            } else {
                                cell.setCellValue(value.toString());
                            }
                        } else {
                            cell.setCellValue("");
                        }
                    } catch (IllegalAccessException e) {
                        log.error("Error accessing field: {}", e.getMessage());
                    }
                }
            }
        }
    }

    private static void autoSizeColumns(Sheet sheet) {
        for (int i = 0; i < sheet.getRow(0).getLastCellNum(); i++) {
            sheet.autoSizeColumn(i);
        }
    }

    private static String getDefaultDesktopPath() {
        return System.getProperty("user.home") + "\\Desktop";
    }

    /**
     *  kimi 不能超过 131072 字数
     * 计算Excel文件中的字数,只适合 Excel文件格式,不包含空格 和 特殊符号
     * @param filePath 文件路径
     * @return 字数
     */
    public static int getWordCount(String filePath) {
        Path excelPath = Paths.get(filePath);
        try (InputStream fis = Files.newInputStream(excelPath);
             Workbook workbook = new XSSFWorkbook(fis)) {
            int wordCount = 0;
            for (Sheet sheet : workbook) {
                for (Row row : sheet) {
                    for (Cell cell : row) {
                        if (cell.getCellType() == CellType.STRING) {
                            String cellValue = cell.getStringCellValue();
                            if (cellValue != null) {
                                wordCount += cellValue.length();
                            }
                        }
                    }
                }
            }
            return wordCount;
        } catch (IOException e) {
            return 0;
        }
    }

}

```