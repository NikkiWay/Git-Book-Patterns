---
description: Template Method
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="AbstractClass" %}
{% code fullWidth="true" %}
```cpp
class AbstractClass
{
public:
    void templateMethod()
    {
        primitiveOperation();
        concreteOperation();
        hook();
    }
    virtual ~AbstractClass() = default;
protected:
    virtual void primitiveOperation() = 0;
    void concreteOperation() 
    { 
        cout << "concreteOperation;" << endl; 
    }
    virtual void hook() 
    { 
        cout << "hook Base;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteClassA" %}
{% code fullWidth="true" %}
```cpp
class ConcreteClassA : public AbstractClass
{
protected:
    void primitiveOperation() override 
    { 
        cout << "primitiveOperation A;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteClassB" %}
{% code fullWidth="true" %}
```cpp
class ConcreteClassB : public AbstractClass
{
protected:
    void primitiveOperation() override 
    { 
        cout << "primitiveOperation B;" << endl; 
    }
    void hook() override 
    { 
        cout << "hook B;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
#include <iostream>

using namespace std;

int main()
{
    ConcreteClassA ca;
    ConcreteClassB cb;

    ca.templateMethod();
    cb.templateMethod();
}
```
{% endcode %}
