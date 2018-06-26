# 字符集
mysql支持很多字符集，在使用查询语句能看到当前数据库下支持的字符集

```
SELECT id，collat​​ion_name FROM information_schema.collat​​ions ORDER BY id;
```

一些数据包中的字符集数值索引就是指的是这里的ID