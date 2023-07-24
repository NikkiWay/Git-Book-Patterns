# Контейнеры с type erasure

#### <mark style="color:orange;">std::any</mark> - контейнер, который может хранить объекты разных типов данных

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main() 
{
    std::any a = 42;
    std::cout << std::any_cast<int>(a) << std::endl;
    a = 3.14;
    std::cout << std::any_cast<double>(a) << std::endl;
    a = std::string("Hello, World!");
    std::cout << std::any_cast<std::string>(a) << std::endl;
    return 0;
}
```
{% endcode %}

#### <mark style="color:orange;">std::variant</mark> - может хранить объекты только заданных типов данных

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main() 
{
    std::variant<int, double, std::string> v;
    v = 42;
    std::cout << std::get<int>(v) << std::endl;
    v = 3.14;
    std::cout << std::get<double>(v) << std::endl;
    v = std::string("Hello, World!");
    std::cout << std::get<std::string>(v) << std::endl;
    return 0;
}
```
{% endcode %}

#### <mark style="color:orange;">std::function</mark> - объект функции, который может хранить функции разных типов данных

{% code lineNumbers="true" fullWidth="true" %}
```cpp
void call_function(const std::function<void ()>& func) 
{
    std::cout << "Calling function... ";
    func();
}
void print_hello() 
{
    std::cout << "Hello, ";
}
void print_world() 
{
    std::cout << "world!" << std::endl;
}
int main() 
{
    std::function<void ()> f1 = print_hello;
    std::function<void ()> f2 = print_world;
    call_function(f1);
    call_function(f2);
    return 0;
}
```
{% endcode %}
