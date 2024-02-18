---
description: Factory method
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

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
      virtual ~Product() = 0;
      
      virtual void run() = 0;
};
 
Product::~Product() {}
```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
{% code fullWidth="true" %}
```cpp
class Creator
{
public:
      virtual ~Creator() = 0;
      
      virtual unique_ptr<Product> createProduct() = 0;
};

Creator::~Creator() = default;
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteCreator" %}
{% code fullWidth="true" %}
```cpp
template <typename Tprod>
class ConcreteCreator : public Creator
{
public:
      virtual unique_ptr<Product> createProduct() override
      {
            return unique_ptr<Product>(new Tprod());
      }
};  
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProduct1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProduct1 : public Product
{
public:
      virtual ~ConcreteProduct1() override 
      {
            cout << "Destructor;" << endl; 
      }
      
      virtual void run() override 
      { 
            cout << "Method run;" << endl;
      }
      
      unique_ptr<Creator> createConcreteCreator()
      {
            return unique_ptr<Creator>(new ConcreteCreator<ConcreteProduct1>());
      }
};
```
{% endcode %}
{% endtab %}

{% tab title="Solution" %}
{% hint style="info" %}
Класс Solution предоставляет метод для регистрации в данном случае Creator'ов для карты (map), состоящий из пар (pair): ключ + значение.
{% endhint %}

{% code fullWidth="true" %}
```cpp
class Solution
{
public:
      typedef unique_ptr<Creator> (*CreateCreator)();
 
      bool registration(size_t id, CreateCreator createfun)
      {
            return callbacks.insert(CallBackMap::value_type(id, createfun)).second;
      }
      
      bool check(size_t id) 
      { 
            return callbacks.erase(id) == 1; 
      }

      unique_ptr<Creator> create(size_t id)
      {
            CallBackMap::const_iterator it = callbacks.find(id);
 
            if (it == callbacks.end())
            {
//                throw IdError();
            }
            return unique_ptr<Creator>((it->second)());
      }
private:
      using CallBackMap = map<size_t, CreateCreator>;
      
      CallBackMap callbacks;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream> 
# include <memory> 
# include <map> 

using namespace std; 

int main()
{
      Solution solution;
      solution.registration(1, createConcreteCreator);
      shared_ptr<Creator> cr(solution.create(1));
      shared_ptr<Product> ptr = cr->createProduct();
      ptr->run();
}
```
{% endcode %}

## Возможные реализации для решения конкретных задач

## Фабричный метод с созданием нового объекта

В рамках данной реализации появляется возможность избавиться от необходимости создания конкретного объекта в коде.

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
    virtual ~Product() = 0;
    virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd1 : public Product
{
public:
    virtual ~ConcreteProd1() override 
    { 
        cout << "Destructor;" << endl; 
    }
    
    virtual void run() override 
    { 
        cout << "Method run;" << endl; 
    }
};

```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
```cpp
class Creator
{
public:
    virtual unique_ptr<Product> createProduct() = 0;
};
```
{% endtab %}

{% tab title="ConcreteCreator" %}
{% code fullWidth="true" %}
```cpp
template <typename Tprod>
class ConcreteCreator : public Creator
{
public:
    virtual unique_ptr<Product> createProduct() override
    {
        return unique_ptr<Product>(new Tprod());
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
    shared_ptr<Creator> cr(new ConCreator<ConProd1>()); 
    shared_ptr<Product> ptr = cr->createProduct();

    ptr->run();
}
```
{% endcode %}

## Фабричный метод с шаблонным базовым классом

BaseCreator - шаблонный базовый класс, определяющий метод create(), который должен быть реализован в подклассах. Creator - класс, наследующийся от BaseCreator, и реализующий метод create(). Он создает экземпляр заданного типа продукта с помощью переданных аргументов.

При использовании данного способа реализации можно создавать экземпляры различных продуктов с помощью общего интерфейса создателя и передавать аргументы для создания конкретных продуктов. Это позволяет более гибко управлять процессом создания объектов и добавлять новые типы продуктов, не изменяя кода создателя.

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
	virtual ~Product() = default;
	virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd1 : public Product
{
private:
	int count;
	double price;
public:
	ConcreteProd1(int c, double p) : count(c), price(p)
	{
		cout << "Calling the ConcreteProd1 constructor;" << endl;
	}
	
	virtual ~ConProd1() override 
	{ 
		cout << "Calling the ConcreteProd1 destructor;" << endl; 
	}
	
	virtual void run() override 
	{ 
		cout << "Calling the run method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="BaseCreator" %}
```cpp
template <typename Tbase, typename ...Args>
class BaseCreator
{
public:
	virtual ~BaseCreator() = default;
	virtual unique_ptr<Tbase> create(Args ...args) = 0;
};
```
{% endtab %}

{% tab title="Creator" %}
{% code fullWidth="true" %}
```cpp
template <typename Tbase, typename Tprod, typename ...Args>
class Creator : public BaseCreator<Tbase, Args...>
{
public:
	virtual unique_ptr<Tbase> create(Args ...args) override
	{
		return unique_ptr<Tbase>(new Tprod(args...));
	}
};

using TbaseCreator = BaseCreator<Product, int, double>;
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
	void use(shared_ptr<TbaseCreator>& cr)
	{
		shared_ptr<Product> ptr = cr->create(1, 100.);

		ptr->run();
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
	shared_ptr<TbaseCreator> cr(new Creator<Product, ConcreteProd1, int, double>());

	unique_ptr<User> us = make_unique<User>();

	us->use(cr);
}
```
{% endcode %}

## Фабричный метод без повторного создания объектов

В классе Creator определено приватное поле product, которое хранит созданный продукт. Если продукт уже создан, то возвращается указатель на него, иначе вызывается метод createProduct() для его создания.

Такая реализация позволяет один раз создав продукт, в дальнейшем только возвращать один и тот же объект продукта через метод getProduct().

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
	virtual ~Product() = default;
	virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd1 : public Product
{
public:
	ConcreteProd1() 
	{ 
		cout << "Calling the ConcreteProd1 constructor;" << endl; 
	}
	
	virtual ~ConcreteProd1() override 
	{ 
		cout << "Calling the ConcreteProd1 destructor;" << endl; 
	}
	
	virtual void run() override 
	{ 
		cout << "Calling the run method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
```cpp
class Creator
{
public:
	shared_ptr<Product> getProduct()
	{
		if (!product)
		{
			product = createProduct();
		}

	return product;
}

protected:
	virtual shared_ptr<Product> createProduct() = 0;

private:
	shared_ptr<Product> product;
};
```
{% endtab %}

{% tab title="ConcreteCreator" %}
{% code fullWidth="true" %}
```cpp
template <typename Tprod>
class ConcreteCreator : public Creator
{
protected:
	virtual shared_ptr<Product> createProduct() override
	{
		return shared_ptr<Product>(new Tprod());
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
	shared_ptr<Creator> cr(new  ConcreteCreator<ConcreteProd1>());
	shared_ptr<Product> ptr1 = cr->getProduct();
	shared_ptr<Product> ptr2 = cr->getProduct();

	cout << ptr1.use_count() << endl;
	ptr1->run();
}
```
{% endcode %}

## Фабричный метод с разделением обязанностей

Класс Solution выполняет роль посредника между клиентским кодом и классами создателей продуктов. Он отвечает за регистрацию методов создания объектов для каждого типа продукта и предоставляет методы для создания объектов по их идентификаторам.

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
    virtual ~Product() = 0;
    virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd1 : public Product
{
public:
	ConcreteProd1() 
	{ 
		cout << "Calling the ConcreteProd1 constructor;" << endl; 
	}
	
	virtual ~ConProd1() override 
	{ 
		cout << "Calling the ConcreteProd1 destructor;" << endl; 
	}
	
	virtual void run() override 
	{ 
		cout << "Calling the run method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd2" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd2 : public Product
{
public:
	ConcreteProd2() 
	{ 
		cout << "Calling the ConcreteProd2 constructor;" << endl; 
	}
	
	virtual ~ConcreteProd2() override 
	{ 
		cout << "Calling the ConcreteProd2 destructor;" << endl; 
	}
	
	virtual void run() override 
	{ 
		cout << "Calling the run method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
```cpp
class Creator
{
public:
	template <typename Tprod>
	static unique_ptr<Creator> createConcreteCreator()
	{
		return unique_ptr<Creator>(new ConcreteCreator<Tprod>());
	}

};
```
{% endtab %}

{% tab title="Solution" %}
{% code fullWidth="true" %}
```cpp
class Solution
{
	using CreateCreator = unique_ptr<Creator>(*)();
	using CallBackMap = map<size_t, CreateCreator>;

public:
	Solution() = default;
	
	Solution(initializer_list<pair<size_t, CreateCreator>> list)
	{
		for (auto elem : list)
			this->registration(elem.first, elem.second);
	}

	bool registration(size_t id, CreateCreator createfun)
	{
		return callbacks.insert(CallBackMap::value_type(id, createfun)).second;
	}
	
	bool check(size_t id) 
	{ 
		return callbacks.erase(id) == 1; 
	}

	unique_ptr<Creator> create(size_t id)
	{
		CallBackMap::const_iterator it = callbacks.find(id);
	
		if (it == callbacks.end())
		{
//			throw IdError();
		}

		return unique_ptr<Creator>((it->second)());
	}

private:
	CallBackMap callbacks;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <initializer_list>
# include <memory>
# include <map>

using namespace std;

int main()
{
	shared_ptr<Solution> solution(new Solution({ {1, &Creator::createConcreteCreator<ConcreteProd1>} }));

	solution->registration(2, &Creator::createConcreteCreator<ConcreteProd2>);

	shared_ptr<Creator> cr(solution->create(2));
	shared_ptr<Product> ptr = cr->createProduct();

	ptr->run();
}
```
{% endcode %}

## **Шаблонный фабричный метод и подмена с перекомпиляцией**

В рамках такого метода реализации, вместо того чтобы определять конкретные классы создателей для каждого типа продукта, определяется класс Creator с шаблонным параметром Tprod. Этот параметр типа указывает на тип создаваемого продукта.

Появляется возможность подмены с перекомпиляцией – это операция, которая позволяет заменить одну реализацию класса на другую, и при этом не требуется перекомпилировать весь код, который использует этот класс.

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
    virtual ~Product() = 0;
    virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd1 : public Product
{
public:
	ConcreteProd1() 
	{ 
		cout << "Calling the ConcreteProd1 constructor;" << endl; 
	}
	
	virtual ~ConProd1() override 
	{ 
		cout << "Calling the ConcreteProd1 destructor;" << endl; 
	}
	
	virtual void run() override 
	{ 
		cout << "Calling the run method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
```cpp
template <typename Tprod>
class Creator
{
public:
	unique_ptr<Product> createProduct()
	{
		return unique_ptr<Product>(new Tprod());
	}
};
```
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
	template<typename Tprod>
	void use(shared_ptr<Creator<Tprod>> cr);
	{
		shared_ptr<Product> ptr = cr->createProduct();

		ptr->run();
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
	shared_ptr<Creator<ConProd1>> cr(new Creator<ConProd1>());

	unique_ptr<User> us = make_unique<User>();

	us->use(cr);
}
```
{% endcode %}

## Фабричный метод и статический полиморфизм

{% hint style="info" %}
Статический полиморфизм (compile-time polymorphism) - это механизм, который позволяет вызывать различные функции или методы с одним и тем же именем, но с разными параметрами или типами данных. Это достигается с помощью перегрузки функций или методов. Компилятор статически выбирает соответствующую функцию или метод на основе типов параметров, указанных при вызове.
{% endhint %}

В рамках такого метода реализации, определи класс Work, который предоставляет статический метод create(), который принимает объект создателя и вызывает его метод create() для создания продукта. Он использует статический полиморфизм для вызова правильного метода на основе типа объекта создателя.

Такая реализация фабричного метода с использованием статического полиморфизма позволяет гибко создавать объекты различных типов продуктов и создателей, используя общий интерфейс и не изменяя существующий код.

{% tabs %}
{% tab title="Product" %}
{% code fullWidth="true" %}
```cpp
class Product
{
public:
    virtual ~Product() = default;
    virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProd1" %}
{% code fullWidth="true" %}
```cpp
class ConcreteProd1 : public Product
{
public:
	ConcreteProd1() 
	{ 
		cout << "Calling the ConcreteProd1 constructor;" << endl; 
	}
	
	virtual ~ConProd1() override 
	{ 
		cout << "Calling the ConcreteProd1 destructor;" << endl; 
	}
	
	virtual void run() override 
	{ 
		cout << "Calling the run method;" << endl; 
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
```cpp
template <typename Tcrt>
class Creator
{
public:
	auto create() const
	{
		return static_cast<const Tcrt*>(this)->create_impl();
	}
};
```
{% endtab %}

{% tab title="ProductCreator" %}
{% code fullWidth="true" %}
```cpp
template <typename Tprod>
class ProductCreator : public Creator<ProductCreator<Tprod>>
{
public:
	unique_ptr<Product> create_impl() const
	{
		return unique_ptr<Product>(new Tprod());
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Work" %}
{% code fullWidth="true" %}
```cpp
class Work
{
public:
	template <typename Type>
	static auto create(const Type& crt)
	{
		return crt.create();
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

void main()
{
	Creator<ProductCreator<ConcreteProd1>> cr;

	auto product = Work::create(cr);

	product->run();
}
```
{% endcode %}
