* enable_if实现
```
template<bool B, class T = void>
struct enalbe_if {};

template<class T>
struct enable_if<true,T> { using type = T;};

```
* 类型检查
```
template <typename... T>
struct are_all_integral : 
public std::conjunction<std::is_integral<T>...>{}

template <typename... T>
void check<T... vals>{
    static_assert(are_all_integral<T...>::value,"All vals must be integral");
}

```
* std::void_t用于生成void类型
```
template<class, class = void>
struct has_type_member : std::false_type {};

template<class T>
struct has_type_member<T, std::void_t<typename T::type>> : std::true_type {};
# 当特化模板的参数与实例化参数完全匹配时，优先选择特化版本所以可以用来判断T是否有type成员
```
* Concepts
```
template <typename BI, typename EI>
concept Neqable = requires(BI bi,EI ei){
    { bi != ei} - > std::convertible_to<bool>;
    //表示bi!=ei的返回值可以转化为bool值
    //这里不是要求bi和ei必须不相等
}
template<typename BI,Nequable<BI> EI>
constexpr bool foo(BI bi, EI ei){
    return true;
}
template<typename BI, typename EI>
constexpr bool foo_3(BI bi, Ei ei) requires Neqable<BI,EI>{
    return true;
}

constexpr bool foo_4(auto bi, Neqable<decltype(bi)> auto ei){
    return true;
}

```
* 可以定义有多个表达式,多个requires字句可以做逻辑与，或,取反操作
```
template <typename T>
concept MyConcept = requires(T a, T b) {
    // 检查 T 类型是否支持加法操作
    a + b;

    // 检查 T 类型是否有 size() 成员函数，且返回值可以转换为 size_t
    { a.size() } -> std::convertible_to<size_t>;

    // 检查 T 类型是否支持相等比较，且结果可以转换为 bool
    { a == b } -> std::convertible_to<bool>;

    // 检查 T 类型是否有嵌套类型 value_type
    typename T::value_type;

    // 检查 T 类型是否有 begin() 和 end() 成员函数，且返回值是迭代器
    { a.begin() } -> std::input_iterator; //input-iterator也是个concept
    { a.end() } -> std::input_iterator;
};
```