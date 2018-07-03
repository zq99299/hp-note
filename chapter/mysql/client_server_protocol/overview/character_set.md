# 字符集
> https://dev.mysql.com/doc/internals/en/character-set.html
mysql支持很多字符集，在使用查询语句能看到当前数据库下支持的字符集

```
SELECT id, collation_name FROM information_schema.collations ORDER BY id;
```

一些数据包中的字符集数值索引就是指的是这里的ID