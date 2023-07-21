---
description: Object pool
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Пулл объектов

## Назначение

Пулл объектов (Object pool) – это порождающий шаблон проектирования, который предоставляет набор инициализированных и готовых для использования объектов. Используется для повышения производительности и управления ресурсами путем повторного использования предварительно созданных объектов вместо создания новых. Это полезно в случаях, когда создание объектов требует значительных ресурсов, таких как соединения с базой данных.

## Решаемые задачи

* Организация и контроль доступа.

Паттерн обеспечивает централизованное управление доступом к объектам пула, что позволяет контролировать и ограничивать количество одновременно используемых объектов.

* Повышение производительности.

Повторное использование объектов позволяет избежать увеличение затрачиваемых ресуров на создание и уничтожение объектов.

* Управление жизненным циклом объектов.

Паттерн упрощает управление жизненным циклом объектов, так как клиенту не нужно явно создавать и уничтожать объекты, а просто получать и возвращать их в пулл.

## Общая реализация на языке C++

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
private:
    static size_t count;
    
public:
    Product() 
    { 
        cout << "Constructor(" << ++count << ");" << endl; 
    }
    ~Product() 
    { 
        cout << "Destructor(" << count-- << ");" << endl; 
    }

    void clear() 
    { 
        cout << "Method clear: 0x" << this << endl; 
    }
};

size_t Product::count = 0;
```
{% endcode %}
{% endtab %}

{% tab title="ObjectPool" %}
{% code fullWidth="true" %}
```cpp
template <typename Type>
class ObjectPool
{
public:
    static shared_ptr<ObjectPool<Type>> instance();
    
    shared_ptr<Type> getObject(); 
    
    bool releaseObject(shared_ptr<Type>& obj);
    
    size_t count() const 
    { 
        return pool.size(); 
    }

    iterator<output_iterator_tag, const pair<bool, shared_ptr<Type>>> begin() const;
    iterator<output_iterator_tag, const pair<bool, shared_ptr<Type>>> end() const;

    ObjectPool(const ObjectPool<Type>&) = delete;    
    ObjectPool<Type>& operator=(const ObjectPool<Type>&) = delete;

private:
    vector<pair<bool, shared_ptr<Type>>> pool;

    ObjectPool() {}

    pair<bool, shared_ptr<Type>> create();

    template <typename Type>
    friend ostream& operator << (ostream& os, const ObjectPool<Type>& pl);
};

template <typename Type>
ostream& operator << (ostream& os, const ObjectPool<Type>& pl)
{
    for (auto elem : pl.pool)
        os << "{" << elem.first << ", 0x" << elem.second << "} ";
        
    return os;
}
```
{% endcode %}
{% endtab %}

{% tab title="Methods" %}
```cpp
#pragma region Methods  

template <typename Type>
shared_ptr<ObjectPool<Type>> ObjectPool<Type>::instance()
{
    static shared_ptr<ObjectPool<Type>> myInstance(new ObjectPool<Type>());

    return myInstance;
}

template <typename Type>
shared_ptr<Type> ObjectPool<Type>::getObject()
{
    size_t i;
    for (i = 0; i < pool.size() && pool[i].first; ++i);

    if (i < pool.size())
    {
        pool[i].first = true;
    }
    else
    {
        pool.push_back(create());
    }

    return pool[i].second;
}

template <typename Type>
bool ObjectPool<Type>::releaseObject(shared_ptr<Type>& obj)
{
    size_t i;
    for (i = 0; i < pool.size() && pool[i].second != obj; ++i);

    if (i == pool.size()) return false;

    obj.reset();
    pool[i].first = false;
    pool[i].second->clear();

    return true;
}

template <typename Type>
pair<bool, shared_ptr<Type>> ObjectPool<Type>::create()
{
    return pair<bool, shared_ptr<Type>>(true, shared_ptr<Type>(new Type()));
}

#pragma endregion
```
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <iterator>
# include <vector>

using namespace std;

int main()
{
    shared_ptr<ObjectPool<Product>> pool = ObjectPool<Product>::instance();

    vector<shared_ptr<Product>> vec(4);

    for (auto& elem : vec)
        elem = pool->getObject();

    pool->releaseObject(vec[1]);

    cout << *pool << endl;

    shared_ptr<Product> ptr = pool->getObject();
    vec[1] = pool->getObject();

    cout << *pool << endl;
}
```
{% endcode %}

## Преимущества

1. Улучшается производительность за счет минимизации создания и уничтожения множества объектов.
2. Возможность ограничивать и контролировать число используемых объектов.
3. Возможность несколько раз использовать один и тот же объект.

## Недостатки

1. Возмоность утечки информации. Если объект не очищается или его состояние не сбрасывается перед возвращением в пулл, может возникнуть утечка информации. Например, если объект содержит конфиденциальные данные или ссылки на другие объекты, эта информация может остаться в объекте после его возврата в пулл.
2. Увеличениюе объема кода и усложнение архитектуры приложения. Внедрение паттерна требует создания дополнительной логики для управления пуллом объектов, обработки доступа к объектам, контроля их состояния и т.д.
