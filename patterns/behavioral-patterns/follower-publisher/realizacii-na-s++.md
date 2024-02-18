---
description: Publisher-subscriber
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="Subscriber" %}
{% code fullWidth="true" %}
```cpp
class Subscriber
{
public:
	virtual ~Subscriber() = default;

	virtual void method() = 0;
};

using Reseiver = Subscriber;
```
{% endcode %}
{% endtab %}

{% tab title="Publisher" %}
{% code fullWidth="true" %}
```cpp
class Publisher
{
	using Action = void(Reseiver::*)();
	
	using Pair = pair<shared_ptr<Reseiver>, Action>;
private:
	vector<Pair> callback;

	int indexOf(shared_ptr<Reseiver> r);

public:
	bool subscribe(shared_ptr<Reseiver> r, Action a);
	bool unsubscribe(shared_ptr<Reseiver> r);
	
	void run();
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteSubscriber" %}
{% code fullWidth="true" %}
```cpp
class ConcreteSubscriber : public Subscriber
{
public:
	virtual void method() override 
	{ 
		cout << "method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Methods Publisher" %}
{% code fullWidth="true" %}
```cpp
#pragma region Methods Publisher

bool Publisher::subscribe(shared_ptr<Reseiver> r, Action a)
{
	if (indexOf(r) != -1) return false;

	Pair pr(r, a);

	callback.push_back(pr);

	return true;
}

bool Publisher::unsubscribe(shared_ptr<Reseiver> r)
{
	int pos = indexOf(r);

	if (pos != -1)
		callback.erase(callback.begin() + pos);

	return pos != -1;
}

void Publisher::run()
{
	cout << "Run:" << endl;
	for (auto elem : callback)
		((*elem.first).*(elem.second))();
}

int Publisher::indexOf(shared_ptr<Reseiver> r)
{
	int i = 0;
	for (auto it = callback.begin(); it != callback.end() && r != (*it).first; i++, ++it);

	return i < callback.size() ? i : -1;
}

#pragma endregion
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <vector>

using namespace std;

int main()
{
	shared_ptr<Subscriber> subscriber1 = make_shared<ConSubscriber>();
	shared_ptr<Subscriber> subscriber2 = make_shared<ConSubscriber>();
	shared_ptr<Publisher> publisher = make_shared<Publisher>();

	publisher->subscribe(subscriber1, &Subscriber::method);
	if (publisher->subscribe(subscriber2, &Subscriber::method))
		publisher->unsubscribe(subscriber1);

	publisher->run();
}
```
{% endcode %}
