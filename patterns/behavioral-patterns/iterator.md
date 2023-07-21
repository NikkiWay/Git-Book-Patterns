---
description: Iterator
---

# Итератор

## Назначение

Поведенческий паттерн итератор (Iterator) предоставляет способ последовательного доступа ко всем элементам составного объекта, не раскрывая его внутреннего представления.

## Решаемые задачи

* сохранение инкапсуляции объектов при переборе в структуре данных
* поддержка нескольких активных обходов одного и того же агрегированного (составленного из подобъектов) объекта
* предоставление единообразного интерфейса с целью обхода различных агрегированных структур (поддержка полиморфной итерации)

## Общая реализация на С++

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
#include <iostream>
#include <string>
#include <vector>
```
{% endcode %}
{% endtab %}

{% tab title="Iterator" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <typename T, typename U>
class Iterator 
{
public:
    typedef typename std::vector<T>::iterator iter_type;
    Iterator(U *p_data, bool reverse = false) : m_p_data_(p_data) 
    {
        m_it_ = m_p_data_->m_data_.begin();
    }
    void First() 
    {
        m_it_ = m_p_data_->m_data_.begin();
    }
    void Next() 
    {
        m_it_++;
    }
    bool IsDone() 
    {
        return (m_it_ == m_p_data_->m_data_.end());
    }
    iter_type Current() 
    {
        return m_it_;
    }
private:
    U *m_p_data_;
    iter_type m_it_;
};
```
{% endcode %}
{% endtab %}

{% tab title="Container" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <class T>
class Container 
{
    friend class Iterator<T, Container>;
public:
    void Add(T a) 
    {
        m_data_.push_back(a);
    }
    Iterator<T, Container> *CreateIterator() 
    {
        return new Iterator<T, Container>(this);
    }
private:
    std::vector<T> m_data_;
};
```
{% endcode %}
{% endtab %}

{% tab title="Data" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Data 
{
public:
    Data(int a = 0) : m_data_(a) {}
    void set_data(int a) 
    {
        m_data_ = a;
    }
    int data() 
    {
        return m_data_;
    }
private:
    int m_data_;
};
```
{% endcode %}
{% endtab %}

{% tab title="ClientCode" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
void ClientCode() 
{
    std::cout << "Iterator with int" << std::endl;
    Container<int> cont;
    for (int i = 0; i < 10; i++) 
        cont.Add(i);
    Iterator<int, Container<int>> *it = cont.CreateIterator();
    for (it->First(); !it->IsDone(); it->Next()) 
        std::cout << *it->Current() << " ";
    std::cout << std::endl;
    Container<Data> cont2;
    Data a(100), b(1000), c(10000);
    cont2.Add(a);
    cont2.Add(b);
    cont2.Add(c);
    std::cout << std::endl << "Iterator with custom Class" << std::endl;
    Iterator<Data, Container<Data>> *it2 = cont2.CreateIterator();
    for (it2->First(); !it2->IsDone(); it2->Next()) 
        std::cout << it2->Current()->data() << " ";
    std::cout << std::endl;
    delete it;
    delete it2;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main() 
{
    ClientCode();
    return 0;
}
```
{% endcode %}

## Преимущества

* упрощение работы со структурами данных: обход элементов без знания об особенностях внутренней реализации структуры данных
* возможность работать с различными типами структур данных независимо от их реализации
* возможность реализовывать различные алгоритмы обработки структур данных

## Недостатки

* добавление новых типов структур данных может потребовать изменения кода итератора

## Связь с другими паттернами

* [Компоновщик](../structural-patterns/composite.md): итераторы довольно часто применяются для обхода рекурсивных структур, создаваемых компоновщиком.
* [Фабричный метод](../creationals-patterns/factory-method.md): полиморфные итераторы поручают фабричным методам инстанцировать (создание экземпляров) подходящие подклассы класса Iterator.
* [Хранитель](memento.md): итератор может использовать хранитель для сохранения состояния итерации и при этом содержит его внутри себя.
