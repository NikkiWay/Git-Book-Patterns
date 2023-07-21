---
description: Strategy
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Стратегия

## Назначение

Паттерн Стратегия (Strategy) – это поведенческий шаблон проектирования, который предназначен для определения реализации алгоритмов, инкапсуляции каждого из них и обеспечения их взаимозаменяемости. Это позволяет легко подменять и модифицировать алгоритмы во время выполнения программы без изменения класса, который их использует. Используется для того, чтобы клиентский код мог выбирать нужный вариант раелизации алгоритма в зависимости от контекста или условий. Каждый вариант реализации алгоритма выделяется в отдельную стратегию.

{% hint style="info" %}
Примером такого алгоритма может быть алгоритм обработки заказов в интернет-магазине. В зависимости от способа доставки заказа (курьерская доставка, почта, самовывоз), нужно применить разные варианты расчета стоимости доставки.
{% endhint %}

## Решаемые задачи

* Разделение алгоритма на отдельные классы

Паттерн позволяет выделить различные варианты реализации алгоритма в отдельные классы, что упрощает структуру кода и делает его более модульным.

* Подмена алгоритма во время выполнения программы

Появляется возможность во время выполнения программы динамически выбирать и подменять нужную стратегию и передавать ее в основной класс для выполнения операции.

* Изменение поведения объекта

Паттер позволяет изменять поведение объекта без изменения самого объекта или его наследников.

## Общая реализация на языке C++

{% tabs %}
{% tab title="Strategy" %}
{% code fullWidth="true" %}
```cpp
class Strategy
{
public:
	virtual void algorithmSort(shared_ptr<double[]> ar, unsigned cnt) = 0;
};

```
{% endcode %}
{% endtab %}

{% tab title="BustStrategy" %}
{% code fullWidth="true" %}
```cpp
template <typename TComparison>
class BustStrategy : public Strategy
{
public:
	virtual void algorithmSort(shared_ptr<double[]> ar, unsigned cnt) override
	{
		for (int i = 0; i < cnt - 1; i++)
			for (int j = i + 1; j < cnt; j++)
			{
				if (TComparison::compare(ar[i], ar[j]) > 0)
					swap(ar[i], ar[j]);
			}
	}

};
```
{% endcode %}
{% endtab %}

{% tab title="Array" %}
{% code fullWidth="true" %}
```cpp
class Array
{
public:
	Array(initializer_list<double> list)
	{
		this->count = list.size();
		this->arr = shared_ptr<double[]>(new double[this->count]);
	
		unsigned i = 0;
		for (auto elem : list)
			arr[i++] = elem;
	}

	void sort(shared_ptr<Strategy> algorithm
	{
		algorithm->algorithmSort(this->arr, this->count);
	}

	const double& operator [](int index) const 
	{ 
		return this->arr[index]; 
	}
	
	unsigned size() const 
	{ 
		return count; 
	}

private:
	shared_ptr<double[]> arr;
	unsigned count;
};
```
{% endcode %}
{% endtab %}

{% tab title="Comparison" %}
{% code fullWidth="true" %}
```cpp
template <typename Type>
class Comparison
{
public:
	static int compare(const Type& elem1, const Type& elem2) { return elem1 - elem2; }
};

ostream& operator <<(ostream& os, const Array& ar)
{
	for (int i = 0; i < ar.size(); i++)
		os << " " << ar[i];
	return os;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <initializer_list>

using namespace std;

void main()
{
	using TStrategy = BustStrategy<Comparison<double>>;
	shared_ptr<Strategy> strategy = make_shared<TStrategy>();

	Array ar{ 8., 6., 4., 3., 2., 7., 1. };

	ar.sort(strategy);

	cout << ar << endl;
}
```
{% endcode %}

## Преимущества

1. Инкапсуляция реализации различных алгоритмов.
2. Вызов всех алгоритмов одним стандартным образом. Все конкретные стратегии реализуют общий интерфейс. Это позволяет вызывать все алгоритмы одним стандартным образом, независимо от конкретной стратегии.
3. Возможность подмены алгоритмов во время выполнения.

## Недостатки

1. Конкретная стратегия может не работать с данными определенного класса, что приводит к появлению зависимостей между конкретными сущностями и стратегиями.
2. Увеличение количества кода за счёт появления параллельных иерархий классов.

## Взаимодействие с другими паттернами

* Паттерн [Фабричный метод](../creationals-patterns/factory-method.md)может использоваться для создания объектов стратегий.
* Паттерн [Декоратор](../structural-patterns/dekorator.md) может использоваться для добавления дополнительного поведения или функциональности к конкретным стратегиям. Декоратор позволяет обернуть стратегию в другой объект и добавить ему дополнительные возможности, не изменяя саму стратегию.
* Паттерн [Команда](command.md) может использоваться для инкапсуляции вызова стратегии в отдельный объект команды. Команда может содержать ссылку на объект стратегии и вызывать его методы при выполнении команды. Это позволяет легко переключаться между разными стратегиями, используя различные команды.
