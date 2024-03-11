---
description: Factory method
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;
    virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Audi" %}
{% code fullWidth="true" %}
```cpp
class Audi : public Car
{
public:
    Audi() 
    { 
        cout << "Calling the Audi constructor;" << endl; 
    }
    
    ~Audi() override 
    { 
        cout << "Calling the Audi destructor;" << endl; 
    }

    void run() override 
    { 
        cout << "Calling the run method Audi;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Bmw" %}
{% code fullWidth="true" %}
```cpp
class Bmw : public Car
{
public:
    Bmw() 
    { 
        cout << "Calling the Bmw constructor;" << endl; 
    }
    
    ~Bmw() override 
    { 
        cout << "Calling the Bmw destructor;" << endl; 
    }

    void run() override 
    { 
        cout << "Calling the run method Bmw;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
{% code fullWidth="true" %}
```cpp
template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;
```
{% endcode %}
{% endtab %}

{% tab title="CarMaker" %}
{% code fullWidth="true" %}
```cpp
class CarMaker
{
public:
    virtual ~CarMaker() = default;
    virtual unique_ptr<Car> createCar() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteCarMaker" %}
{% code fullWidth="true" %}
```cpp
template <Derivative<Car> Tprod>
requires NotAbstract<Tprod>
class ConcreteCarMaker : public CarMaker
{
public:
    unique_ptr<Car> createCar() override 
    {
        return make_unique<Tprod>();
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="CrCarMaker" %}
{% code fullWidth="true" %}
```cpp
class CrCarMaker
{
public:
    template <Derivative<Product> Tprod>
    NotAbstract<Tprod>
    static unique_ptr<CarMaker> createConcreteCarMaker() 
    {
        return make_unique<ConcreteCarMaker<Tprod>>();
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="CarUser" %}
{% code fullWidth="true" %}
```cpp
class CarUser
{
public:
    void use(shared_ptr<CarMaker>& cr)
    {
        if (!cr) throw runtime_error("The creator is missing!");

        shared_ptr<Car> ptr = cr->createCar();
        ptr->run();
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Класс Solution выполняет роль посредника между клиентским кодом и классами создателей продуктов. Он отвечает за регистрацию методов создания объектов для каждого типа продукта и предоставляет методы для создания объектов по их идентификаторам.
{% endhint %}

{% tabs %}
{% tab title="Solution" %}
{% code fullWidth="true" %}
```cpp
class Solution
{
public:
    using CreateCarMaker = unique_ptr<CarMaker>(&)();
    using CallBackMap = map<size_t, CreateCarMaker>;

public:
    Solution() = default;
    Solution(initializer_list<pair<size_t, CreateCarMaker>> list);

    bool registration(size_t id, CreateCarMaker createfun);
    bool check(size_t id) 
    { 
        return callbacks.erase(id) == 1; 
    }

    unique_ptr<CarMaker> create(size_t id);

private:
    CallBackMap callbacks;
};
```
{% endcode %}
{% endtab %}

{% tab title="Methods" %}
{% code fullWidth="true" %}
```cpp
# pragma region Solution
Solution::Solution(initializer_list<pair<size_t, CreateCarMaker>> list)
{
    for (auto&& elem : list)
        this->registration(elem.first, elem.second);
}

bool Solution::registration(size_t id, CreateCarMaker createfun)
{
    return callbacks.insert(CallBackMap::value_type(id, createfun)).second;
}

unique_ptr<CarMaker> Solution::create(size_t id)
{
    CallBackMap::const_iterator it = callbacks.find(id);

    return it != callbacks.end() ? unique_ptr<CarMaker>(it->second()) : nullptr;
}

shared_ptr<Solution> make_solution(initializer_list<pair<size_t, Solution::CreateCarMaker>> list)
{
    return shared_ptr<Solution>(new Solution(list));
}
# pragma endregion
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <initializer_list>
# include <memory>
# include <map>
# include <exception>

using namespace std;

int main()
{
    try
    {
        shared_ptr<Solution> solution
        CrCarMaker::createConcreteCarMaker<Audi>} });

        if (!solution->registration(2, CrCarMaker::createConcreteCarMaker<Bmw>))
        {
            throw runtime_error("Error registration!");
        }
        shared_ptr<CarMaker> cr(solution->create(2));

        User{}.use(cr);
    }
    catch (runtime_error& err)
    {
        cout << err.what() << endl;
    }
}
```
{% endcode %}
