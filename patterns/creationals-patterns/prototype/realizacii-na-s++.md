---
description: Prototype
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="BaseObject" %}
{% code fullWidth="true" %}
```cpp
class BaseObject
{
public:
    virtual ~BaseObject() = default;

    virtual unique_ptr<BaseObject> clone() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteObject" %}
{% code fullWidth="true" %}
```cpp
class ConcreteObject : public BaseObject
{
public:
    ConcreteObject() 
    {
        cout << "Default constructor;" << endl; 
    }
    
    ConcreteObject(const Object1& obj) 
    { 
        cout << "Copy constructor;" << endl; 
    }
    
    ~ConcreteObject() 
    { 
        cout << "Destructor;" << endl; 
    }

    virtual unique_ptr<BaseObject> clone() override
    {
        return make_unique<ConcreteObject>(*this)
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>

using namespace std;

int main()
{
    shared_ptr<BaseObject> ptr1 = make_shared<ConcreteObject>();    
    auto ptr2 = ptr1->clone();
}
```
{% endcode %}
