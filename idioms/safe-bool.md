---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Safe bool

## Назначение

Идиома Safe bool (безопасное преобразование в булево значение) - это подход в программировании, который обеспечивает безопасное преобразование объектов в булево значение. Назначение идиомы состоит в том, чтобы предоставить объектам способность быть использованными в условных операторах при проверке на истинность без явного преобразования в булево значение. Это особенно полезно в контексте условных операторов, где требуется использование объектов, которые не являются простыми булевыми значениями.&#x20;

## Решаемые задачи

* Проверка корректности объектов с помощью условных операторов

В языке программирования C++ условные операторы, такие как if или while, ожидают, что условие будет иметь булево значение (true или false). Однако в некоторых случаях объекты не могут быть просто преобразованы в булево значение, так как они представляют более сложные состояния или требуют дополнительной логики для определения истинности или ложности. Идиома safe bool помогает решить эту задачу. Вместо использования явного преобразования в булево значение, можно просто проверить объект на истинность с помощью условных операторов.

## Общая реализация на языке C++

{% hint style="info" %}
Идиома может быть реализована несколькими способами:

1. Перегрузка оператора приведения к типу operator bool().
2. Перегрузка оператора приведения к типу operator void \*().
3. Перегрузка оператора приведения к логическому типу (basic\_ios).
{% endhint %}

{% tabs %}
{% tab title="Перегрузка оператора bool" %}
{% hint style="info" %}
В этом примере перегружен оператор приведения к типу bool, который возвращает true, если значение объекта MyClass не равно нулю, и false в противном случае. Это позволяет использовать объекты MyClass в условиях ветвлений if и while.
{% endhint %}

{% code fullWidth="true" %}
```cpp
class MyClass 
{
public:
    MyClass(int value) : value_(value) { }

    operator bool() const 
    { 
        return value_ != 0; 
    }

private:
    int value_;
};
```
{% endcode %}

{% code fullWidth="true" %}
```cpp
#include <iostream>

int main() 
{
    MyClass obj1(10);
    MyClass obj2(0);

    if (obj1) 
    {
        std:: cout << "obj1 is true" << std:: endl;
    }
    
    if (!obj2)
    {
        std:: cout << "obj2 is false" << std:: endl;
    }

    return 0;
}
```
{% endcode %}
{% endtab %}

{% tab title="Перегрузка оператора void*" %}
{% hint style="info" %}
В этом примере перегружен оператор приведения к типу void\*, который возвращает указатель на объект MyClass, если значение объекта не равно нулю, и nullptr в противном случае. Это позволяет использовать объекты MyClass в условиях ветвлений if и while, а также при работе с указателями.
{% endhint %}

{% code fullWidth="true" %}
```cpp
class MyClass {
public:
    MyClass(int value) : value_(value) { }

    operator void* () const 
    {
        return value_ != 0 ? this : nullptr;
    }
    
private:
    int value_;
};
```
{% endcode %}

<pre class="language-cpp" data-full-width="true"><code class="lang-cpp">#include &#x3C;iostream>

<strong>int main() {
</strong>    MyClass obj1(10);
    MyClass obj2(0);

    if (obj1) 
    {
        std:: cout &#x3C;&#x3C; "obj1 is true" &#x3C;&#x3C; std:: endl;
    }
    
    if (!obj2) 
    {
        std:: cout &#x3C;&#x3C; "obj2 is false" &#x3C;&#x3C; std:: endl;
    }

    return 0;
}
</code></pre>
{% endtab %}

{% tab title="Перегрузка оператора basic_ios" %}
{% hint style="info" %}
В этом примере перегружен оператор приведения к логическому типу (basic\_ios[^1]). Служебное слово explicit означает, что эта функция сработает только при явном вызове преобразования, например:     static\_cast\<bool>(поток). Но в стандарте C++11 отмечено, что данная функция будет срабатывать и в условиях ветвлений if, циклов while и других подобных случаях
{% endhint %}

{% code fullWidth="true" %}
```cpp
struct Testable
{
    explicit operator bool() const 
    { 
        // Обработка
        return false; 
    }
};
```
{% endcode %}

{% code fullWidth="true" %}
```cpp
int main()
{
    Testable a, b;
    if (a) 
    { 
        // Процесс
    }
    
    return 0;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

[^1]: `basic_ios` является базовым классом для ввода-вывода в стандартной библиотеке C++. Он предоставляет базовую функциональность для работы с потоками ввода-вывода, такими как `std::cin`, `std::cout`, `std::ifstream`, `std::ofstream` и другими.
