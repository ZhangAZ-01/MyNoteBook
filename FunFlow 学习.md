# FunFlow 学习

## CPO

customization point object

```C++
// 将变参模板转为类型列表
template <typename... Input>
struct ReceiverBase {
  using input_t = traits::TypeList<Input...>;
};

// 处理无输入情况
template <>
struct ReceiverBase<void> {
  using input_t = traits::TypeList<>;
};

template <typename R>
using receiver_input_t = typename R::input_t;


// 类型索引
template <size_t Index, typename R>
using receiver_input_element_t = type_element_t<Index, receiver_input_t<R>>;

namespace receiver_cpo {
struct SetValueFn {
  template <typename R, typename... Values>
  auto operator()(R&& receiver, Values&&... values) const {
    return tag_invoke(SetValueFn{}, std::move(receiver), std::forward<Values>(values)...);
  }
};
}  // namespace receiver_cpo
```

`input_t` 类型列表
