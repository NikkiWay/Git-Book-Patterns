---
description: Mediator
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

# Посредник

## Назначение

Паттерн Посредник (Mediator) – это поведенческий шаблон проектирования, который используется для упрощения взаимодействия между объектами путем вынесения логики их взаимодействия в отдельный класс. Этот класс выступает в роли посредника, который контролирует и координирует взаимодействие между объектами, вместо того чтобы позволять им обмениваться информацией напрямую. Используется, когда имеется большое количество объектов, каждый из которых должен взаимодействовать с другими объектами, и прямое взаимодействие между всеми парами объектов становится сложным и запутанным.

## Решаемые задачи

* Упрощение взаимодействия между объектами

Паттерн обеспечивает централизованное взаимодействие между объектами, устраняя необходимость прямого взаимодействия между каждой парой объектов.

* Снижение зависимости между объектами

Паттерн позволяет объектам взаимодействовать друг с другом через абстрактный интерфейс, не зависящий от конкретных классов. Это позволяет объектам меняться независимо друг от друга, не затрагивая другие части системы.

* Избавление от лишней логики взаимодействия в классах

Появляется возможность очистить классы от избыточной логики, связанной с их взаимодействием. Вместо того чтобы каждый класс самостоятельно управлял своими зависимостями и коммуникацией с другими классами, эта логика выносится в посредник.

## Общая реализация на языке C++

{% tabs %}
{% tab title="Mediator" %}
{% code fullWidth="true" %}
```cpp
class Mediator
{
protected:
    list<shared_ptr<Colleague>> colleagues;

public:
    virtual ~Mediator() = default;

    virtual bool send(const Colleague* coleague, shared_ptr<Message> msg) = 0;

    static bool add(shared_ptr<Mediator> mediator, initializer_list<shared_ptr<Colleague>> list)
    {
        if (!mediator || list.size() == 0) return false;
    
        for (auto elem : list)
        {
            mediator->colleagues.push_back(elem);
            elem->setMediator(mediator);
        }
        return true;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteMediator" %}
{% code fullWidth="true" %}
```cpp
class ConcreteMediator : public Mediator
{
public:
    virtual bool send(const Colleague* coleague, shared_ptr<Message> msg) override
    {
        bool flag = false;
        for (auto& elem : colleagues)
        {
            if (dynamic_cast<const ColleagueLeft*>(colleague) && dynamic_cast<ColleagueRight*>(elem.get()))
            {
                elem->receive(msg);
                flag = true;
            }
            else if (dynamic_cast<const ColleagueRight*>(colleague) && dynamic_cast<ColleagueLeft*>(elem.get()))
            {
                elem->receive(msg);
                flag = true;
            }
        }
        return flag;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Colleague" %}
{% code fullWidth="true" %}
```cpp
class Colleague
{
private:
    weak_ptr<Mediator> mediator;

public:
    virtual ~Colleague() = default;

    void setMediator(shared_ptr<Mediator> mdr) 
    { 
        mediator = mdr; 
    }

    virtual bool send(shared_ptr<Message> msg);
    
    virtual void receive(shared_ptr<Message> msg) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ColleagueLeft" %}
{% code fullWidth="true" %}
```cpp
class ColleagueLeft : public Colleague
{
public:
    virtual void receive(shared_ptr<Message> msg) override 
    { 
        cout << "Right - > Left;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="ColleagueRight" %}
{% code fullWidth="true" %}
```cpp
class ColleagueRight : public Colleague
{
public:
    virtual void receive(shared_ptr<Message> msg) override 
    { 
        cout << "Left - > Right;" << endl; 
    }
};

```
{% endcode %}
{% endtab %}

{% tab title="Message" %}
{% hint style="info" %}
Класс "Message" или "Request" используется для представления сообщений или запросов, которые передаются между объектами через посредника.

Этот класс обычно содержит информацию, необходимую для выполнения определенной операции или взаимодействия между объектами. Он может содержать данные, параметры, команды и другую информацию, которая требуется для передачи между объектами.
{% endhint %}

{% code fullWidth="true" %}
```cpp
class Message {}; 
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <list>
# include <vector>

using namespace std;

int main()
{
    shared_ptr<Mediator> mediator = make_shared<ConcreteMediator>();

    shared_ptr<Colleague> col1 = make_shared<ColleagueLeft>();
    shared_ptr<Colleague> col2 = make_shared<ColleagueRight>();
    shared_ptr<Colleague> col3 = make_shared<ColleagueLeft>();
    shared_ptr<Colleague> col4 = make_shared<ColleagueLeft>();

    Mediator::add(mediator, { col1, col2, col3, col4 });

    shared_ptr<Message> msg = make_shared<Message>();

    col1->send(msg);
    col2->send(msg);
}
```
{% endcode %}

## Преимущества

1. Устранение прямых зависимостей между объектами.
2. Уменьшение дублирования кода. Паттерн позволяет выделить поведение, которое должно быть общим для нескольких объектов, в отдельный класс посредника.
3. Расширяемость системы. Паттерн облегчает добавление новых объектов и изменение взаимодействия между ними. Новые объекты могут быть легко интегрированы в систему, а изменения взаимодействия между объектами могут быть внесены в посредника.

## Недостатки

1. Увеличение количества кода за счёт появления дополнительных иерархий классов.
2. Увеличение времени выполнения. Объекты отправляют сообщения посреднику, который затем передает их другим объектам. Этот дополнительный[ уровень косвенности](#user-content-fn-1)[^1] может привести к снижению производительности программы.

## Связь с другими паттернами

* Паттерн Посредник может быть реализован как [Одиночка](../creationals-patterns/singleton.md), чтобы гарантировать единственный экземпляр посредника в системе. Это обеспечивает централизованный контроль и управление взаимодействием между объектами.
* Паттерн Посредник может работать с паттерном [Стратегия](strategy.md) для динамического изменения поведения объектов. Посредник может выбирать различные стратегии, в зависимости от текущего состояния системы, и передавать их объектам для выполнения соответствующих действий.
* Паттерн посредник может использоваться вместе с паттерном [Цепочка обязанностей](chain-of-responsibility.md) для передачи запросов между объектами. Посредник может принимать запросы и передавать их по цепочке объектов, пока один из объектов не обработает запрос.

[^1]: Уровень косвенности означает количество промежуточных шагов, которые необходимо выполнить для достижения конечного результата.
