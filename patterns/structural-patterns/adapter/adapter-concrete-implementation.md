---
description: Adapter
---

# Возможные реализации для решения конкретных задач

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <vector>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="class Interface" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Interface
{
public:
    virtual ~Interface() = default;
    virtual void request() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="class Adapter" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <typename Type>
class Adapter : public Interface
{
public:
    using MethodPtr = void (Type::*)();

    Adapter(shared_ptr<Type> o, MethodPtr m) : object(o), method(m) {}

    void request() override 
    { 
    ((*object).*method)(); 
    }

private:
    shared_ptr<Type> object;
    MethodPtr method;
};
```
{% endcode %}
{% endtab %}

{% tab title="class AdapteeA" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class AdapteeA
{
public:
    ~AdapteeA() 
    { 
        cout << "Destructor class AdapteeA;" << endl; 
    }
    void specRequestA() 
    { 
        cout << "Method AdapteeA::specRequestA;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="class AdapteeB" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class AdapteeB
{
public:
    ~AdapteeB() 
    { 
        cout << "Destructor class AdapteeB;" << endl; 
    }
    void specRequestB() 
    { 
        cout << "Method AdapteeB::specRequestB;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="initialize()" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
auto initialize()
{
    using InterPtr = shared_ptr<Interface>;
    
    vector<InterPtr> vec{
        make_shared<Adapter<AdapteeA>>(make_shared<AdapteeA>(), &AdapteeA::specRequestA),
        make_shared<Adapter<AdapteeB>>(make_shared<AdapteeB>(), &AdapteeB::specRequestB)
    };

    return vec;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    auto v = initialize();

    for (const auto& elem : v)
        elem->request();
}
```
{% endcode %}