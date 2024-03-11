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

# Возможные реализации для решения конкретных задач

## Фабричный метод с созданием нового объекта

В рамках данной реализации появляется возможность избавиться от необходимости создания конкретного объекта в коде.

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```cpp
class Car
{
public:
    virtual ~Car() = default;  
    virtual void run() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Audi" %}
{% code fullWidth="true" %}
```cpp
class Audi : public Car
{
public:
    Audi() { 
        cout << "Calling the Audi constructor;" << endl; 
    }
    
    ~Audi() override { 
        cout << "Calling the Audi destructor;" << endl; 
    }
    
    void run() override { 
        cout << "Calling the run method;" << endl;
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

{% tab title="CarMaker" %}
{% code fullWidth="true" %}
```cpp
class CarMaker
{
public:
    virtual ~CarMaker() = default;
    virtual unique_ptr<Car> create() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="AudiMaker" %}
{% code fullWidth="true" %}
```cpp
template <Derivative<Car> Tprod>
requires NotAbstract<Tprod>
class AudiMaker : public CarMaker
{
public:
	unique_ptr<Car> create() override
	{
		return make_unique<Tprod>();
	}
};
```
{% endcode %}
{% endtab %}

{% tab title="CarUser" %}
{% code fullWidth="true" %}
```cpp
class CarUser
{
public:
	void use(shared_ptr<CarMaker>& cr)
	{
		shared_ptr<Car> ptr = cr->create();
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
	shared_ptr<CarMaker> cr = make_shared<AudiMaker<Audi>>();
	CarUser{}.use(cr);
}
```
{% endcode %}

## Фабричный метод с шаблонным CarMaker

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

{% tab title="ConProd1" %}
{% code fullWidth="true" %}
```cpp
class ConProd1 : public Product
{
public:
    ConProd1() 
    { 
        cout << "Calling the ConProd1 constructor;" << endl; 
    }
    
    ~ConProd1() override 
    { 
        cout << "Calling the ConProd1 destructor;" << endl;
    }

    void run() override 
    { 
        cout << "Calling the run method;" << endl; 
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
template <Derivative<Product> Tprod>
requires NotAbstract<Tprod>
class Creator
{
public:
    unique_ptr<Product> create()
    {
        return make_unique<Tprod>();
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
    template<NotAbstract Tprod>
    requires Derivative<Tprod, Product>
    void use(shared_ptr<Creator<Tprod>> cr);
};
```
{% endcode %}
{% endtab %}

{% tab title="Method User" %}
{% code fullWidth="true" %}
```cpp
template<NotAbstract Tprod>
requires Derivative<Tprod, Product>
void User::use(shared_ptr<Creator<Tprod>> cr)
{
    shared_ptr<Product> ptr = cr->create();
    ptr->run();
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
    using CreatorProd1_t = Creator<ConProd1>;
    shared_ptr<CreatorProd1_t> cr = make_shared<CreatorProd1_t>();

    unique_ptr<User> us = make_unique<User>();

    us->use(cr);
}
```
{% endcode %}

## Фабричный метод без повторного создания объектов. Идиома NVI (Non-Virtual Interface).

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

{% tab title="ConProd1" %}
{% code fullWidth="true" %}
```cpp
class ConProd1 : public Product
{
public:
    ConProd1() 
    { 
        cout << "Calling the ConProd1 constructor;" << endl; 
    }
    ~ConProd1() override 
    { 
        cout << "Calling the ConProd1 destructor;" << endl; 
    }

    void run() override 
    { 
        cout << "Calling the run method;" << endl; 
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

{% tab title="Creator" %}
{% code fullWidth="true" %}
```cpp
class Creator
{
public:
    virtual ~Creator() = default;
    shared_ptr<Product> getProduct();

protected:
    virtual shared_ptr<Product> createProduct() = 0;

private:
    shared_ptr<Product> product{ nullptr };
};
```
{% endcode %}
{% endtab %}

{% tab title="ConCreator" %}
{% code fullWidth="true" %}
```cpp
template <Derivative<Product> Tprod>
requires NotAbstract<Tprod>
class ConCreator : public Creator
{
protected:
    shared_ptr<Product> createProduct() override
    {
        return  make_shared<Tprod>();
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Method Creator" %}
{% code fullWidth="true" %}
```cpp
# pragma region Method Creator
shared_ptr<Product> Creator::getProduct()
{
    if (!product)
    {
        product = createProduct();
    }

    return product;
}
# pragma endregion
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```cpp
class User
{
public:
    void use(shared_ptr<Creator>& cr)
    {
        shared_ptr<Product> ptr1 = cr->getProduct();
        shared_ptr<Product> ptr2 = cr->getProduct();
    
        cout << "use count = " << ptr1.use_count() << endl;
        ptr1->run();
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
    shared_ptr<Creator> cr = make_shared<ConCreator<ConProd1>>();
    unique_ptr<User> us = make_unique<User>();
    us->use(cr);
}
```
{% endcode %}

## Фабричный метод с шаблонным базовым классом CarMaker

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

{% tab title="ConProd1" %}
{% code fullWidth="true" %}
```cpp
class ConProd1 : public Product
{
private:
    int count;
    double price;
public:
    ConProd1(int c, double p) : count(c), price(p)
    {
        cout << "Calling the ConProd1 constructor;" << endl;
    }
    
    ~ConProd1() override 
    { 
        cout << "Calling the ConProd1 destructor;" << endl; 
    }

    void run() override 
    { 
        cout << "Count = " << count << "; Price = " << price << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="ConProd2" %}
{% code fullWidth="true" %}
```cpp
class ConProd2// : public Product
{
public:
    ConProd2(int c, double p)
    {
        cout << "Calling the ConProd2 constructor;" << endl;
    }
    
    virtual ~ConProd2() 
    { 
        cout << "Calling the ConProd2 destructor;" << endl; 
    }
    
    virtual void run() 
    { 
        cout << "Calling the run method ConProd2;" << endl; 
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

{% tab title="Variants of the concept Constructible" %}
{% code fullWidth="true" %}
```cpp
# pragma region Variants of the concept Constructible
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

# pragma endregion
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
    virtual unique_ptr<Tbase> create(Args&& ...args) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Creator" %}
{% code fullWidth="true" %}
```cpp
template <typename Tbase, typename Tprod, typename... Args>
requires NotAbstract<Tprod>&& Derivative<Tprod, Tbase>&& Constructible<Tprod, Args...>
class Creator : public BaseCreator<Tbase, Args...>
{
public:
    unique_ptr<Tbase> create(Args&& ...args) override
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
using BaseCreator_t = BaseCreator<Product, int, double>;

class User
{
public:
    void use(shared_ptr<BaseCreator_t>& cr)
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
    shared_ptr<BaseCreator_t> cr = make_shared<Creator<Product, ConProd1, int, double>>();
    unique_ptr<User> us = make_unique<User>();
    us->use(cr);
}
```
{% endcode %}

## Фабричный метод и статический полиморфизм

{% hint style="info" %}
Статический полиморфизм (compile-time polymorphism) - это механизм, который позволяет вызывать различные функции или методы с одним и тем же именем, но с разными параметрами или типами данных. Это достигается с помощью перегрузки функций или методов. Компилятор статически выбирает соответствующую функцию или метод на основе типов параметров, указанных при вызове.
{% endhint %}

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

{% tab title="ConProd1" %}
{% code fullWidth="true" %}
```cpp
class ConProd1 : public Product
{
public:
    ConProd1() 
    { 
        cout << "Calling the ConProd1 constructor;" << endl; 
    }
    
    ~ConProd1() override 
    { 
        cout << "Calling the ConProd1 destructor;" << endl; 
    }

    void run() override 
    { 
        cout << "Calling the run method;" << endl; 
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
```
{% endcode %}
{% endtab %}

{% tab title="ProductCreator" %}
{% code fullWidth="true" %}
```cpp
template <Derivative<Product> Tprod>
requires NotAbstract<Tprod>
class ProductCreator : public Creator<ProductCreator<Tprod>>
{
public:
    unique_ptr<Product> create_impl() const
    {
        return make_unique<Tprod>();
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
    template<Derivative<Product> Tprod>
    void use(Creator<ProductCreator<Tprod>>& cr) requires NotAbstract<Tprod>
    {
        auto product = cr.create();
        product->run();
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
    Creator<ProductCreator<ConProd1>> cr;
    User{}.use(cr);
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
```
{% endcode %}
{% endtab %}

{% tab title="Concept" %}
{% code fullWidth="true" %}
```cpp
template <typename Tder, typename Tbase = Command>
concept Derived = is_base_of_v<Tbase, Tder>;
```
{% endcode %}
{% endtab %}

{% tab title="CommandCreator" %}
{% code fullWidth="true" %}
```cpp
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
```
{% endcode %}
{% endtab %}

{% tab title="SimpleCommand" %}
{% code fullWidth="true" %}
```cpp
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
