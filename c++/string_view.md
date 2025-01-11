* std::string按值传递字符串会拷贝一份，而std::string_view不拷贝字符串，只是一个字符串的查看器，只要用于创建的std::string或char*在有效范围内std::string_view就是有效的,所以std::string_view通常作为不可变的字符串参数使用,std::string_view可以作为返回值，但要小心使用。
```
#include <iostream>
#include <string>
#include <string_view>
void print_s(std::string_view s) {//如果s是字符串那么会发送拷贝
    std::cout << s << std::endl;
}
int main()
{
    std::string value{ "abcdef" };
    print_s(value);
    std::string_view sv{};
    {
        std::string s{ "Hello, world!" };
        sv = s; // sv is now viewing s
    } //s变量销毁了，访问sv是未定义行为

    std::cout << sv << '\n'; //访问sv是未定义行为

    using namespace std::string_literals;
    std::string_view name{ "Alex"s }; // "Alex"s创建了一个临时的std::string
    std::cout << name << '\n'; // n访问name是未定义行为

    std::string s2{ "Hello, world!" };
    std::string_view sv2{ s2 };
    s2 = "Hello, a!";    // 修改s2会使sv2失效
    std::cout << sv2 << '\n';   // 未定义行为

    std::string_view value_view(value);
    value_view.remove_prefix(1);
    std::cout << value_view << '\n';//输出bcdef

    value_view.remove_suffix(2);
    std::cout << value_view << '\n';//输出bcd //注意value_view这个时候不是以null结尾的字符串


    std::cout << value << '\n';//依然是abcdef

    return 0;
}
```
* 推荐使用std::string_view作为参数而不是const std::string_view
如下比较，当const std::string&接收一个c风格的字符串时参数会创建一个临时的std::string对象，性能不好。
```
#include <iostream>
#include <string>
#include <string_view>

void printSV(std::string_view sv)
{
    std::cout << sv << '\n';
}

void printS(const std::string& s)
{
    std::cout << s << '\n';
}

int main()
{
    std::string s{ "Hello, world" };
    std::string_view sv { s };

    printSV(s);              // ok: inexpensive conversion from std::string to std::string_view
    printSV(sv);             // ok: inexpensive copy of std::string_view
    printSV("Hello, world"); // ok: inexpensive conversion of C-style string literal to std::string_view

    printS(s);              // ok: inexpensive bind to std::string argument
    //printS(sv);             // compile error: cannot implicit convert std::string_view to std::string
    printS(static_cast<std::string>(sv)); // bad: expensive creation of std::string temporary
    printS("Hello, world"); // 这里会创建一份临时的std::string对象，性能不好

    return 0;
}
```
* std::string 的拷贝构造和拷贝赋值是申请了一片新的内存并把数据拷贝到这片内存，并不是写时复制。
* 参考[string_view介绍](https://www.learncpp.com/cpp-tutorial/introduction-to-stdstring_view)

