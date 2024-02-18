---
description: Property
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="includes" %}
{% code fullWidth="true" %}
```cpp
#include <iostream>
#include <memory>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="Property" %}
{% code fullWidth="true" %}
```cpp
template <typename Owner, typename Type>
class Property
{
    using Getter = Type(Owner::*)() const;
    using Setter = void (Owner::*)(const Type&);
private:
    Owner* owner;
    Getter methodGet;
    Setter methodSet;
public:
    Property() = default;
    Property(Owner* const owr, Getter getmethod, Setter setmethod) : owner(owr), methodGet(getmethod), methodSet(setmethod) {}
    void init(Owner* const owr, Getter getmethod, Setter setmethod)
    {
        owner = owr;
        methodGet = getmethod;
        methodSet = setmethod;
    }
    operator Type() // Getter
    { 
        return (owner->*methodGet)(); 
    }
    void operator=(const Type& data) // Setter
    { 
        (owner->*methodSet)(data); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Object" %}
{% code fullWidth="true" %}
```cpp
class Object
{
private:
    double value;
public:
    Object(double v) : value(v) 
    { 
        Value.init(this, &Object::getValue, &Object::setValue); 
    }
    double getValue() const 
    { 
        return value; 
    }
    void setValue(const double& v) 
    { 
        value = v; 
    }
    Property<Object, double> Value;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    Object obj(5.);
    cout << "value = " << obj.Value << endl;
    obj.Value = 10.;
    cout << "value = " << obj.Value << endl;
    unique_ptr<Object> ptr = make_unique<Object>(15.);
    cout << "value =" << ptr->Value << endl;
    obj = *ptr;
    obj.Value = ptr->Value;
}
```
{% endcode %}
