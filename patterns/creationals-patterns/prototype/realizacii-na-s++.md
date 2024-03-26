---
description: Prototype
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
	virtual unique_ptr<Car> clone() = 0;
};


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

	unique_ptr<Car> clone() override
	{
		return make_unique<Sedan>(*this);
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
	void use(shared_ptr<Car>&car)
	{
		auto newCar = car->clone();
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
	shared_ptr<Car> sedan = make_shared<Sedan>();
	User{}.use(sedan);
}
```
{% endcode %}
