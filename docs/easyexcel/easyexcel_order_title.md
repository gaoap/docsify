# EasyExcel 动态调整标题输出顺序

1、昨天客户对excel的cell格式输出不满意，之前用的工具，对单独的cell格式设置非常麻烦。最终发现EasyExcel 超级好用，中间遇到的问题就是客户要求动态调整excel的字段顺序排列。这个需求颇费了一些功夫才解决。随记录如下。

逻辑代码：

```java
/**
 * 指定写入的列,通过反射动态设值标题顺序
 * <p>
 * 1. 创建excel对应的实体对象 参照{@link IndexData}
 * <p>
 * 2. 使用{@link ExcelProperty}注解指定写入的列
 * <p>
 * 3. 直接写即可
 */
@Test
public void indexWrite() throws NoSuchFieldException, IllegalAccessException {
    String fileName = TestFileUtil.getPath() + "indexWrite" + System.currentTimeMillis() + ".xlsx";
    //IndexData.class为设置标题及顺序类，主要是通过动态设值注解属性“order”控制字段顺序
    Field[] fds = ReflectUtil.getFields(IndexData.class);
    for (Field fd : fds) {
        ExcelProperty ap = fd.getAnnotation(ExcelProperty.class);
        //获取代理处理器
        InvocationHandler invocationHandler = Proxy.getInvocationHandler(ap);
        // 获取私有 memberValues 属性，这里面包含注解类的所用属性状态
        Field f = invocationHandler.getClass().getDeclaredField("memberValues");
        f.setAccessible(true);
        // 获取实例的属性map
        Map<String, Object> memberValues = (Map<String, Object>) f.get(invocationHandler);
        System.out.println(fd.getName());
        for (Map.Entry me : memberValues.entrySet()) {
            System.out.println(me.getKey() + "#####" + me.getValue());
        }
        //重新设置排序字段
        if (fd.getName().equals("string")) {
            memberValues.put("order", 3);
        }
        if (fd.getName().equals("date")) {
            memberValues.put("order", 4);
        }
        if (fd.getName().equals("doubleData")) {
            memberValues.put("order", 1);
        }
       // 修改属性值
     //  memberValues.put("name", tableName);
    }
    // 这里 需要指定写用哪个class去写，然后写到第一个sheet，名字为模板 然后文件流会自动关闭
    //data()生成的是一批DemoData类数据
    EasyExcel.write(fileName, IndexData.class).sheet("模板").doWrite(data());
}
```

```java
@Getter
@Setter
@EqualsAndHashCode
public class IndexData {
    @ExcelProperty(value = "字符串标题", order = 4)
    //字段需要与数据的字段对应，这样数据输出的时候，就会按照“order”的排序输出
    private String string;
    @ExcelProperty(value = "日期标题", order = 1)
    private Date date;
    @ExcelProperty(value = "数字标题", order = 0)
    private Double doubleData;
}
```

```java
@Getter
@Setter
@EqualsAndHashCode
public class DemoData {
    //字段string与IndexData的字段string对应即可。输出excel的时候，会按照IndexData中字段的ExcelProperty注解的order属性排序。
    @ExcelProperty("字符串标题")
    private String string;
    @ExcelProperty("日期标题")
    private Date date;
    @ExcelProperty("数字标题")
    private Double doubleData;
}
```