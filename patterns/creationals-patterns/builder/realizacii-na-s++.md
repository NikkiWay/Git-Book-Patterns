---
description: Builder
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
		cout << "Calling the Sedan constructor;" << endl; 
	}
	
	~Sedan() override 
	{ 
		cout << "Calling the Sedan destructor;" << endl; 
	}

	void drive() override 
	{ 
		cout << "Calling the drive method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="CarBuilder" %}
{% code fullWidth="true" %}
```cpp
class CarBuilder
{
public:
	virtual ~CarBuilder() = default;

	virtual bool buildEngine() = 0;
	virtual bool buildChassis() = 0;

	shared_ptr<Car> getCar();

protected:
	virtual shared_ptr<Car> createCar() = 0;

	shared_ptr<Car> car{ nullptr };
	size_t part{ 0 };
};
```
{% endcode %}
{% endtab %}

{% tab title="SedanBuilder" %}
{% code fullWidth="true" %}
```cpp
class SedanBuilder : public CarBuilder
{
public:
	bool buildEngine() override
	{
		if (!part)
			++part;

		if (part != 1) return false;
		
		cout << "Building part 1: Engine for Sedan;" << endl;
		return true;
	}
	
	bool buildChassis() override
	{
		if (part == 1)
			++part;

		if (part != 2) return false;

		cout << "Building part 2: Chassis for Sedan;" << endl;
	}

protected:
	shared_ptr<Product> createCar() override;
};
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
	virtual shared_ptr<Car> create() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="CarDirector" %}
{% code fullWidth="true" %}
```cpp
class CarDirector : public CarFactory
{
public:
	CarDirector(shared_ptr<CarBuilder> builder) : br(builder) {}

	shared_ptr<Car> create() override
	{
		if (br->buildEngine() && br->buildChassis()) return br->getCar();

		return nullptr;
	}

private:
	shared_ptr<CarBuilder> br;
};
```
{% endcode %}
{% endtab %}

{% tab title="Methods" %}
{% code fullWidth="true" %}
```cpp
# pragma region Methods
shared_ptr<Car> CarBuilder::getCar()
{
	if (!car) { car = createCar(); }

	return car;
}

shared_ptr<Car> SedanBuilder::createCar()
{
	if (part == 2) { car = make_shared<Sedan>(); }

	return car;
}

# pragma endregion
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
		shared_ptr<Car> car = factory->create();

		if (car)
			car->drive();
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
	shared_ptr<CarBuilder> builder = make_shared<SedanBuilder>();
	shared_ptr<CarFactory> factory = make_shared<CarDirector>(builder);

	User{}.use(factory);
}
```
{% endcode %}
