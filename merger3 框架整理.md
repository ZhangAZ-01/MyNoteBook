# merger3 框架整理

## common

公共定义

### tess2_utils

旧架构模块

## config

配置文件

区分本地和线上

- global_config 全局配置信息
- meta 所有数据配置结构
- holder 组织配置，解析

## entity

数据团队生成数据，生成到 Redis 或 RocksDB，Entity 用来存储数据。

将数据序列化后，将二进制数据填入到 entity 中。

### entity_builder

level 0: 用户侧数据

level 1: 商品侧数据

KVEntityBuilder：从 Redis 中获取。

ValuelessEntityBuilder：缺少 value 的 entityBuilder

QPEntityBuilder：解析复杂查询，通过查询结果构建

SimpleContextEntityBuilder：填充最初版本 context protobuf

NestedContextEntityBuilder：填充新版本 protobuf

ValuedEntityBuilder：构建 value 来自于内部计算或者模型计算的 Entity

ListEntityBuilder：用于构建 ListEntity 类型

### doc_builder

HubBytesDocBuilder：使用 DocsWriter，writer_ 对象通过 ToDocsByMove 方法转移到 google::protobuf::Message* doc

HubRawDocBuilder：不使用 DocsWriter 结构，直接向 entity 中添加

Tess2BytesDocBuilder

### helper

NameToKeyType：转换成 key

NameToValueType：多一步判断 name 是否为空，转换成 value

ToL0EntityOptions：用户侧 entity 选择

ToL1EntityOptions：商品侧 entity 选择

### list_entity_view

ReshapePadding：转换模型填充到原始 list_entity

IsInited：检查 list_entity 是否被初始化

CreativeSize：返回指定 list_entity 中 Creative 的数量

ToOutputRange：将输入范围转换为输出范围

BuildListEntity：构建一个 list_entity

DefaultPadding：默认的填充值

### qp_entity_handler

FillQuerySegList：填充查询段列表

FillQueryProcIntent：填充查询过程意图

FillQueryProcessResult：填充查询结果

FillRelevanceBertEmb：填充到模型

## featurehub

测试的 merger 下游数据

## flow

将所有技术串联，找到入口流程。

```C++
struct FlowData {
  SessionBase* session;
  grpc::Status status;
};
```

### args

#### CreativeFlowArgs::Parse

1. 从触发器中获取输入信息
2. 获取分发标签
3. 获取创意分发标签
4. 获取创意输入信息
5. 获取具体分发器位置
6. 启用创意覆盖

#### MixerFlowArgs::Parse

与 CreativeFlowsArgs 相似。

#### UnifiedFlowArgs::Parse

1. 初始化 Unified 配置
2. UnifiedFlowArgs::Parse：通过最终 syntax 获取 tess_version
3. PrepareEntityList：解析所需要的 entities
4. ProcessUnifiedEntitySpecially：特殊处理 user entity
5. PrepareRedisIdentifiers：准备需要的 redis identifiers

### init

- RunFlow(callback_t&& cb)
- GenerateFlow()
  - just() 用于创建一个包含初始数据的流程
    - FlowData{session(), grpc::Status()} 初始化
    - transfer(session()->main_context()) 转移上下文
    - Initialize() 初始化

- Initialize()
  - PrepareAppcfg() 检查并准备应用配置
  - OnRecvRequest() 接收并处理请求
  - CheckRequest()  检查请求有效性
  - Prepare()
    - creative_handler
    - mixer_handler
  - PrepareFlowTypes() 准备 Flow 类型
  - PrepareFlowArgs()  准备 Flow 类型参数
  - PrepareComposedAbtid()
  - PrepareMergerLog() 准备合并日志

#### redis

- PreCheck
- ProcessSlice 进程切片
  - MergerResult
- GenerateFlow 生成 Flow
- RunFlow(callback_t&& cb)
- GenerateFlow()
  - just()
    - FlowData{session(), grpc::Status()}
    - transfer(session()->main_context())
      - **Precheck()**
      - merger_error_sink(then(MakeSlices()))
      - merger_error_sink(ProcessSlices())
        - **Finally();**

#### optimus

- RunFlow(callback_t&& cb)
- GenerateFlow()
  - just()
    - FlowData{session(), grpc::Status()}
    - transfer(session()->main_context())
      - **Preprocess()**
      - merger_error_sink(then(MakeSlices()))
      - merger_error_sink(ProcessSlices())
        - **Finally();**

#### unified、mixer、creative、fepie_result、rank、prerank、dssm_vector

- RunFlow(callback_t&& cb)
- GenerateFlow()
  - just()
    - FlowData{session(), grpc::Status()}
    - transfer(session()->main_context())
      - **Precheck()**
      - merger_error_sink(then(MakeSlices()))
      - merger_error_sink(ProcessSlices())
        - **ProcessBusiness()**

#### Finally()

```C++
auto Finally() {
  return [](merger_result_t<FlowData> result) {
    if (auto p = std::get_if<merger_error_t>(&result); p) {
      LOG(INFO) << p->error_message();
      return FlowData{};  // redis 错误不下沉
    }

    return std::get<FlowData>(result);
  };
}
```

### call_*

使用 grpc

#### call_mixer

MixerCall：管理一个 gRPC 异步调用的完整生命周期，包括调用的发起、处理响应和错误。

MixerCallReceiver 和 MixerCallSender：这些类用于连接 gRPC 调用的生命周期管理到整个数据处理流。它们使用 oxygen::funflow 库的功能将异步调用集成到数据流中。

MixerCallFn：提供一个方便的接口来创建和配置 MixerCallSender 和 MixerCallReceiver。

## kafka

日志信息进行持久化

持久化操作：MergerLogMessage 中填充的日志数据会通过 Kafka 的生产者接口发送到指定的 Kafka 主题。

发送逻辑：KafkaMessage::Process() 方法中，构建 Kafka 消息并调用 Kafka 生产者的 ProduceMessage() 方法发送消息。如果发送失败，会释放已分配的资源。

## launcher 启动服务

### launcher.h

launcher
max_request
proxy_flag

### launcher 实现

## metric 监控数据信息

## redis 分布式存储

items_：类型：absl::flat_hash_map<std::string, Item> 存储 Redis 条目的映射，以 identifier 为键，Item 为值。

is_unfied：用于标识该条目是否是统一管理的 Redis 条目。

## resource 依赖的外部资源

adjustment_resource: 调整和校准资源

fepie_dynamic_loader: fepie 动态加载

fepie_resource: fepie 资源

full_memory: 将数据完全加载到内存

- FullMemoryLoader：将大量数据从持久存储加载到内存中。
- FullMemoryLoaders：管理多个 FullMemoryLoader 实例的单例类。

pid_resource: 管理和操作 goodsid 和 pid 的映射。

- LoadFromRealFile：从指定路径的文件加载 PID 资源，并解析 JSON 格式的数据，将 goodsid 和 pid 存储在 goodsid_to_pid_ 映射中。

resource: 资源加载、注册和管理。

- WaitingForLoadingResource：持续检查资源加载是否完成，没有完成每 10 秒检查一次。
- LoadResourceFromFile：从指定路径的配置文件加载资源，根据场景信息注册资源。
- RegisterResources：调用 LoadResourceFromFile 函数从文件加载资源，并进行注册。

- OnUnifiedRedisMetaUpdate：当 UnifiedResource 类型的资源发生变化时，函数会根据新的元数据更新 Redis 客户端的选项。
- RegisterGlobalResourceNotifier：为不同类型的资源注册了动态通知器，当这些资源发生变化时，会调用相应的回调函数进行处理和更新。

rocksdb_resource: 管理和操作 RocksDB 数据库中的资源。

- RocksDBResource：用于管理 RocksDB 数据库中的资源。
- FullMemoryResource：用于将 RocksDB 数据库中的资源加载到内存中。

unified_resource：管理统一资源的元数据。

## scene 可搭载场景

### ad 广告

### biz 业务

### front 前端

### otd 流程管理

### proxy 代理

### rec 推荐

### search 搜索

### supply 供应

### ug 用户内容

## scorer 打分系统

## session

## syntax 解析

用于解析和处理方法调用语法的系统。

## tools
