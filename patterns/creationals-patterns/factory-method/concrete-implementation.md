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

# Возможные реализации для решения конкретных задач на С++

## Фабричный метод с созданием нового объекта

В рамках данной реализации появляется возможность избавиться от необходимости создания конкретного объекта в коде.

Идея заключается в создании иерархии классов, объекты которых будут отвечать за создание наших объектов, то есть базовый класс будет иметь метод create виртуальный, а последующие создатели будут реализовывать этот метод.

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;  
    virtual void drive() = 0;
};

class Sedan : public Car
{
public:
    Sedan() 
    { 
        cout << "Sedan constructor called" << endl; 
    }
    
    ~Sedan() override 
    { 
        cout << "Sedan destructor called" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving sedan" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
```cpp
template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>; 
```
{% endtab %}

{% tab title="CarCreator" %}
{% code fullWidth="true" %}
```cpp
class CarCreator
{
public:
    virtual ~CarCreator() = default;
    virtual unique_ptr<Car> create() const = 0;
};

template <Derivative<Car> TCar>
requires NotAbstract<TCar>
class ConcreteCarCreator : public CarCreator
{
public:
    unique_ptr<Car> create() const override
    {
        return make_unique<TCar>();
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
    void use(const shared_ptr<CarCreator>& creator)
    {
        shared_ptr<Car> car = creator->create();
        car->drive();
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
	shared_ptr<CarCreator> creator = make_shared<ConcreteCarCreator<Sedan>>();
	
	User{}.use(creator);
}
```
{% endcode %}

## Фабричный метод без повторного создания объектов. Идиома NVI (Non-Virtual Interface).

Задан базовый абстрактный класс CarCreator с public методом getCar(), protected виртуальным методом createCar() и приватным полем car.

Метод getCar() создает Car, если он уже не создан и возвращает поле car.

Неабстрактный класс ConcreteCarCreator наследуется от CarCreator и определяет метод createCar(), для создания конкретного объекта.

Фабричный метод с использованием идиомы NVI применяется в случаях множественного использования тяжелого (тяжело создаваемого) объекта, при отсутствии необходимости в нескольких объектов данного типа.

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;
    virtual void drive() = 0;
};


class Sedan : public Car
{
public:
    Sedan() 
    { 
        cout << "Sedan constructor called" << endl; 
    }
    
    ~Sedan() override 
    { 
        cout << "Sedan destructor called" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving sedan" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
{% code fullWidth="true" %}
```cpp
template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;
```
{% endcode %}
{% endtab %}

{% tab title="CarCreator" %}
{% code fullWidth="true" %}
```cpp
class CarCreator
{
public:
    virtual ~CarCreator() = default;

    shared_ptr<Car> getCar();

protected:
    virtual shared_ptr<Car> create() = 0;

private:
    shared_ptr<Car> car{ nullptr };
};


template <Derivative<Car> TCar>
requires NotAbstract<TCar>
class ConcreteCarCreator : public CarCreator
{
protected:
    shared_ptr<Car> create() override
    {
        return make_shared<TCar>();
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Method CarCreator" %}
{% code fullWidth="true" %}
```cpp
shared_ptr<Car> CarCreator::getCar()
{
    if (!car)
    {
        car = create();
    }

    return car;
}
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
    void use(shared_ptr<CarCreator>& creator)
    {
        shared_ptr<Car> car1 = creator->getCar();
        shared_ptr<Car> car2 = creator->getCar();

        cout << "use count = " << car1.use_count() << endl;
        car1->drive();
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
    shared_ptr<CarCreator> creator = make_shared<ConcreteCarCreator<Sedan>>();

    unique_ptr<User> user = make_unique<User>();
    user->use(creator);

    return 0;
}
```
{% endcode %}

## Фабричный метод с шаблонным CarCreator

В данном случае задан один единственный CarCreator, он является шаблонным, что позволяет избавиться от необходимости создания конкретных креаторов ([Creator](./#uml-diagramma)) для каждого типа объекта.

При данном подходе решение о создании объекта переносится на стадию компиляции и от этого выполнение происходит быстрее, но при добавлении новых типов объектов придется перекомпилировать всю кодовую базу, содержащую данный креатор.

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;
    virtual void drive() = 0;
};


class Sedan : public Car
{
public:
    Sedan() 
    { 
        cout << "Sedan constructor called" << endl; 
    }
    
    ~Sedan() override 
    { 
        cout << "Sedan destructor called" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving sedan" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
```cpp
template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;
```
{% endtab %}

{% tab title="CarCreator" %}
{% code fullWidth="true" %}
```cpp
template <Derivative<Car> TCar>
requires NotAbstract<TCar>
class CarCreator
{
public:
    unique_ptr<Car> create()
    {
        return make_unique<TCar>();
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
    template<NotAbstract TCar>
    requires Derivative<TCar, Car>
    void use(shared_ptr<CarCreator<TCar>> creator);
};


template<NotAbstract TCar>
requires Derivative<TCar, Car>
void User::use(shared_ptr<CarCreator<TCar>> creator)
{
    shared_ptr<Car> car = creator->create();
    car->drive();
}
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
	using SedanCreator_t = CarCreator<Sedan>;
  	shared_ptr<SedanCreator_t> sedanCreator = make_shared<SedanCreator_t>();

	unique_ptr<User> user = make_unique<User>();
	
	user->use(sedanCreator);
}
```
{% endcode %}

## Фабричный метод со статическим шаблонным методом create

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;
    virtual void drive() = 0;
};


class Sedan : public Car
{
private:
	int seats;
	double weight;
	
public:
    Sedan(int s, double w) : seats(s), weight(w)
    { 
        cout << "Sedan constructor called" << endl; 
    }
    
    ~Sedan() override 
    { 
        cout << "Sedan destructor called" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving sedan" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
{% code fullWidth="true" %}
```cpp
template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;

template<typename Type, typename... Args>
concept Constructible = is_constructible_v<Type, Args...>;
```
{% endcode %}
{% endtab %}

{% tab title="CarCreator" %}
{% code fullWidth="true" %}
```cpp
class CarCreator
{
public:
	template <Derivative<Car> TCar, typename... Args>
	requires NotAbstract<TCar> && Constructible<TCar, Args...>
	static unique_ptr<Car> create(Args&& ...args) 
	{
		return make_unique<TCar>(forward<Args>(args)...);
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

    template<NotAbstract TCar, typename... Args>
    void use(Args&& ...args) requires Derivative<TCar, Car>
    {
        shared_ptr<Car> ptr = CarCreator::create<TCar>(forward<Args>(args)...);
        
        ptr->drive();
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
# include <concepts>

using namespace std;

int main()
{
	unique_ptr<User> us = make_unique<User>();

	us->use<Sedan>(1, 100.);
}
```
{% endcode %}

## Фабричный метод с шаблонным базовым классом Creator

BaseCreator - абстрактный базовый класс создаваемых объектов продуктов (в нашем случае Car), от которого наследуется конкретный шаблонный Creator. Параметр шаблона Tbase - это абстрактный базовый класс иерархии классов продуктов.

Идея в создании базового абстрактного шаблонного класса BaseCreator, принимающего абстрактный класс, заключается в возможности не создавать для каждой иерархии продуктов свой базовый класс Creator.

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;
    virtual void drive() = 0;
};


class Sedan : public Car
{
private:
	int seats;
	double weight;
	
public:
    Sedan(int s, double w) : seats(s), weight(w)
    { 
        cout << "Sedan constructor called" << endl; 
    }
    
    ~Sedan() override 
    { 
        cout << "Sedan destructor called" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving sedan" << endl; 
    }
};


class SUV : public Car 
{
public:
    SUV(int s, double w) 
    {
        cout << "Calling the SUV constructor;" << endl;
    }
    
    ~SUV() override 
    { 
        cout << "Calling the SUV destructor;" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving SUV;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
```cpp
template <typename Type>
concept Abstract = is_abstract_v<Type>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;

template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;
```
{% endtab %}

{% tab title="Constructible" %}
{% code fullWidth="true" %}
```cpp
# define V_1

# ifdef V_1
template<typename Type, typename... Args>
concept Constructible = requires(Args... args)
{
    Type{ args... };
};

# elif defined(V_2)
template<typename Type, typename... Args>
concept Constructible = requires
{
    Type{ declval<Args>()... };
};

# elif defined(V_3)
template<typename Type, typename... Args>
concept Constructible = is_constructible_v<Type, Args...>;

# endif 
```
{% endcode %}
{% endtab %}

{% tab title="BaseCreator" %}
{% code fullWidth="true" %}
```cpp
template <Abstract Tbase, typename... Args>
class BaseCreator
{
public:
    virtual ~BaseCreator() = default;
    virtual unique_ptr<Tbase> create(Args&& ...args) const = 0;
};


template <typename Tbase, typename Tprod, typename... Args>
requires NotAbstract<Tprod>&& Derivative<Tprod, Tbase>&& Constructible<Tprod, Args...>
class Creator : public BaseCreator<Tbase, Args...>
{
public:
    unique_ptr<Tbase> create(Args&& ...args) const override
    {
        return make_unique<Tprod>(forward<Args>(args)...);
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
using BaseCarCreator_t = BaseCreator<Car, int, double>;

class User
{
public:
    void use(const shared_ptr<BaseCarCreator_t>& creator)
    {
        shared_ptr<Car> car = creator->create(1, 100.);
        car->drive();
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

using SedanCreator_t = Creator<Car, Sedan, int, double>;

int main()
{
    shared_ptr<BaseCarCreator_t> creator = make_shared<SedanCreator_t>();
    unique_ptr<User> user = make_unique<User>();
    user->use(creator);
}
```
{% endcode %}

## Фабричный метод и "Cтатический полиморфизм" (CRTP)

{% hint style="info" %}
Статический полиморфизм (compile-time polymorphism) - это механизм, который позволяет вызывать различные функции или методы с одним и тем же именем, но с разными параметрами или типами данных. Это достигается с помощью перегрузки функций или методов. Компилятор статически выбирает соответствующую функцию или метод на основе типов параметров, указанных при вызове.
{% endhint %}

Задан базовый шаблонный класс Creator, который имеет метод create(). В методе происходит приведение объекта класса к типу, задаваемым параметром шаблона Tcrt, и вызывается метод по созданию объекта (create\_impl). Любой креатор, который будет использован для создания базового Creator должен иметь данную функцию по созданию конкретного объекта.

Подход дает большую гибкость для расширения типов креаторов, не меняя написанного кода. Также содержит все плюсы и минусы шаблонных классов.

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;
    virtual void drive() = 0;
};


class Sedan : public Car
{
public:
    Sedan() 
    { 
        cout << "Sedan constructor called" << endl; 
    }
    
    ~Sedan() override 
    { 
        cout << "Sedan destructor called" << endl; 
    }

    void drive() override 
    { 
        cout << "Driving sedan" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Concepts" %}
```cpp
template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;
```
{% endtab %}

{% tab title="Creator" %}
{% code fullWidth="true" %}
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


template <Derivative<Car> TCar>
requires NotAbstract<TCar>
class CarCreator : public Creator<CarCreator<TCar>>
{
public:
    unique_ptr<Car> create_impl() const
    {
        return make_unique<TCar>();
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
    template<Derivative<Car> TCar>
    void use(const Creator<CarCreator<TCar>>& creator) requires NotAbstract<TCar>
    {
        auto car = creator.create();
        car->drive();
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
    Creator<CarCreator<Sedan>> creator;
    User{}.use(creator);
}
```
{% endcode %}

## Фабричный метод с использованием идиомы «стирание типа» (Type erasure)

{% tabs %}
{% tab title="Concepts" %}
{% code fullWidth="true" %}
```cpp
template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <typename Type>
concept NotAbstract = !is_abstract_v<Type>;
```
{% endcode %}
{% endtab %}

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

{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
	virtual ~Car() = default;
	virtual void drive() = 0;
};

class Sedan : public Car
{
public:
	Sedan()
	{
		cout << "Sedan constructor called" << endl;
	}

	~Sedan() override
	{
		cout << "Sedan destructor called" << endl;
	}

	void drive() override
	{
		cout << "Driving sedan" << endl;
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="CarCreator" %}
{% code fullWidth="true" %}
```cpp
class CarCreator
{
public:
	template <Derivative<Car> TCar>
	requires NotAbstract<TCar>
	CarCreator(Type2Type<TCar>)
	{
		ptrmethod = &CarCreator::doCreate<TCar>;		
	}

	unique_ptr<Car> create()
	{
		return (this->*ptrmethod)();
	}

private:
	template <typename TCar>
	unique_ptr<Car> doCreate()
	{
		return make_unique<TCar>();
	}

	using Prtmethod_t = unique_ptr<Car>(CarCreator::*)();

	Prtmethod_t ptrmethod;
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
	void use(shared_ptr<CarCreator>& creator)
	{
		shared_ptr<Car> ptr = creator->create();

		ptr->drive();
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
# include <concepts>

using namespace std;

int main()
{
	shared_ptr<CarCreator> creator = make_shared<CarCreator>(Type2Type<Sedan>());

	unique_ptr<User> user = make_unique<User>();

	user->use(creator);
}
```
{% endcode %}

## Использование паттерна «фабричный метод» для паттерна Command

{% tabs %}
{% tab title="BaseCommandCreator" %}
{% code fullWidth="true" %}
```cpp
class BaseCommandCreator
{
public:
    ~BaseCommandCreator() = default;

    virtual shared_ptr<Command> create_command() const = 0;
};


template <typename Tder, typename Tbase = Command>
concept Derived = is_base_of_v<Tbase, Tder>;


template <Derived<Command> Type>
class CommandCreator : public BaseCommandCreator
{
public:
    template <typename... Args>
    CommandCreator(Args ...args)
    {
        create_func = [args...]() { return make_shared<Type>(args...); };
    }
    ~CommandCreator() = default;

    shared_ptr<Command> create_command() const override
    {
        return create_func();
    }
private:
    function<shared_ptr<Command>()> create_func;
};
```
{% endcode %}
{% endtab %}

{% tab title="MFP" %}
{% code fullWidth="true" %}
```cpp
# pragma region Member_Function_Pointer
namespace MFP
{
    template <typename T>
    struct is_member_function_pointer_helper : std::false_type {};

    template <typename T, typename U>
    struct is_member_function_pointer_helper<T U::*> : std::is_function<T> {};

    template <typename T>
    struct is_member_function_pointer
        : is_member_function_pointer_helper< typename std::remove_cv<T>::type > {};

    template <typename T>
    inline constexpr bool is_member_function_pointer_v = is_member_function_pointer<T>::value;
}
# pragma endregion
```
{% endcode %}
{% endtab %}

{% tab title="Command" %}
{% code fullWidth="true" %}
```cpp
class Command
{
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
};


template <typename Reseiver>
requires is_class_v<Reseiver> && MFP::is_member_function_pointer_v<void (Reseiver::*)()>
class SimpleCommand : public Command
{
    using Action = void(Reseiver::*)();
    using Pair = pair<shared_ptr<Reseiver>, Action>;
private:
    Pair call;

public:
    SimpleCommand(shared_ptr<Reseiver> r, Action a) : call(r, a) {}
    void execute() override 
    { 
        ((*call.first).*call.second)(); 
    }
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
    void operation() 
    { 
        cout << "Run method;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Invoker" %}
{% code fullWidth="true" %}
```cpp
class Invoker
{
public:
    void run(shared_ptr<Command> com) 
    { 
        com->execute(); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="SimpleComCreator" %}
{% code fullWidth="true" %}
```cpp
template <typename Type>
using SimpleComCreator = CommandCreator<SimpleCommand<Type>>;
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <functional>

using namespace std;

int main()
{
    shared_ptr<Invoker> inv = make_shared<Invoker>();
    shared_ptr<Object> obj = make_shared<Object>();

    shared_ptr<BaseCommandCreator> cr = make_shared<SimpleComCreator<Object>>(obj, &Object::operation);

    shared_ptr<Command> com = cr->create_command();

    inv->run(com);
}
```
{% endcode %}
