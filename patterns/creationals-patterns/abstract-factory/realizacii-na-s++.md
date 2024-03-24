---
description: Abstract factory
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="BaseGraphics" %}
{% code fullWidth="true" %}
```cpp
class BaseGraphics 
{
public:
    virtual ~BaseGraphics() = 0;
};

BaseGraphics::~BaseGraphics() {}
```
{% endcode %}
{% endtab %}

{% tab title="QtGraphics" %}
{% code fullWidth="true" %}
```cpp
class QtGraphics : public BaseGraphics
{
public:
	QtGraphics(shared_ptr<Image> im) 
	{
		cout << "Calling the QtGraphics constructor;" << endl; 
	}
	
	~QtGraphics() override 
	{ 
		cout << "Calling the QtGraphics destructor;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="AbstractGraphFactory" %}
{% code fullWidth="true" %}
```cpp
class AbstractGraphFactory
{
public:
	virtual ~AbstractGraphFactory() = default;

	virtual unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) = 0;
	virtual unique_ptr<BasePen> createPen(shared_ptr<Color> cl) = 0;
	virtual unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="QtGraphFactory" %}
{% code fullWidth="true" %}
```cpp
class QtGraphFactory : public AbstractGraphFactory
{
public:
	unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) override
	{
		return make_unique<QtGraphics>(im);
	}

	unique_ptr<BasePen> createPen(shared_ptr<Color> cl) override
	{
		return make_unique<QtPen>();
	}

	unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) override
	{
		return make_unique<QtBrush>();
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
	void use(shared_ptr<AbstractGraphFactory>& cr)
	{
		shared_ptr<Image> image = make_shared<Image>();
		auto graphics = cr->createGraphics(image);
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Pen" %}
{% code fullWidth="true" %}
```cpp
class BasePen {};

class QtPen : public BasePen {};
```
{% endcode %}
{% endtab %}

{% tab title="Brush" %}
{% code fullWidth="true" %}
```cpp
class BaseBrush {};

class QtBrush : public BaseBrush {};
```
{% endcode %}
{% endtab %}

{% tab title="Image" %}
{% code fullWidth="true" %}
```cpp
class Image {};
```
{% endcode %}
{% endtab %}

{% tab title="Color" %}
{% code fullWidth="true" %}
```cpp
class Color {};
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
	shared_ptr<AbstractGraphFactory> grfactory = make_shared<QtGraphFactory>();

	unique_ptr<User> us = make_unique<User>();

	us->use(grfactory);
}
```
{% endcode %}
