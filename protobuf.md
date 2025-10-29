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

## 常用成员函数

- 所有的 protobuf 都继承自 google::protobuf::Message。

- Message 提供的成员函数

1. `SerializeToString()` / `SerializeToArray()`
`SerializeToString(string*output)`：将消息序列化为字符串，并存储在指定的 output 字符串中。
`SerializeToArray(void* data, int size)`：将消息序列化为字节数组，并存储在指定的数据区域中。

2. `ParseFromString(const string& data)` / `ParseFromArray(const void* data, int size)`：解析消息的内容。
3. `ByteSizeLong()`：返回消息序列化后的字节大小，以 int64_t 类型表示。
4. `SerializeToOstream()` / `ParseFromIstream()`
`SerializeToOstream(ostream*output_stream)`：将消息序列化到给定的输出流中。
`ParseFromIstream(istream* input_stream)`：从输入流中解析消息的内容。

5. `CopyFrom(const Message& from)`：从另一个消息对象复制内容到当前消息对象。
6. `MergeFrom(const Message& from)`：合并另一个消息对象的内容到当前消息对象，用于合并部分更新的消息。
7. `Clear()`：清除消息的所有字段值，将消息重置为空状态。
8. `GetDescriptor()`：获取消息类型的描述符，用于动态获取消息结构信息。
9. `MessageToJsonString()` ：将消息序列化为 JSON 字符串。可以使用 `ok()` 检查是否成功。
10. `DeleteSubrange(int start, int count)`：删除指定范围内的元素。

## 字段废弃

### `[deprecated=true]`

```plain
message MyMessage {
    string deprecated_my_field = 1 [deprecated=true];
}
```

### `reserved`

```plain
message MyMessage {
    reserved 1; // Deprecated "my_field"
}
```

### 总结

两种方法都可以；第一种选择将定义保留在模型中，因此**仍然可以访问和查询**，但它可能会在可用时生成构建警告（对于从架构更新的客户端）。在某些情况下，这可能很有用。

第二种选择是将字段从模型中完全移除，因此当客户端从架构更新时，任何现有用法都会在构建时中断。

## `Arena`

实现 Protobuf 的内存管理，使用 Arena 分配时，新对象从一块称为 Arena 的大型预分配内存中分配。通过丢弃整个 Arena，可以一次性释放所有对象。

## `CHECK`

用于在运行时检查条件是否为真。如果条件为假，程序将终止并输出错误信息。
