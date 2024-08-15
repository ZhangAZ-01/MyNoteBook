# C++ 语法

## `[[nodiscard]]`

## `auto`

### 总结

`auto` 是一个不能一概而论的语言特性：

- 优美的 `auto` 可以大大增加可读性，提升开发效率；
- 拙劣的 `auto` 也会降低可读性，降低代码质量，甚至降低程序性能。

### 最佳实践

- 必须使用 `auto` 的场景：
  - 迭代器对象必须使用 `auto`；
  - Range-based for loop map-like 对象必须使用 `auto&` 或者 `const auto&`；
  - Lambda 表达式必须使用 `auto`；

- 必须不使用 `auto` 的场景：
  - 展望类型 (不能指针转类型) 和 `std::string_view` 必须使用类型名；

- 其它场景尽量避免类型名，如果使用 `auto`，必须要求一致性、可读性、合理性上的收益；
  - 如果对指针类型使用 `auto`，必须使用 `auto*` 或者 `const auto*`；
  - 如果对指针型和引用型使用 `auto`，必须使用 `auto&` 或者 `const auto&`；
  - 这有助于的选项！如果要求公牍，首先必须考虑是不是合理。

## `std::function`

- 可以封装函数、函数对象、lambda 表达式、bind 表达式、函数指针、成员函数指针等。
- 实现函数回调。
- 回调函数大多使用在异步编程。

```C++
class FeedsPairDedup {
 private:
  using callback_t = std::function<void(pb_feeds_item_t*, pb_feeds_item_t*)>;
}

void FeedsPairDedup::Dedup(bool prepare_enabled, const std::vector<size_t>& ref_indexes, const callback_t& callback,pb_feeds_item_t* items, pb_feeds_item_t* output_items) {
  if (prepare_enabled) {
    // ...
    callback(&from_items, output_items);
  } else {
    callback(items, output_items);
  }
}

void FeedsPairDedup::Dedup_V1(bool prepare_enabled, const std::vector<size_t>& ref_indexes, pb_feeds_item_t* items,pb_feeds_item_t* output_items) {
  Dedup(
      prepare_enabled, ref_indexes,
      [&](pb_feeds_item_t* from_items, pb_feeds_item_t* to_items) {
        this->DeleteDuplicateItems_V1(from_items, to_items);
      },
      items, output_items);
}

void FeedsPairDedup::Dedup_V2(bool prepare_enabled, const std::vector<size_t>& ref_indexes, pb_feeds_item_t* items,pb_feeds_item_t* output_items) {
  Dedup(
      prepare_enabled, ref_indexes,
      [&](pb_feeds_item_t* from_items, pb_feeds_item_t* to_items) {
        this->DeleteDuplicateItems_V2(from_items, to_items);
      },
      items, output_items);
}
```

## 使用推荐

## 指针 & 引用

入参用引用，出参用指针。
传入参数：只读不修改，传 `const&`。
传出参数：先读后写，传 `*`。

### 代码块注释

可以使用如下形式注释代码块，清晰快捷。

```C++
#if 0
// ...
#endif

#if 1
// ...
#endif
```

### std::vector::reserve

- 预先分配空间，减少内存分配数和对象拷贝/移动数。
  - 分配大小不确定，根据经验进行 reserve。
  - 分配大小确定，使用 `reserve` + `emplace_back` 或 `resize` + `[]`。
  - 复用 `vector` 对象。

### std::shared_ptr

- 不用裸指针上持有堆上对象，使用智能指针。
- 优先使用 `unique_ptr`。
  - 当且仅当**共享所有权**时使用 `shared_ptr`，并且优先使用 `std::make_shared` 以及避免拷贝（参数，返回值，放到容器中）。

## 单元测试

- 测试代码和被测试代码在一个命名空间。
- 通过断言进行测试。
- 使用 gtest。

## 失败测试

测试应该根据所掌握（完全包围）的条件失败并且测试应该不成功。

## std::string_view::npos

non-position

## std::string & std::string_view

- `std::string`: 进行动态内存分配。
- `std::string_view`: 不进行动态内存分配，只读。构造和复制几乎是即时的，因为它只是复制指针和长度。
  - `std::string_view` 里面的 `data()` 方法不会在遇到 `\0` 时截断数据，只会返回开头指针。
  - `carbon::safe_strtof` 中不提供 `std::string` 的版本，因为 `std::string_view` 不能截取 `\0`。

## lowest() && min()

- `std::numeric_limits<float>::lowest()`: 返回 float 类型的最小负值，即数值上最接近负无穷大的值。
- `std::numeric_limits<float>::min()`: 返回 float 类型的最小正值，即最接近零的正数值，但不包括零。

## 可读性

- 命名注意单复数，命名的 vector 或其他类型，使用复数。
- 使用 using 命名较长的类型，增加可读性。

```C++
using pb_feeds_items_t = google::protobuf::RepeatedPtrField<FeedsItem>;
```
