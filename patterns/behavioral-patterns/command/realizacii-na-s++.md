---
description: Command
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="Command" %}
{% code fullWidth="true" %}
```cpp
template <typename Reseiver>
class Command
{
public:
	virtual ~Command() = default;
	
	virtual void execute(shared_ptr<Reseiver>) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteCommand" %}
{% code fullWidth="true" %}
```cpp
template <typename Reseiver>
class ConcreteCommand : public Command<Reseiver>
{
	using Action = void(Reseiver::*)();
private:
	Action act;

public:
	ConcreteCommand(Action a) : act(a) {}

	virtual void execute(shared_ptr<Reseiver> r) override { ((*r).*act)(); }
};
```
{% endcode %}
{% endtab %}

{% tab title="Object" %}
{% code fullWidth="true" %}
```cpp
class Object
{
public:
    virtual void run() = 0; 
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteObject" %}
{% code fullWidth="true" %}
```cpp
class ConcreteObject : public Object
{
public:
	virtual void run() override 
	{ 
		cout << "Run method;" << endl; 
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
	shared_ptr<Command<Object>> command = make_shared<SimpleCommand<Object>>(&Object::run);

	shared_ptr<Object> obj = make_shared<ConObject>();

	command->execute(obj);
}
```
{% endcode %}
