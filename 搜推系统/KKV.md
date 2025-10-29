# KKV

## hub 实现

### ReleDoc

输入

input_entities 是否包含 query_terms 和 item_ids (appcfg 中配置)。
检查是否配置 RocksDB 资源

主 key：query 词
次 key：term
value


kkv
主键 doc_id   商品 id
次键 term  统计信息（词频，位置信息）
原始特征不满足 拿不到全局的统计特征
