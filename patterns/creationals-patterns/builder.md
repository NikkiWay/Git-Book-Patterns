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

# Строитель

## Назначение

Строитель — это порождающий паттерн проектирования, который позволяет создавать сложные объекты пошагово. Разбив процесс конструирования сложного объекта на отдельные шаги, паттерн позволяет создавать сложные объекты с различными наборами параметров и настроек.

{% hint style="info" %}
Примером сложного объекта может служить объект почтового сообщения Email. У этого объекта может быть множество параметров, таких как отправитель, получатель, тема, текст, вложения и т.д. Используя паттерн "Строитель", мы можем поэтапно создать объект почтового сообщения с различными параметрами.
{% endhint %}

## Решаемые задачи

* Создание сложного объекта с различными вариантами конфигурации

Появляется возможность создавать сложные объекты пошагово с разными вариантами конфигурации. Каждый шаг строителя определяет значения и настройки для соответствующей части объекта.

* Отделение процесса контроля за созданием объекта от самого процесса создания объекта.&#x20;

Одна сущность (строитель) определяет шаги конструирования объекта, в то время как другая сущность (директор) управляет последовательностью этих шагов, обеспечивая создание объекта с нужной конфигурацией.

## Общая реализация на языке C++

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
	Product() 
	{ 
		cout << "Calling the ConProd1 constructor;" << endl; 
	}
	
	~Product()
	{ 
		cout << "Calling the ConProd1 destructor;" << endl; 
	}
	
	void run() 
	{ 
		cout << "Calling the run method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Builder" %}
{% code fullWidth="true" %}
```cpp
class Builder
{
public:
	virtual bool buildPart1() = 0;
	virtual bool buildPart2() = 0;

	shared_ptr<Product> getProduct();

protected:
	virtual shared_ptr<Product> createProduct() = 0;

	shared_ptr<Product> product;
};

shared_ptr<Product> Builder::getProduct()
{
	if (!product) 
	{ 
		product = createProduct(); 
	}
	return product;
}
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteBuilder" %}
{% code fullWidth="true" %}
```cpp
class ConcreteBuilder : public Builder
{
public:
	virtual bool buildPart1() override
	{ 
		cout << "Completed part: " << ++part << ";" << endl;
		return true;
	}
	
	virtual bool buildPart2() override
	{ 
		cout << "Completed part: " << ++part << ";" << endl;
		return true;
	}

protected:
	virtual shared_ptr<Product> createProduct() override;

private:
	size_t part{ 0 };
};

shared_ptr<Product> ConcreteBuilder::createProduct()
{
	if (part == 2) 
	{ 
		product = make_shared<Product>(); 
	}
	return product;
}
```
{% endcode %}
{% endtab %}

{% tab title="Director" %}
{% code fullWidth="true" %}
```cpp
class Director
{
public:
	shared_ptr<Product> create(shared_ptr<Builder> builder)
	{
		if (builder->buildPart1() && builder->buildPart2()) 
			return builder->getProduct();
			
		return shared_ptr<Product>();
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

using namespace std;

int main()
{
	shared_ptr<Builder> builder = make_shared<ConBuilder>();
	shared_ptr<Director> director = make_shared<Director>();
	shared_ptr<Product> prod = director->create(builder);
	if (prod)
		prod->run();
}
```
{% endcode %}

## Преимущества

1. Разделение ответственности по созданию объекта.
2. Сокрытие сложной логики создания объекта.&#x20;
