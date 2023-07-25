---
description: Curiously recurring template pattern
---

# CRTP

Идиома CRTP предоставляет статическое полиморфное поведение без использования виртуальных функций, что позволяет увеличить производительность за счет отказа от динамического связывания.

{% hint style="info" %}
Статический полиморфизм означает, что выбор, какая функция будет вызвана, определяется на этапе компиляции, а не во время выполнения.
{% endhint %}

Идиома CRTP разделяет функциональность, зависящую от типа и связывает функциональность, независимую от типа, с базовым классом, используя шаблон с "саморекурсией".&#x20;

## Решаемые задачи

* достижение статического полиморфизма
* избежание дублирования кода
* получение возможности повторного использования кода

## Пример реализации идиомы CRTP&#x20;

{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <class derived>
struct compare {};

struct value : compare<value> 
{
    int m_x;
    value(int x) : m_x(x) {}
    bool operator<(const value &rhs) const 
    { 
        return m_x < rhs.m_x; 
    }
};

template <class derived>
bool operator > (const compare<derived> &lhs, const compare<derived> &rhs) 
{
    // static_assert(std::is_base_of_v<compare<derived>, derived>); // Безопасность времени компиляции
    return (static_cast<const derived&>(rhs) < static_cast<const derived&>(lhs));
}

int main() 
{
    value v1{5}, v2{10};
    cout << boolalpha << "v1 > v2: " << (v1 > v2) << '\n';
    return 0;
}
```
{% endcode %}
