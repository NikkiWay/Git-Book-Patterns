---
description: Property
---

# Свойство

## Назначение

Паттерн свойство представляет собой объединение двух методов - получения и изменения значения, что позволяет обеспечить сохранение целостности доступа к данным объекта.

## Решаемые задачи

* инкапсуляция данных: паттерн Свойство помогает обеспечить инкапсуляцию данных, скрывая их реализацию и предоставляя контролируемый доступ к ним
* валидация данных: возможность добавления логики проверки данных на соответствие определенным правилам или условиям в методах, реализующих паттерн Свойство
* ленивая инициализация: паттерн Свойство может быть использован для реализации ленивой инициализации, при которой инициализация объекта или вычисление значения откладывается до момента его фактического использования

{% hint style="info" %}
Основная цель ленивой инициализации - избежать необязательных ресурсозатратных операций или вычислений, которые могут быть не нужны до момента использования объекта. Это позволяет повысить производительность и эффективность написанной программы.
{% endhint %}

## Общая реализация на С++

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
#include <iostream>
#include <memory>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="Property" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <typename Owner, typename Type>
class Property
{
    using Getter = Type(Owner::*)() const;
    using Setter = void (Owner::*)(const Type&);
private:
    Owner* owner;
    Getter methodGet;
    Setter methodSet;
public:
    Property() = default;
    Property(Owner* const owr, Getter getmethod, Setter setmethod) : owner(owr), methodGet(getmethod), methodSet(setmethod) {}
    void init(Owner* const owr, Getter getmethod, Setter setmethod)
    {
        owner = owr;
        methodGet = getmethod;
        methodSet = setmethod;
    }
    operator Type() // Getter
    { 
        return (owner->*methodGet)(); 
    }
    void operator=(const Type& data) // Setter
    { 
        (owner->*methodSet)(data); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Object" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Object
{
private:
    double value;
public:
    Object(double v) : value(v) 
    { 
        Value.init(this, &Object::getValue, &Object::setValue); 
    }
    double getValue() const 
    { 
        return value; 
    }
    void setValue(const double& v) 
    { 
        value = v; 
    }
    Property<Object, double> Value;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    Object obj(5.);
    cout << "value = " << obj.Value << endl;
    obj.Value = 10.;
    cout << "value = " << obj.Value << endl;
    unique_ptr<Object> ptr = make_unique<Object>(15.);
    cout << "value =" << ptr->Value << endl;
    obj = *ptr;
    obj.Value = ptr->Value;
}
```
{% endcode %}

## Преимущества

* реализация принципа инкапсуляции
* обеспечение контроля доступа к данным, которое позволяет избежать ошибок при работе с объектами
* возможность добавлять дополнительную логику при доступе к данным (например, проверку на корректность значений или логирования запроса доступа к данным)

## Недостатки

* приводит к увеличению количества типового кода, так как для каждого поля объекта требуется создание методов доступа

## Связь с другими паттернами

* [Одиночка](../../creationals-patterns/singleton.md): паттерн Свойство может быть частью реализации паттерна Одиночки. В этом случае свойство, представляющее экземпляр Singleton класса, может быть доступно через геттер и сеттер методы Property.
* [Декоратор](../../structural-patterns/dekorator.md): паттерн Свойство может использоваться вместе с паттерном Декоратор для добавления дополнительной функциональности к свойству объекта. Декоратор может обернуть объект Свойства и расширить его поведение без изменения самого объекта.
* [Заместитель](../../structural-patterns/proxy.md): паттерн Свойство может быть реализован в виде прокси-объекта, который обеспечивает контроль доступа к свойству. Заместитель может добавлять дополнительную логику перед доступом к свойству или ограничивать доступ к нему.
* [Адаптер](../../structural-patterns/adapter/): паттерн Свойство может быть использован в связке с паттерном Adapter для адаптации интерфейса свойства к требуемому интерфейсу.
