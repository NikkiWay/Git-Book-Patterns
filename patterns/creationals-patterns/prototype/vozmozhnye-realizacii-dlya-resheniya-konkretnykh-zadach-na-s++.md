# Возможные реализации для решения конкретных задач на С++

## Прототип. Идиома NVI (Non-Virtual Interface)

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
	virtual ~Car() = default;
	unique_ptr<Car> clone();

private:
	virtual unique_ptr<Car> doClone() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Sedan" %}
{% code fullWidth="true" %}
```cpp
class Sedan : public Car
{
public:
	Sedan() 
	{ 
		cout << "Calling the default constructor;" << endl; 
	}
	
	Sedan(const Sedan& car) 
	{ 
		cout << "Calling the Copy constructor;" << endl; 
	}
	
	~Sedan() override 
	{ 
		cout << "Calling the destructor;" << endl; 
	}

private:
	unique_ptr<Car> doClone() override
	{
		return make_unique<Sedan>(*this);
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="LuxurySedan" %}
{% code fullWidth="true" %}
```cpp
class LuxurySedan : public Sedan { };

unique_ptr<Car> Car::clone()
{
	unique_ptr<Car> carClone = doClone();

	if (typeid(*carClone) != typeid(*this)) throw runtime_error("Error clone!");
	
	return carClone;
}
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
	void use(shared_ptr<Car>& car)
	{
		auto newCar = car->clone();
	}
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <exception>

using namespace std;

int main()
{
	try
	{
		shared_ptr<Car> luxsedan = make_shared<LuxurySedan>();
		User{}.use(luxsedan);
	}
	catch (exception& err)
	{
		cout << err.what() << endl;
	}
}
```
{% endcode %}

## Прототип (Prototype). «Статический полиморфизм» (CRTP)

{% tabs %}
{% tab title="Base" %}
{% code fullWidth="true" %}
```cpp
class Base
{
public:
    virtual ~Base() = default;

    virtual unique_ptr<Base> clone() const = 0;
    virtual ostream& print(ostream& os) const = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Cloner" %}
{% code fullWidth="true" %}
```cpp
template <typename Type>
concept Abstract = is_abstract_v<Type>;

template <Abstract Base, typename Derived>
class Cloner : public Base
{
public:
    unique_ptr<Base> clone() const override
    {
        return make_unique<Derived>(static_cast<const Derived&>(*this));
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Cloner in C++23" %}
{% code fullWidth="true" %}
```cpp
template <typename Base>
struct Cloner : public Base
{
    template <typename Self>
    unique_ptr<Base> clone(this Selt&& self) const override
    {
        return unique_ptr<Base>(new Self(self));
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car : public Cloner<Base, Car>
{
private:
    int serialNum;

public:
    Car(int d) : serialNum(d) 
    { 
        cout << "Calling the constructor;" << endl; 
    }
    
    Car(const Car& car) : serialNum(car.serialNum)
    {
        cout << "Calling the Copy constructor;" << endl;
    }
    
    ~Car() override 
    { 
        cout << "Calling the destructor;" << endl; 
    }

    ostream& print(ostream& os) const override
    {
        return os << "serialNum = " << serialNum;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Car in C++23" %}
{% code fullWidth="true" %}
```cpp
class Car : public Cloner<Base>
{
private:
    int serialNum;

public:
    Base(int d) : serialNum(d) {}

    ostream& print(ostream& os) const override
    {
        return os << "serialNum = " << serialNum;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="operator <<" %}
{% code fullWidth="true" %}
```cpp
ostream& operator <<(ostream& os, const unique_ptr<Base>& obj)
{
    return obj->print(os);
}
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
    void use(unique_ptr<Base>& vehicle)
    {
        auto newVehicle = vehicle->clone();

        cout << newVehicle << endl;
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <concepts>

using namespace std;

int main()
{
    unique_ptr<Base> vehicle = make_unique<Car>(10);
    User{}.use(vehicle);
}
```
{% endcode %}

## Прототип (Prototype). «Статический полиморфизм» (CRTP) и множественное наследование

{% tabs %}
{% tab title="Base" %}
{% code fullWidth="true" %}
```cpp
class Base
{
public:
    virtual ~Base() = default;

    unique_ptr<Base> clone() const;
    virtual ostream& print(ostream& os) const = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="BaseCloner" %}
{% code fullWidth="true" %}
```cpp
class BaseCloner
{
public:
    virtual unique_ptr<Base> doClone() const = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Cloner" %}
{% code fullWidth="true" %}
```cpp
template <typename Derived>
class Cloner : public BaseCloner
{
public:
    unique_ptr<Base> doClone() const override
    {
        return make_unique<Derived>(static_cast<const Derived&>(*this));
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car : public Base, public Cloner<Car>
{
private:
    int serialNum;

public:
    Car(int d) : serialNum(d) 
    { 
        cout << "Calling the constructor;" << endl; 
    }
    
    Car(const Car& car) : serialNum(car.serialNum)
    {
        cout << "Calling the Copy constructor;" << endl;
    }
    
    ~Car() override 
    { 
        cout << "Calling the destructor;" << endl; 
    }

    ostream& print(ostream& os) const override
    {
        return os << "serialNum = " << serialNum;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="clone()" %}
{% code fullWidth="true" %}
```cpp
unique_ptr<Base> Base::clone() const
{
    return dynamic_cast<const BaseCloner*>(this)->doClone();
}
```
{% endcode %}
{% endtab %}

{% tab title="operator <<" %}
{% code fullWidth="true" %}
```cpp
ostream& operator <<(ostream& os, const unique_ptr<Base>& obj)
{
    return obj->print(os);
}
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
    void use(unique_ptr<Base>& vehicle)
    {
        auto newVehicle = vehicle->clone();

        cout << newVehicle << endl;
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <concepts>

using namespace std;

int main()
{
    unique_ptr<Base> vehicle = make_unique<Car>(10);
    User{}.use(vehicle);
}
```
{% endcode %}
