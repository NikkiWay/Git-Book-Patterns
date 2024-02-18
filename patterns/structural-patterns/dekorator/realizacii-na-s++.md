---
description: Decorator
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

{% tab title="Component" %}
{% code fullWidth="true" %}
```cpp
class Component
{
public:
    virtual ~Component() = default;
    virtual void operation() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteComponent" %}
{% code fullWidth="true" %}
```cpp
class ConcreteComponent : public Component
{
public:
    void operation() override 
    { 
        cout << "ConcreteComponent; "; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Decorator" %}
{% code fullWidth="true" %}
```cpp
class Decorator : public Component
{
protected:
    shared_ptr<Component> component;
public:
    Decorator(shared_ptr<Component> comp) : component(comp) {}
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteDecorator" %}
{% code fullWidth="true" %}
```cpp
class ConcreteDecorator : public Decorator
{
public:
    using Decorator::Decorator;
    void operation() override;
};

# pragma region Method
void ConcreteDecorator::operation()
{
    if (component)
    {
        component->operation();
        cout << "ConDecorator; ";
    }
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
    shared_ptr<Component> component = make_shared<ConcreteComponent>();
    shared_ptr<Component> decorator1 = make_shared<ConcreteDecorator>(component);

    decorator1->operation();
    cout << endl;

    shared_ptr<Component> decorator2 = make_shared<ConcreteDecorator>(decorator1);

    decorator2->operation();
    cout << endl;
}
```
{% endcode %}
