# merger3

## 启动服务

### 本地

terminal1:  ./server_XXX --logtostderr --mock_tracer  --server_addr=

terminal2:  export CARBON_METRICS_HTTP_ADDR=127.0.0.1:35881
            ./client_XXX  --logtostderr --api=XXX --dp=1 --item_size=5 --timeout=200 --qps=1 --domain=

terminal3:  ./server_featurehub --logtostderr --server_hub_addr=

### 补充

查看端口是否被占用 netstat -vanp tcp | grep 端口号

编译某个服务 ./build.sh --version v3 --scene featurehub

10.189.108.33

logtostderr：将日志输出到 stderr。
api：选择场景
dp：分发策略
item_size：要求的 item size。
timeout：超时时间（以毫秒为单位）
qps：每秒查询次数
domain：addr
mock_tracker：模拟追踪器

## common

### define.h

用于在 merger3 中定义常量、类型别名、枚举、结构体和一些实用的内联函数。

### global_object

- 管理全局配置和资源

- struct Gflags: 存储 gflags 的值，避免其他模块使用时到处传值。

- struct GlobalObject: 全局对象容器，管理多个系统关键资源。

  - TimerManager: 用于全局的时间事件管理。

  - token_pools: 包含固定大小令牌池用于流量控制，token_pool 方法可以获取特定类型的令牌池。

  - Redis Manager: 管理异步 Redis 连接和操作，用于应用配置和统一配置的数据获取。

### goods_image_id

goods_image_id = {goods_id}_{image_id}

用于解析格式为 {goods_id}_{image_id} 的字符串，并能够生成这种形式的字符串。

- ExtractGoodsImageId: 从完整的商品图片 ID 中提取商品 ID 和图片 ID。

- ExtractGoodsId: 仅从商品图片 ID 中提取商品 ID。

- ExtractImageId: 仅从商品图片 ID 中提取图片 ID。

- GenerateGoodsImageId: 在给定商品 ID 的前提下，生成包含随机图片 ID 的字符串。

### time

### tess2_utils

- 用于处理与 Tess2 系统相关的数据转换和缓冲操作。

- AbtFromV3: 将版本 3 的特定字段映射到版本 2 的格式中。

## config

配置文件

这些配置被组织在一个统一的全局配置类 GlobalConfig 中，并通过 GlobalConfigNotifier 类实现了在系统运行时的初始化和更新。

区分本地和线上

- global_config 全局配置信息
- meta 所有数据配置结构
- holder 组织配置，解析
区分本地和线上

- global_config 全局配置信息
- meta 所有数据配置结构
- holder 组织配置，解析

## entity

数据团队生成数据，生成到 Redis 或 RocksDB，Entity 用来存储数据。

将数据序列化后，将二进制数据填入到 entity 中。

数据团队生成数据，生成到 Redis 或 RocksDB，Entity 用来存储数据。

将数据序列化后，将二进制数据填入到 entity 中。

### builder

helper：选择哪个级别的 entity。

builder：定义构建 entity 的基本配置，可以根据具体的业务场景实现不同的 EntityBuilder 和 DocBuilder。

list_entity：解析给定的 JSON 数据并填充 QueryProcessInfo 对象。

### entity_builder

level 0: 用户侧数据，请求级别的数据，比如用户、上下文等；

level 1: 商品侧数据，排序级别的数据，比如商品、品牌、档期等；

KVEntityBuilder：从 Redis 中获取。

ValuelessEntityBuilder：缺少 value 的 entityBuilder

QPEntityBuilder：解析复杂查询，通过查询结果构建

SimpleContextEntityBuilder：填充最初版本 context protobuf

NestedContextEntityBuilder：填充新版本 protobuf

ValuedEntityBuilder：构建 value 来自于内部计算或者模型计算的 Entity

ListEntityBuilder：用于构建 ListEntity 类型

## launcher

初始化以及启动相关服务。

## session

用于管理处理请求和生成响应的整个流程。
它是一个通用的会话管理类，可以在具体的业务逻辑中通过指定 Request 和 Response 类型来实例化。
这个类的主要功能包括接收请求、处理请求、管理流程、生成响应、记录日志等。

AdaptInputAbt：兼容 abt 参数，进行参数适配以及过滤无关参数。

OnRecvRequest() 方法是接收到请求后的主入口，负责初始化一些必要的字段（如 unique_request_id_），适配请求参数，并调用具体的请求处理逻辑。

OnSendResponse() 方法在流程结束后调用，用于生成最终的响应。它还会记录一些相关的日志，并上报相关的监控指标。

RunFlow() 方法启动整个流程的执行。它会初始化一个流程对象，并开始执行流程中的各个步骤。

FlowCallback() 方法用于处理流程中的各个步骤，当一个步骤完成后，它会决定是否继续执行下一个步骤。

### promotion 扶持

Promote 方法：进行筛选和排序。

DeDup 方法：用于去除重复的扶持项，主要基于 getter 获取的 SPU。

### session_delegate

一些基础接口和动态配置以及 entity_builder 调度。

### session_base

包括 session 管理、数据操作、日志记录、流处理等，为具体的业务逻辑提供了一个框架。

### pid_adjuster

pid 扶持策略。

### supporter

用于处理特定的支持（Promotion）操作。

用于处理支持逻辑的工具类，主要用于在排序或筛选过程中进行流量扶持操作。

## dispatcher

设置 listener，visitor，语法解析器以及语法管理器。

## scene

### search
