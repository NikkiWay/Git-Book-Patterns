---
description: Abstract factory
---

# Возможные реализации для решения конкретных задач на С++

## Абстрактная фабрика с использованием генерации иерархии классов

{% tabs %}
{% tab title="Type List" %}
{% code fullWidth="true" %}
```cpp
# pragma region Type List
class NullType {};

template <typename... Types>
struct TypeList;

template <>
struct TypeList<> {};

template <typename H>
struct TypeList<H>
{
	using Head = H;
	using Tail = NullType;
};

template <typename H, typename... Types>
struct TypeList<H, Types...>
{
	using Head = H;
	using Tail = TypeList<Types...>;
};

#pragma endregion
```
{% endcode %}
{% endtab %}

{% tab title="Reverse List" %}
{% code fullWidth="true" %}
```cpp
# pragma region Reverse List
template <typename TList, typename Type> struct Append;

template <typename Type>
struct Append<TypeList<>, Type>
{
	using Result = TypeList<Type>;
};

template <typename Head, typename Type>
struct Append<TypeList<Head>, Type>
{
	using Result = TypeList<Head, Type>;
};

template <typename Head, typename... Tail, typename Type>
struct Append<TypeList<Head, Tail...>, Type>
{
	using Result = TypeList<Head, Tail..., Type>;
};

template <typename TList> struct Reverse;

template <typename Type>
struct Reverse<TypeList<Type>>
{
	using Result = TypeList<Type>;
};

template <typename Head, typename Tail>
struct Reverse<TypeList<Head, Tail>>
{
	using Result = Append<TypeList<Tail>, Head>::Result;
};

template <typename Head, typename... Tail>
struct Reverse<TypeList<Head, Tail...>>
{
	using Result = Append<typename Reverse<TypeList<Tail...>>::Result, Head>::Result;
};

# pragma endregion
```
{% endcode %}
{% endtab %}

{% tab title="Scatter Hierarchy" %}
{% code fullWidth="true" %}
```cpp
# pragma region Scatter Hierarchy
template <typename TList, template <typename> typename Unit>
class ScatterHierarchy;

template <template <typename> typename Unit>
class ScatterHierarchy<TypeList<>, Unit> {};

template <template <typename> typename Unit, typename AtomicType>
class ScatterHierarchy<TypeList<AtomicType>, Unit> : public Unit<AtomicType>
{
public:
	using LeftBase = Unit<AtomicType>;
};

template <template <typename> typename Unit, typename Head, typename... Tail>
class ScatterHierarchy<TypeList<Head, Tail...>, Unit>
	: public ScatterHierarchy<TypeList<Head>, Unit>, public ScatterHierarchy<TypeList<Tail...>, Unit>
{
public:
	using TList = TypeList<Head, Tail...>;
	using LeftBase = ScatterHierarchy<TypeList<Head>, Unit>;
	using RightBase = ScatterHierarchy<TypeList<Tail...>, Unit>;
};

# pragma endregion
```
{% endcode %}
{% endtab %}

{% tab title="Linear Hierarchy" %}
{% code fullWidth="true" %}
```cpp
# pragma region Linear Hierarchy
struct EmptyType {};

template
<
	typename TList,
	template <typename AtomicType, typename Base> typename Unit,
	typename Root = EmptyType 
>
class LinearHierarchy;

template
<
	typename Root,
	template <typename, typename> typename Unit,
	typename Head, typename... Tail
>
class LinearHierarchy<TypeList<Head, Tail...>, Unit, Root>
	: public Unit< Head, LinearHierarchy<TypeList<Tail...>, Unit, Root> >
{};

template
<
	typename Root,
	template <typename, typename> typename Unit,
	typename Head
>
class LinearHierarchy<TypeList<Head>, Unit, Root>
	: public Unit<Head, Root>
{};

# pragma endregion
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
{% code fullWidth="true" %}
```cpp
template <typename Type>
concept Abstract = is_abstract_v<Type>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Type2Type" %}
{% code fullWidth="true" %}
```cpp
template < typename T>
class Type2Type
{
public:
	using type = T;
};
```
{% endcode %}
{% endtab %}

{% tab title="BaseCreator" %}
{% code fullWidth="true" %}
```cpp
template <Abstract Tbase>
class BaseCreator
{
public:
	virtual ~BaseCreator() = default;

	virtual unique_ptr<Tbase> create(Type2Type<Tbase>) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="AbstractFactory" %}
{% code fullWidth="true" %}
```cpp
template <template <Abstract> typename Unit, Abstract... Types>
class AbstractFactory : public ScatterHierarchy<TypeList<Types...>, Unit>
{
public:
	using ProductList = TypeList<Types...>;
	template <typename Type>
	unique_ptr<Type> create()
	{
		BaseCreator<Type>& unit = *this;
		return unit.create(Type2Type<Type>());
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
{% code fullWidth="true" %}
```cpp
template <NotAbstract ConcreteProduct, typename Base>
//requires is_base_of_v<typename Base::ProductList::Head, ConcreteProduct>
class Creator : public Base
{
private:
	using BaseProductList = typename Base::ProductList;

protected:
	using ProductList = typename BaseProductList::Tail;

public:
	using AbstractProduct = typename BaseProductList::Head;

	unique_ptr<AbstractProduct> create(Type2Type<AbstractProduct>) override
	{
		return make_unique<ConcreteProduct>();
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteFactory" %}
{% code fullWidth="true" %}
```cpp
template
<
	typename AbstractFact,
	template <typename, typename> typename Creator,
	NotAbstract... Types
>
class ConcreteFactory
	: public LinearHierarchy<typename Reverse<TypeList<Types...>>::Result, Creator, AbstractFact>
{
	using AbstractProductList = typename AbstractFact::ProductList;
	using ConcreteProductList = TypeList<Types...>;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

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


class QtGraphics : public BaseGraphics
{
public:
	QtGraphics() 
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

{% tab title="BasePen" %}
{% code fullWidth="true" %}
```cpp
class BasePen
{
public:	
    virtual ~BasePen() = 0;
};

BasePen::~BasePen() {}


class QtPen : public BasePen {};
```
{% endcode %}
{% endtab %}

{% tab title="BaseBrush" %}
{% code fullWidth="true" %}
```cpp
class BaseBrush
{
public:	
    virtual ~BaseBrush() = 0;
};

BaseBrush::~BaseBrush() {}


class QtBrush : public BaseBrush {};
```
{% endcode %}
{% endtab %}

{% tab title="using" %}
{% code fullWidth="true" %}
```cpp
using AbstractGrFactory = AbstractFactory<BaseCreator, BaseGraphics, BasePen, BaseBrush>;
using QtGrFactory = ConcreteFactory<AbstractGrFactory, Creator, QtGraphics, QtPen, QtBrush>;
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
	void use(shared_ptr<AbstractGrFactory>& cr)
	{
		auto ptr = cr->create<BaseGraphics>();
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
	shared_ptr<AbstractGrFactory> gr = make_shared<QtGrFactory>();

	unique_ptr<User> us = make_unique<User>();

	us->use(gr);
}
```
{% endcode %}
