* 新增表字段

```mysql
	alter table pf_port_5min add ifinoctets_old varchar(60) comment '入字节数数字Bytes';
	alter table pf_port_5min add ifoutoctets_old varchar(60) comment '出字节数数字Bytes';
```

* 新增字段时提示：

```
 Row size too large. The maximum row size for the used table type, not counting BLOBs, is 8126. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs;
```

[MySQL出现The maximum row size for the used table type, not counting BLOBs, is 8126.错误](https://blog.csdn.net/dbagaoshou/article/details/52484964)

暂时将2个字段修改double进行保存

```mysql
	alter table pf_port_5min MODIFY ifinoctets_old double comment '入字节数数字Bytes';
	alter table pf_port_5min add ifoutoctets_old double comment '出字节数数字Bytes';
```

