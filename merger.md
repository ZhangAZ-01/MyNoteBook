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

meta.h：存储配置信息变量。

## entity

### builder

helper：选择哪个级别的 entity。

builder：定义构建 entity 的基本配置，可以根据具体的业务场景实现不同的 EntityBuilder 和 DocBuilder。

list_entity：解析给定的 JSON 数据并填充 QueryProcessInfo 对象。

## launcher

初始化以及启动相关服务。
