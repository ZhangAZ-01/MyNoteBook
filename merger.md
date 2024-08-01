# merger3

## common

## define.h

用于在 merger3 中定义常量、类型别名、枚举、结构体和一些实用的内联函数。

```C++
/************************************************************************/
/* Type aliases */
/************************************************************************/
// Hub 标签到模型信息的映射表
using models_t = phmap::flat_hash_map<std::string, phmap::flat_hash_set<std::string>>;
// 分片
using slice_t = std::pair<size_t, size_t>; // 表示一个范围或分片，例如开始和结束的索引
using slices_t = std::vector<slice_t>; // 存储多个范围或分片信息

using pbssmap_t = google::protobuf::Map<std::string, std::string>; // 存储键值对，用于序列号和传输结构化数据
using ssmap_t = phmap::flat_hash_map<std::string, std::string>;

#define UNIFIED_USER_PLACEHOLDER_ENTITY_TYPE 30
```

```C++
/************************************************************************/
/* TokenPoolType */
/************************************************************************/
struct TokenPoolType {
  enum Type {
    RELEVANCE = 0,  //
    MAX = 1,        //
  };

  inline static const std::array<std::string, MAX> NAMES = {
      "relevance",  //
  };

  inline static const std::string& TypeToName(Type t) { return NAMES[t]; }
  inline static Type NameToType(const std::string& name) {
    for (size_t i = 0; i < MAX; ++i) {
      if (name == NAMES[i]) {
        return (Type)i;
      }
    }
    return RELEVANCE;
  }
};
```

## global_object

- 管理全局配置和资源

- struct Gflags: 存储 gflags 的值，避免其他模块使用时到处传值。

- struct GlobalObject: 全局对象容器，管理多个系统关键资源。

  - TimerManager: 用于全局的时间事件管理。

  - token_pools: 包含固定大小令牌池用于流量控制，token_pool 方法可以获取特定类型的令牌池。

  - Redis Manager: 管理异步 Redis 连接和操作，用于应用配置和统一配置的数据获取。

## goods_image_id

goods_image_id = {goods_id}_{image_id}

用于解析格式为 {goods_id}_{image_id} 的字符串，并能够生成这种形式的字符串。

- ExtractGoodsImageId: 从完整的商品图片 ID 中提取商品 ID 和图片 ID。

- ExtractGoodsId: 仅从商品图片 ID 中提取商品 ID。

- ExtractImageId: 仅从商品图片 ID 中提取图片 ID。

- GenerateGoodsImageId: 在给定商品 ID 的前提下，生成包含随机图片 ID 的字符串。

## time

### time_feature

#### 成员变量

- struct tm：用于保存时间和日期的结构体，包含年、月、日、小时、分钟、秒等元素。

- time_t time_: 存储 Unix 时间戳，表示从 1970 年 1 月 1 日以来的秒数。

#### 常量定义

- 时间段

- 季节

- 季度

#### 成员方法

- SetCurrentTime(): 设置当前时间。

- SetTime(const char* s): 设置特定的时间，通常用于单元测试。

- EpochSeconds(): 返回 Unix 时间戳。

- HourOfDay(): 返回一天中的小时数（0-23）。

- DayOfWeek(): 返回一周中的天数（0 表示周日到 6 表示周六）。

- DayOfWeek2(): 返回一周中的天数，从 1（周一）到 7（周日）。

- DayOfMonth(): 返回月份的天数（0-30）。

- DayOfYear(): 返回年中的天数（0-365）。

- IsWeekend(): 返回是否是周末。

- WeekOfYear(): 返回年中的周数（0-52）。

- TimeOfDay(): 返回代表当前时间的一天中时间段的字符串。

- Quarter(): 返回当前季度的字符串。

- Season(): 返回当前季节的字符串。

### time_collector

监控和记录特定代码段的执行时间。

- Reset()：重置收集器，清除所有已收集的数据。

- AddTime()：记录一个时间段，以功能类型 FType、名称和时间长度。

- String()：返回所有收集到的时间数据的字符串表示。

- EnterPhase() 和 LeavePhase()：标记一个时间测量阶段的开始和结束，可以通过索引或阶段名称来管理。

- Traverse()：允许外部函数或 lambda 表达式访问收集到的所有时间数据，进行遍历操作。

## tess2_utils

- 用于处理与 Tess2 系统相关的数据转换和缓冲操作。

- 这些组件包括帮助函数、数据读写器和缓冲区管理器，旨在提供高效的数据处理和转换功能。

- Tess2Helper

  - 提供检验数据有效性的静态方法。

  - 实现从旧版本数据到新版本的转换。

  - 提供了输出实体列表调试日志的方法。

- IsZeroCopyScoreValid: 验证 ZeroCopyBuffer 对象偏移量是否结束，数据大小是否符合期望。

- IsZeroCopyDocValid: 检查 EntityList 是否具有有效的文档偏移和实体偏移。

- AbtFromV3: 将版本 3 的特定字段映射到版本 2 的格式中。

## config

## launcher

### async_content.h

### launcher.cc

### launcher.h

## 启动服务

### 本地

terminal1:  ./server_search --logtostderr --mock_tracer  --server_addr=

terminal2:  export CARBON_METRICS_HTTP_ADDR=127.0.0.1:35881
            ./client_search  --logtostderr --api=search --dp=1 --item_size=5 --timeout=200 --qps=1 --domain=

terminal3:  ./server_featurehub --logtostderr --server_hub_addr=

### 补充

查看端口是否被占用 netstat -vanp tcp | grep 端口号

编译某个服务 ./build.sh --version v3 --scene featurehub
