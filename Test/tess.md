#

降低 fepie 计算量
user->item
query->item

线上 tendis，u2i key 为 uid，value 为 map
q2i 的 key 为 spu_id
找出这一类组合的 value

根据 user 或 query 组合 id 在全集做过滤
tendis 原生接口只支持单 key

从 merger 查 tendis，从 merger 内部嵌入插件

UserData 后续用于存储组合 key 列表，如 q2i 的 spu_id 列表

通过插件增强 Tendis，通过插件访问实现 Tendis 存算一体
通过插件执行这两个函数

```C++
// merger 使用
// 准备统计特征插件需要的 userdata
bool PrepareUserData(itemid[], UserData* u2i_userdata, UserData* q2i_userdata);
// 判断指针非空才填充
// 使用判断，did_list 配了哪个才生效哪个

// tendis wasm 插件使用
// input: UserPreferStat bytes，序列化的 bytes
// output: 经过 userdata.keys 过滤保留后的 UserPreferStat bytes
void U2IProcess(const std::string& input, const UserData& userdata, std::string* output);

// tendis wasm 插件使用
// input: QueryPreferStat bytes
// output: 经过 userdata.keys 过滤保留后的 QueryPreferStat bytes
void Q2IProcess(const std::string& input, const UserData& userdata, std::string* output)
```

```C++
// 启动一个虚拟机加载这个插件，虚拟机提供四个接口，通过这四个接口和 Tendis 数据交互，插件每一次执行时都会分配一个 ctx_id，根据这个 id 进行数据交互
// 获取 UserData
extern "C" int32_t wasmVMGetUserData(int32_t ctx_id, const char** data, int32_t* size);
// 获取 Redis 原信息
extern "C" int32_t wasmVMGetMeta(int32_t ctx_id, const char** data, int32_t* size);
// get 出的 val
// 这里 Get 之后就可以调 Process 那个函数了，得到结果后再写回
extern "C" int32_t wasmVMGetVal(int32_t ctx_id, const char** data, int32_t* size);
// 插件处理后的回写数据给 Tendis 再通过命令返回给 merger
extern "C" void writeResponse(int32_t ctx_id, const char* data, int32_t size);
```

1. UserData 的 pb?
2. PrepareUserData 是否需要分两个接口实现，这里输出的 user_data 是需要去通过 item_id 查资源填 pb（UserData）的？
3. 目录？wasm statistic
4. Process 函数输入 (input) 的来源，user_id 和 query_id 是怎么对应？
5. 都是在 merger 内部调用的？

先通过资源文件和 item_id 加载 UserData（key_list），再传入完整 input（pb）根据 UserData 去 Process 函数过滤，需要过滤出的结果再写回 Tendis？

现在需要丰富这个逻辑，通过 item_id_list 去查资源，查出有传入 u2i_userdata 则获取 spu_id 填充，查出有传入 q2i_userdata 则获取 brandsn * 10^8 + cat3 填入到 keys，这个代码应该如何修改，似乎只需要两个 lambda 进行遍历？
