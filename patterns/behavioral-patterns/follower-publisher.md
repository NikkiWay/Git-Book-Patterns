---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Подписчик-издатель

Паттерн Подписчик-издатель (Publisher-subscriber) – поведенческий шаблон проектирования, позволяющий установить зависимость между объектами таким образом, чтобы при изменении состояния одного объекта (Издателя) все зависимые от него объекты (Подписчики) автоматически уведомлялись и обновлялись. У издателя определены методы подписки/отписки на изменения его состояния. Используется, когда часто надо передавать какое-то обновление многим объектам на этапе выполнения программы.

## Решаемые задачи

* Уведомление об изменениях

Паттерн позволяет объектам автоматически уведомлять друг друга о произошедших изменениях или событиях, что помогает поддерживать согласованность данных между объектами.

* Разделение ответственностей

Паттерн позволяет отделить логику обработки событий от объектов, генерирующих эти события.

## Общая реализация на языке C++

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

{% code fullWidth="true" %}
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

## Преимущества

1. Издатели не зависят от конкретных подписчиков и могут отправлять уведомления или запросы без прямой зависимости от них.&#x20;
2. Подписчики могут  подписаться/отписаться на издателя во время  исполнения программы.

## Недостатки

1. Возможна утечка памяти при циклической зависимости.

{% hint style="info" %}
Циклическая зависимость происходит когда объект A подписывается на уведомления объекта B, а объект B в свою очередь подписывается на уведомления объекта A. Это означает, что каждый объект ссылается на другой объект, и они не могут быть удалены из памяти, так как они всегда ссылаются друг на друга. Такая ситуация может привести к утечке памяти, поскольку объекты будут продолжать существовать в памяти, даже если они больше не нужны.&#x20;
{% endhint %}

2. Произвольный порядок оповещения подписчиков.&#x20;
3. Необходимость держать список подписчиков. Издатели должны хранить список своих подписчиков для того, чтобы иметь возможность отправлять им уведомления или запросы. При удалении издателя, необходимо отписать всех подписчиков из списка, чтобы избежать утечек памяти и нежелательного взаимодействия.

## Связь с другими паттернами

1. Паттерн [Фасад](../structural-patterns/facade.md) может быть использован для создания удобного интерфейса для взаимодействия с паттерном. Фасад может скрыть сложности подписки, отписки и уведомлений, предоставляя простой интерфейс для взаимодействия с издателями и подписчиками.
2. Паттерн [Посредник](opekun.md) может быть использован вместе с паттерном для управления сложными взаимодействиями между объектами. Посредник может выступать в роли издателя, а другие объекты могут быть подписчиками, взаимодействуя через посредника.
3. Паттерн [Цепочка обязанностей](chain-of-responsibility.md) может быть использован для обработки уведомлений, полученных от издателей. Каждый объект в цепочке может быть подписчиком, который обрабатывает уведомления, а затем передает их следующему объекту в цепочке, если не может обработать сам.

\
