---
description: Factory method
---

# Реализации на С++

## Общая реализация на языке С++

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

{% tab title="Creator" %}
{% code fullWidth="true" %}
```cpp
class Creator
{
public:
      virtual ~Creator() = 0;
      
      virtual unique_ptr<Product> createProduct() = 0;
};

Creator::~Creator() = default;
```
{% endcode %}
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

{% tab title="ConcreteProduct1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProduct1 : public Product
{
public:
      virtual ~ConcreteProduct1() override 
      {
            cout << "Destructor;" << endl; 
      }
      
      virtual void run() override 
      { 
            cout << "Method run;" << endl;
      }
      
      unique_ptr<Creator> createConcreteCreator()
      {
            return unique_ptr<Creator>(new ConcreteCreator<ConcreteProduct1>());
      }
};
```
{% endcode %}
{% endtab %}

{% tab title="Solution" %}
{% hint style="info" %}
Класс Solution предоставляет метод для регистрации в данном случае Creator'ов для карты (map), состоящий из пар (pair): ключ + значение.
{% endhint %}

{% code fullWidth="true" %}
```cpp
class Solution
{
public:
      typedef unique_ptr<Creator> (*CreateCreator)();
 
      bool registration(size_t id, CreateCreator createfun)
      {
            return callbacks.insert(CallBackMap::value_type(id, createfun)).second;
      }
      
      bool check(size_t id) 
      { 
            return callbacks.erase(id) == 1; 
      }

      unique_ptr<Creator> create(size_t id)
      {
            CallBackMap::const_iterator it = callbacks.find(id);
 
            if (it == callbacks.end())
            {
//                throw IdError();
            }
            return unique_ptr<Creator>((it->second)());
      }
private:
      using CallBackMap = map<size_t, CreateCreator>;
      
      CallBackMap callbacks;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream> 
# include <memory> 
# include <map> 

using namespace std; 

int main()
{
      Solution solution;
      solution.registration(1, createConcreteCreator);
      shared_ptr<Creator> cr(solution.create(1));
      shared_ptr<Product> ptr = cr->createProduct();
      ptr->run();
}
```
{% endcode %}
