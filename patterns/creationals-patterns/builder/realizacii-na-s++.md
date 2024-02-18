---
description: Builder
---

# Реализации на С++

## Общая реализация на языке С++

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

{% code lineNumbers="true" fullWidth="true" %}
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
