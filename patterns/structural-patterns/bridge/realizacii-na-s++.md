---
description: Bridge
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="includes" %}
{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="Implementor" %}
{% code fullWidth="true" %}
```cpp
class Implementor
{
public:
    virtual ~Implementor() = default;
    virtual void operationImp() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Abstraction" %}
{% code fullWidth="true" %}
```cpp
class Abstraction
{
protected:
    shared_ptr<Implementor> implementor;
public:
    Abstraction(shared_ptr<Implementor> imp) : implementor(imp) {}
    virtual ~Abstraction() = default;
    virtual void operation() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteImplementor" %}
{% code fullWidth="true" %}
```cpp
class ConcreteImplementor : public Implementor
{
public:
    virtual void operationImp() override 
    { 
        cout << "Implementor;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Entity" %}
{% code fullWidth="true" %}
```cpp
class Entity : public Abstraction
{
public:
    using Abstraction::Abstraction;
    virtual void operation() override 
    {     
        cout << "Entity: "; 
        implementor->operationImp(); 
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    shared_ptr<Implementor> implementor = make_shared<ConcreteImplementor>();
    shared_ptr<Abstraction> abstraction = make_shared<Entity>(implementor);

    abstraction->operation();
}
```
{% endcode %}
