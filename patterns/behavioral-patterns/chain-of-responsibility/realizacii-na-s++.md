---
description: Chain of Responsibility
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="AbstractHandler" %}
{% code fullWidth="true" %}
```cpp
class AbstractHandler
{
	using PtrAbstractHandler = shared_ptr<AbstractHandler>;
protected:
	PtrAbstractHandler next;

	virtual bool run() = 0;

public:
	using Default = shared_ptr<AbstractHandler>;

	virtual ~AbstractHandler() = default;

	virtual bool handle() = 0;

	void add(PtrAbstractHandler node);
	void add(PtrAbstractHandler node1, PtrAbstractHandler node2, ...);
	void add(initializer_list<PtrAbstractHandler> list);
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteHandler" %}
{% code fullWidth="true" %}
```cpp
class ConcreteHandler : public AbstractHandler
{
private:
	bool condition
	{ 
		false 
	};

protected:
	virtual bool run() override 
	{ 
		cout << "Method run;" << endl; 
		return true; 
	}

public:
	ConcreteHandler() : ConcreteHandler(false) {}
	
	ConcreteHandler(bool c) : condition(c) 
	{ 
		cout << "Constructor;" << endl; 
	}
	
	virtual ~ConcreteHandler() override 
	{ 
		cout << "Destructor;" << endl; 
	}

	virtual bool handle() override
	{
		if (!condition) return next ? next->handle() : false;

		return run();
	}

};
```
{% endcode %}
{% endtab %}

{% tab title="Methods" %}
{% code fullWidth="true" %}
```cpp
#pragma region Methods

void AbstractHandler::add(PtrAbstractHandler node)
{
	if (next)
		next->add(node);
	else
		next = node;
}

void AbstractHandler::add(PtrAbstractHandler node1, PtrAbstractHandler node2, ...)
{
	for (Default* ptr = &node1; *ptr; ++ptr)
		add(*ptr);
}

void AbstractHandler::add(initializer_list<PtrAbstractHandler> list)
{
	for (auto elem : list)
		add(elem);
}

#pragma endregion
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <initializer_list>
# include <memory>

using namespace std;

int main()
{
	shared_ptr<AbstractHandler> chain = make_shared<ConcreteHandler>();

	chain->add(
		{
		make_shared<ConcreteHandler>(false),
		make_shared<ConcreteHandler>(true),
		make_shared<ConcreteHandler>(true),
		AbstractHandler::Default()
		}
	);

	cout << "Result = " << chain->handle() << ";" << endl;
}
```
{% endcode %}
