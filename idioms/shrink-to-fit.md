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

# Shrink to fit

## Назначение

Идиома Shrink to fit решает проблему избыточного использования памяти при работе с контейнерами.&#x20;

Реализация заключается в создании или вызове метода shrink\_to\_fit(), который принудительно уменьшает выделенную память до минимально возможного размера, необходимого для хранения содержимого контейнерами.

## Решаемые задачи

* экономия ресурсов памяти
* уменьшение нагрузки на систему

## Примеры реализации идиомы Shrink to fit

#### Первый способ&#x20;

{% code lineNumbers="true" fullWidth="true" %}
```cpp
std::vector<int> v;
// v заменяется его временной копией, которая является оптимальной по емкости
std::vector<int>(v.begin(), v.end()).swap(v);
```
{% endcode %}

#### Второй способ

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main() 
{
    std::vector<int> vec = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

    for (auto it = vec.begin(); it != vec.end(); ) 
    {
        it = vec.erase(it);
        if (it != vec.end()) 
            ++it;
    }
    vec.shrink_to_fit(); // Уменьшаем выделенную память до минимально возможного размера    
    std::cout << "Size: " << vec.size() << std::endl;
    std::cout << "Capacity: " << vec.capacity() << std::endl;
    return 0;
}
```
{% endcode %}

{% hint style="info" %}
В C++11 некоторые контейнеры (такие как vector, deque, basic\_string) объявляют идиому "Shrink to fit", как функцию shrink\_to\_fit().

shrink\_to\_fit() - это необязательный запрос на уменьшение capacity() до size()
{% endhint %}
