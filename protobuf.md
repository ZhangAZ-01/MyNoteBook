# protobuf3

## 定义一个消息类型

```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

- `singular`：一个格式良好的消息应该有 0 个或者 1 个这种字段（但是不能超过 1 个）。
- `repeated`：在一个格式良好的消息中，这种字段可以重复任意多次（包括 0 次）。重复的值的顺序会被保留。
- `reserved`：保留表示符。

## 导入定义

```proto
import "myproject/other_protos.proto";
```

## 更新消息类型

- 不要更改任何已有的字段的数值标识。
- 如果你增加新的字段，使用旧格式的字段仍然可以被你新产生的代码所解析。

## Any

- Any 类型消息允许你在没有指定他们的.proto 定义的情况下使用消息作为一个嵌套类型。
- 一个 Any 类型包括一个可以被序列化 bytes 类型的任意消息，以及一个 URL 作为一个全局标识符和解析消息类型。
- 为了使用 Any 类型，你需要导入 import google/protobuf/any.proto

## Maps

```proto
map<key_type, value_type> map_field = N;
```

- Map 的字段可以是 repeated。

## Packages

- 可选的 package 声明符，用来防止不同的消息类型有命名冲突。
