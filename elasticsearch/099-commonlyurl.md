* 查询索引的字段信息

```shell
GET http://10.8.100.99:9200/pf_device_5min/_mapping
```

* 新建索引同时设定字段信息及分片等信息

```shell
PUT http://10.8.100.99:9200/pf_device_5min/
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    },
    "mappings": {
		"properties": {
			"id": {
				"type": "text"
			},
			"timestamp": {
				"type": "date",
				"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
			},
			"timezone": {
				"type": "text"
			},
			"period": {
				"type": "long"
			},
			"vendorname": {
				"type": "text"
			},
			"elementtype": {
				"type": "text"
			},
			"starttime": {
				"type": "date",
				"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
			},
			"dn": {
				"type": "text"
			},
			"pdn": {
				"type": "text"
			},
			"ptype": {
				"type": "text"
			},
			"vdn": {
				"type": "text"
			},
			"userlabel": {
				"type": "text"
			},
			"rs_cpu_util": {
				"type": "double"
			},
			"rs_mem_used": {
				"type": "double"
			},
			"rs_mem_total": {
				"type": "double"
			},
			"rs_mem_util": {
				"type": "double"
			},
			"rs_temperature": {
				"type": "double"
			},
			"connect_num": {
				"type": "double"
			}
		}
    }
}

```

* 已有索引新增字段

```shell
PUT	http://10.8.100.99:9200/pf_device_5min/_mapping
{
  "properties": {
    "tesssssss": {
      "type": "text"
    }
  }
}
```

* 查询所有索引的概要信息

```shell
GET http://192.168.111.20:9200/_cat/indices
```

