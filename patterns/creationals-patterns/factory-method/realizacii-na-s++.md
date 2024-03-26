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
    virtual void drive() = 0;
};


class Sedan : public Car
{
public:
    Sedan() 
    { 
        cout << "Sedan constructor called" << endl; 
    }
    
    ~Sedan() override 
    { 
        cout << "Sedan destructor called" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving sedan" << endl; 
    }
};

class SUV : public Car 
{
public:
    SUV() 
    {
        cout << "Calling the SUV constructor;" << endl;
    }
    
    ~SUV() override 
    { 
        cout << "Calling the SUV destructor;" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving SUV;" << endl; 
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

{% tab title="CarFactory" %}
{% code fullWidth="true" %}
```cpp
class CarFactory
{
public:
    virtual ~CarFactory() = default;
    virtual unique_ptr<Car> createCar() = 0;
};


template <Derivative<Car> TCar>
requires NotAbstract<TCar>
class ConcreteCarFactory : public CarFactory
{
public:
    unique_ptr<Car> createCar() override 
    {
        return make_unique<TCar>();
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="CarFactoryMaker" %}
{% code fullWidth="true" %}
```cpp
class CarFactoryMaker
{
public:
    template <Derivative<Car> TCar>
    NotAbstract<TCar>
    static unique_ptr<CarFactory> createCarFactory() 
    {
        return make_unique<ConcreteCarFactory<TCar>>();
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
    void use(shared_ptr<CarFactory>& factory)
    {
        if (!factory) throw runtime_error("The creator is missing!");

        shared_ptr<Car> car = factory->createCar();
        car->drive();
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
{% tab title="VehicleSolution" %}
{% code fullWidth="true" %}
```cpp
class VehicleSolution
{
public:
    using CreateCarMaker = unique_ptr<CarFactory>(&)();
    using CallBackMap = map<size_t, CreateCarFactory>;

public:
    VehicleSolution() = default;
    VehicleSolution(initializer_list<pair<size_t, CreateCarFactory>> list);

    bool registration(size_t id, CreateCarFactory createfun);
    bool check(size_t id) 
    { 
        return callbacks.erase(id) == 1; 
    }

    unique_ptr<CarFactory> create(size_t id);

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
VehicleSolution::VehicleSolution(initializer_list<pair<size_t, CreateCarFactory>> list)
{
    for (auto&& elem : list)
        this->registration(elem.first, elem.second);
}

bool VehicleSolution::registration(size_t id, CreateCarFactory createfun)
{
    return callbacks.insert(CallBackMap::value_type(id, createfun)).second;
}

unique_ptr<CarFactory> VehicleSolution::create(size_t id)
{
    CallBackMap::const_iterator it = callbacks.find(id);

    return it != callbacks.end() ? unique_ptr<CarFactory>(it->second()) : nullptr;
}

shared_ptr<VehicleSolution> make_solution(initializer_list<pair<size_t, VehicleSolution::CreateCarFactory>> list)
{
    return shared_ptr<VehicleSolution>(new VehicleSolution(list));
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
        shared_ptr<VehicleSolution> solution
        = make_solution({ {1, CarFactoryMaker::createCarFactory<Sedan>} });

        if (!solution->registration(2, CarFactoryMaker::createCarFactory<SUV>))
        {
            throw runtime_error("Error registration!");
        }
        shared_ptr<CarFactory> cr(solution->create(2));

        User{}.use(cr);
    }
    catch (runtime_error& err)
    {
        cout << err.what() << endl;
    }
}
```
{% endcode %}
