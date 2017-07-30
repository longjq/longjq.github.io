---
title: Elasticsearch配置文件
categories: Elastic
tags:
  - Elastic
---
### 自动加载

2.3版本开始，新增自动加载功能`--auto-reload`

```
# 设置自动加载test.conf配置文件
$ bin/logstash -f test.conf --auto-reload
```

默认每3秒监测一次配置文件，如果要修改该时间。

使用`--reload-interval <秒数>`参数

```
# 设置每4秒读取一次test.conf配置文件
$ bin/logstash -f test.conf --reload-interval 4
```

