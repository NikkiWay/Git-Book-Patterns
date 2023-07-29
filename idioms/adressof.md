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

# Addressof

## Назначение

Идиома Addressof используется для получения адреса объекта. Она позволяет получить фактический адрес, где данные хранятся в памяти компьютера. Затем этот адрес может быть использован для различных целей, например, для передачи в функцию, сохранения в указатель или создания ссылки.

## Решаемые задачи

* Нахождение реального адреса объекта в памяти

C++ позволяет перегружать унарный оператор амперсанда (&) для типов классов. Тип возврата такого оператора не обязательно должен быть фактическим адресом объекта.  Идиома addressof позволяет найти реальный адрес объекта независимо от перегруженного унарного оператора амперсанда и его защиты доступа.

## Общая реализация на языке C++

{% tabs %}
{% tab title="Addressof" %}
{% hint style="info" %}
Внутри функции addressof() используется оператор reinterpret\_cast, который выполняет преобразование указателей между различными типами. В данном случае, функция addressof использует reinterpret\_cast для преобразования ссылки на объект типа T в указатель на этот объект. Преобразование начинается с преобразования ссылки на объект типа T в ссылку на объект типа const volatile char, чтобы обеспечить максимальную возможную совместимость между типами. Затем используется оператор const\_cast для удаления квалификатора const с ссылки на объект типа const volatile char, чтобы получить ссылку на изменяемый объект типа volatile char.
{% endhint %}

{% code fullWidth="true" %}
```cpp
template<class T>
T *addressof(T &v)
{
    return reinterpret_cast<T *> (&const_cast<char &> (reinterpret_cast <const volatile char &> (v)));
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
int main()
{
    nonaddressable na;
    nonaddressable *naptr = addressof(na);
    
    return 0;
}
```
{% endcode %}
