---
description: Adapter
---

# Возможные реализации для решения конкретных задач

{% tabs %}
{% tab title="Шаблон Adapter" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <vector>

using namespace std;

class Interface
{
public:
    virtual ~Interface() = default;
    virtual void request() = 0;
};

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

auto initialize()
{
    using InterPtr = shared_ptr<Interface>;
    
    vector<InterPtr> vec{
        make_shared<Adapter<AdapteeA>>(make_shared<AdapteeA>(), &AdapteeA::specRequestA),
        make_shared<Adapter<AdapteeB>>(make_shared<AdapteeB>(), &AdapteeB::specRequestB)
    };

    return vec;
}

int main()
{
    auto v = initialize();

    for (const auto& elem : v)
        elem->request();
}
```
{% endcode %}
{% endtab %}
{% endtabs %}
