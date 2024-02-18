---
description: Composite
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="includes" %}
{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <initializer_list>
# include <memory>
# include <vector>
```
{% endcode %}
{% endtab %}

{% tab title="usings" %}
{% code fullWidth="true" %}
```cpp
using namespace std;

class Component

using PtrComponent = shared_ptr<Component>;
using VectorComponent = vector<PtrComponent>;
```
{% endcode %}
{% endtab %}

{% tab title="Component" %}
{% code fullWidth="true" %}
```cpp
class Component
{
public:
    using value_type = Component;
    using size_type = size_t;
    using iterator = VectorComponent::const_iterator;
    using const_iterator = VectorComponent::const_iterator;

    virtual ~Component() = default;

    virtual void operation() = 0;

    virtual bool isComposite() const 
    { 
        return false; 
    }
    virtual bool add(initializer_list<PtrComponent> comp) 
    { 
        return false; 
    }
    virtual bool remove(const iterator& it) 
    { 
        return false; 
    }
    virtual iterator begin() const 
    { 
        return iterator(); 
    }
    virtual iterator end() const 
    { 
        return iterator(); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Figure" %}
{% code fullWidth="true" %}
```cpp
class Figure : public Component
{
public:
    virtual void operation() override 
    { 
        cout << "Figure method;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Camera" %}
{% code fullWidth="true" %}
```cpp
class Camera : public Component
{
public:
    virtual void operation() override 
    { 
        cout << "Camera method;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Composite" %}
{% code fullWidth="true" %}
```cpp
class Composite : public Component
{
private:
    VectorComponent vec;
public:
    Composite() = default;
    Composite(PtrComponent first, ...);

    void operation() override;

    bool isComposite() const override 
    { 
        return true; 
    }
    bool add(initializer_list<PtrComponent> list) override;
    bool remove(const iterator& it) override 
    { 
        vec.erase(it); return true; 
    }
    iterator begin() const override 
    { 
        return vec.begin(); 
    }
    iterator end() const override 
    { 
        return vec.end(); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Composite Methods" %}
{% code fullWidth="true" %}
```cpp
# pragma region Methods
Composite::Composite(PtrComponent first, ...)
{
    for (shared_ptr<Component>* ptr = &first; *ptr; ++ptr)
        vec.push_back(*ptr);
}

void Composite::operation()
{
    cout << "Composite method:" << endl;
    for (auto elem : vec)
        elem->operation();
}

bool Composite::add(initializer_list<PtrComponent> list)
{
    for (auto elem : list)
        vec.push_back(elem);
return true;
}
# pragma endregion
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    using Default = shared_ptr<Component>;
    PtrComponent fig = make_shared<Figure>(), cam = make_shared<Camera>();
    auto composite1 = make_shared<Composite>(fig, cam, Default{});

    composite1->add({ make_shared<Figure>(), make_shared<Camera>() });
    composite1->operation();
    cout << endl;

    auto it = composite1->begin();

    composite1->remove(++it);
    composite1->operation();
    cout << endl;

    auto composite2 = make_shared<Composite>(make_shared<Figure>(), composite1, Default());

    composite2->operation();
}
```
{% endcode %}
