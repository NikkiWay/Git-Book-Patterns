---
description: Mediator
---

# Реализации на С++

## Общая реализация на языке С++

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

{% code lineNumbers="true" fullWidth="true" %}
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
