---
description: Adapter
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="includes" %}
{% code fullWidth="true" %}
```cpp
// подключаем нужные для работы программы библиотеки
# include <iostream>
# include <memory>

// подключаем пространство имен std
using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="BaseAdaptee" %}
{% code fullWidth="true" %}
```cpp
// класс BaseAdaptee является базовым описанием интерфейса, 
// который нуждается в адаптации
 
class BaseAdaptee
{
public:
    virtual ~BaseAdaptee() = default;
    virtual void specificRequest() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteAdaptee" %}
{% code fullWidth="true" %}
```cpp
// класс ConcreteAdaptee определяет существующий интерфейс, 
// который нуждается в адаптации
// (реализует адаптируемый интерфейс) 

class ConcreteAdaptee : public BaseAdaptee
{
public:
    virtual void specificRequest() override 
    { 
        cout << "Method ConcreteAdaptee;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Adapter" %}
{% code fullWidth="true" %}
```cpp
// Adapter - базовый абстрактный класс для адапетров

// Adapter адаптирует интерфейс Adaptee к зависящему от 
// предметной области интерфейсу, которым пользуется Client

// Client - сущность, которая вступает во взаимоотношения с объектами, 
// удовлетворяющими интерфейсу
 
class Adapter
{
public:
    virtual ~Adapter() = default;
    virtual void request() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteAdapter" %}
{% code fullWidth="true" %}
```cpp
// конкретная реализация экзмпляра BaseAdapter

// ConcreteAdapter адаптирует интерфейс BaseAdaptee к интерфейсу Adapter,
// чтобы клиентский код мог использовать адаптируемый класс через адаптер

class ConcreteAdapter : public Adapter
{
private:
    shared_ptr<BaseAdaptee>  adaptee;
public:
    ConcreteAdapter(shared_ptr<BaseAdaptee> ad) : adaptee(ad) {}
    void request() override;
};
```
{% endcode %}
{% endtab %}

{% tab title="Methods" %}
{% code fullWidth="true" %}
```cpp
# pragma region Methods
void ConcreteAdapter::request()
{
    cout << "Adapter: ";
    if (adaptee)
    {
        adaptee->specificRequest();
    }
    else
    {
        cout << "Empty!" << endl;
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    shared_ptr<BaseAdaptee> adaptee = make_shared<ConcreteAdaptee>();
    shared_ptr<Adapter> adapter = make_shared<ConcreteAdapter>(adaptee);

    adapter->request();
}
```
{% endcode %}

## Возможные реализации для решения конкретных задач

## Шаблонный адаптер

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <vector>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="class Interface" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Interface
{
public:
    virtual ~Interface() = default;
    virtual void request() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="class Adapter" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <typename Type>
class Adapter : public Interface
{
public:
    using MethodPtr = void (Type::*)();

    Adapter(shared_ptr<Type> o, MethodPtr m) : object(o), method(m) {}

    void request() override 
    { 
    ((*object).*method)(); 
    }

private:
    shared_ptr<Type> object;
    MethodPtr method;
};
```
{% endcode %}
{% endtab %}

{% tab title="class AdapteeA" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class AdapteeA
{
public:
    ~AdapteeA() 
    { 
        cout << "Destructor class AdapteeA;" << endl; 
    }
    void specRequestA() 
    { 
        cout << "Method AdapteeA::specRequestA;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="class AdapteeB" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class AdapteeB
{
public:
    ~AdapteeB() 
    { 
        cout << "Destructor class AdapteeB;" << endl; 
    }
    void specRequestB() 
    { 
        cout << "Method AdapteeB::specRequestB;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="initialize()" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
auto initialize()
{
    using InterPtr = shared_ptr<Interface>;
    
    vector<InterPtr> vec{
        make_shared<Adapter<AdapteeA>>(make_shared<AdapteeA>(), &AdapteeA::specRequestA),
        make_shared<Adapter<AdapteeB>>(make_shared<AdapteeB>(), &AdapteeB::specRequestB)
    };

    return vec;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    auto v = initialize();

    for (const auto& elem : v)
        elem->request();
}
```
{% endcode %}
