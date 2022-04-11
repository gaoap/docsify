**MySQL导出表结构**



 查询SQL：

```sql
SELECT 
	TABLES.table_name 表名称,
	TABLES.TABLE_COMMENT 表备注,
	COLUMN_NAME 字段名,
	COLUMN_COMMENT 备注,
	COLUMN_TYPE 数据类型,
	##DATA_TYPE 字段类型,
    ##CHARACTER_MAXIMUM_LENGTH 长度,
	IS_NULLABLE 是否为空,
	COLUMN_DEFAULT 默认值 
FROM
	INFORMATION_SCHEMA.COLUMNS COLUMNS,
	INFORMATION_SCHEMA.TABLES TABLES 
WHERE
	COLUMNS.table_schema = 'crir_test' 
	AND ( COLUMNS.table_name LIKE 'crir%' OR COLUMNS.table_name LIKE 'opf%' ) 
	AND TABLES.table_name = COLUMNS.table_name
```

然后通过客户端工具，如：Navicat等使用"Export Result"导出Excel格式即可。
