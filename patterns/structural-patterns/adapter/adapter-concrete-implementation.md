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
