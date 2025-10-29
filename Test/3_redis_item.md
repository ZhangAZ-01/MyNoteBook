现在根据这四个文件，我希望将 hmget 这种存在多个 subkey 的请求，我希望拆成多条 Redis命令请求，每一条为hmget，每一条的subkey 为一个，如何修改代码可以拆分，并在 MergeResult 合并拆分后分别查询的结果，帮我检查我已经修改的部分并补全MergeResult


bool SessionBase::PrepareRedisContext() {
  auto* ctx = redis_context();
  const auto& unified = appcfg()->basic()->unified;

  for (auto ftype : flow_handler_.flows()) {
    if (FlowDefine::is_special_flow(ftype)) {
      continue;
    }

      // 调用 Hub.Score 接口时，从 appcfg 中获取 redis identifiers
      for (const auto& redis_identifier : flow_handler_.mutable_flow_args(ftype)->redis_identifier_list()) {
        auto fit = appcfg()->redis_items()->redis_items.find(redis_identifier);
        if (fit != appcfg()->redis_items()->redis_items.end()) {
          RedisContext::Item item;
          const auto& redis_item_config = fit->second;
          // For grey subkeys.
          if (redis_item_config.redis_operation_type == REDIS_OPERATION_TYPE::REDIS_OPERATION_TYPE_HM_GET &&
              redis_item_config.subkeys.size() > 1) {
            for (const auto& subkey : redis_item_config.subkeys) {
              RedisContext::Item sub_item;
              sub_item.identifier = redis_item_config.identifier + "." + subkey;
              sub_item.origin_identifier = redis_item_config.identifier;
              sub_item.subkey = subkey;
              std::string broker_id = redis_item_config.broker_id;
              sub_item.label = appcfg()->GetRedisLabel(broker_id);
              sub_item.key = GetRedisKey(EntityHelper::NameToKeyType(redis_item_config.key_type));
              sub_item.timeout_ms = appcfg()->GetRedisTimeout();
              sub_item.redis_operation_type = REDIS_OPERATION_TYPE::REDIS_OPERATION_TYPE_HM_GET;
              ctx->AddItem(sub_item);
            }
          } else {
            item.identifier = redis_item_config.identifier;
            item.prefix = redis_item_config.prefix;
            item.compress_type = redis_item_config.compress_type;
            item.redis_operation_type = redis_item_config.redis_operation_type;
            item.key = GetRedisKey(EntityHelper::NameToKeyType(redis_item_config.key_type));
            // 配置 subkey
            if (item.redis_operation_type == REDIS_OPERATION_TYPE::REDIS_OPERATION_TYPE_HM_GET) {
              if (!redis_item_config.subkeys.empty()) {
                item.hm_fields = redis_item_config.subkeys;
                oxygen::Dedup(&item.hm_fields);
              } else {
                LOG(ERROR)
                    << "The configuration of `redis_operation_type` and `subkeys` was not matched, redis_identifier="
                    << item.identifier << ", request_id=" << unique_request_id();
              }
            }
            item.timeout_ms = appcfg()->GetRedisTimeout();
            item.label = appcfg()->GetRedisLabel(redis_item_config.broker_id);
            if (!redis_item_config.grey.broker_id.empty() &&
                Random::GetInstance().GenerateFloat() < redis_item_config.grey.rate) {
              item.label = appcfg()->GetRedisLabel(redis_item_config.grey.broker_id);
            }
            ctx->AddItem(item);
          }

        } else {
          LOG(ERROR) << "invalid redis_identifier=" << redis_identifier << ", request_id=" << unique_request_id()
                     << ", api=" << api() << ", uid=" << request_base().uid();
        }
      }
    }

  return true;
}


auto MergeResult() {
  return [](oxygen::funflow::Source<FlowRedisData> source) -> merger_result_t<FlowData> {
    FlowData data;
    double max_cost_time = 0;
    SessionBase* session = nullptr;

    // 用于合并 origin_identifier -> vector<hm_value>
    phmap::flat_hash_map<std::string, std::vector<std::string>> merged_hm_values;
    phmap::flat_hash_map<std::string, size_t> merged_value_byte_size;
    phmap::flat_hash_map<std::string, std::string> merged_value;

    for (const auto& it : source.outputs()) {
      data.session = it.session;
      session = it.session;

      const auto* item = it.item;
      std::string origin_identifier = item->origin_identifier.empty()
                                          ? std::string(item->identifier)
                                          : item->origin_identifier;

      ReportMetric(METRIC_TYPE_REDIS_MISS_RATE,
                   {{"api", it.session->metrics_api()},
                    {"identifier", origin_identifier},
                    {"status", item->value_byte_size == 0 ? "missed" : "returned"}},
                   1);

      if (item->value_byte_size > 0) {
        ReportMetric(METRIC_TYPE_REDIS_LENGTH,
                     {{"api", it.session->metrics_api()}, {"identifier", origin_identifier}},
                     item->value_byte_size);
      }

      if (item->redis_operation_type == REDIS_OPERATION_TYPE_HM_GET) {
        merged_hm_values[origin_identifier].push_back(item->value);
        merged_value_byte_size[origin_identifier] += item->value_byte_size;
      }

      if (max_cost_time < it.cost_time) {
        max_cost_time = it.cost_time;
      }
    }

    for (const auto& [origin_id, values] : merged_hm_values) {
      auto* origin_item = session->redis_context()->MutableItem(origin_id);
      if (origin_item) {
        origin_item->hm_values = std::move(values);
        origin_item->value_byte_size = merged_value_byte_size.at(origin_id);
        origin_item->value.clear();
        for (const auto& v : origin_item->hm_values) {
          origin_item->value += v;
        }
      }
    }

    session->flow_handler()->AddTime(
        FlowDefine::TypeToName(session->flow_handler()->current_flow_type()),
        max_cost_time);

    return data;
  };
}
