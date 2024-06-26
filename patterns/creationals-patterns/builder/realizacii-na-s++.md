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
	virtual shared_ptr<Car> create() = 0;

	shared_ptr<Car> car{ nullptr };
	size_t part{ 0 };
};


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
	shared_ptr<Car> create() override;
};
```
{% endcode %}
{% endtab %}

{% tab title="Methods" %}
{% code fullWidth="true" %}
```cpp
shared_ptr<Car> CarBuilder::getCar()
{
	if (!car) { car = create(); }

	return car;
}

shared_ptr<Car> SedanBuilder::create()
{
	if (part == 2) { car = make_shared<Sedan>(); }

	return car;
}
```
{% endcode %}
{% endtab %}

{% tab title="CarCreator" %}
{% code fullWidth="true" %}
```cpp
class CarCreator
{
public:
	virtual ~CarCreator() = default;
	virtual shared_ptr<Car> create() = 0;
};


class CarDirector : public CarCreator
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

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
	void use(shared_ptr<CarCreator>& creator)
	{
		shared_ptr<Car> car = creator->create();

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
	shared_ptr<CarCreator> creator = make_shared<CarDirector>(builder);

	User{}.use(creator);
}
```
{% endcode %}
