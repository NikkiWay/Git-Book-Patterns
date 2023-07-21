---
description: Factory method
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

# Возможные реализации для решения конкретных задач

## Новый объект

В рамках данной реализации появляется возможность избавиться от необходимости создания конкретного объекта в коде.

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
    virtual ~Product() = 0;
    virtual void run() = 0;
};

Product::~Product() {}
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd1 : public Product
{
public:
    virtual ~ConcreteProd1() override 
    { 
        cout << "Destructor;" << endl; 
    }
    virtual void run() override 
    { 
        cout << "Method run;" << endl; 
    }
};

```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
```cpp
class Creator
{
public:
    virtual unique_ptr<Product> createProduct() = 0;
};
```
{% endtab %}

{% tab title="ConcreteCreator" %}
{% code fullWidth="true" %}
```cpp
template <typename Tprod>
class ConcreteCreator : public Creator
{
public:
    virtual unique_ptr<Product> createProduct() override
    {
        return unique_ptr<Product>(new Tprod());
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>

using namespace std;

int main()
{
    shared_ptr<Creator> cr(new ConCreator<ConProd1>()); 
    shared_ptr<Product> ptr = cr->createProduct();

    ptr->run();
}
```
{% endcode %}

## Шаблонный базовый класс
