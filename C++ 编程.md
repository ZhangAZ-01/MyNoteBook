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

## 使用推荐

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

## std::string_view::npos

non-position
