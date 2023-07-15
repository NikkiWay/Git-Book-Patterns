---
description: Memento
---

# Опекун

## Назначение

Паттерн **опекун** реализует идею вынесения логики сохранения состояний в отдельный объект, ответственный за сохранение состояний объекта и возвращение его в предыдущее состояние.

<mark style="color:green;">Не нарушая инкапсуляции</mark>, фиксирует и выносит за пределы объекта его внутреннее состояние так, чтобы позднее можно было восстановить в нем объект.

## Решаемы задачи

* возможность сохранить мгновенный снимок состояния объекта (или его части), чтобы впоследствии объект можно было восстановить в том же состоянии
* используется в случае, если прямое получение состояния объекта раскрывает детали реализации и нарушает инкапсуляцию объекта

## Общая реализация на С++

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <list>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="Caretaker" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Memento;

class Caretaker
{
public:
    unique_ptr<Memento> getMemento();
    void setMemento(unique_ptr<Memento> memento);
private:
    list<unique_ptr<Memento>> mementos;
};
```
{% endcode %}
{% endtab %}

{% tab title="Originator" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Originator
{
public:
    Originator(int s) : state(s) {}
    const int getState() const 
    { 
        return state; 
    }
    void setState(int s) 
    { 
        state = s; 
    }
    std::unique_ptr<Memento> createMemento() 
    { 
        return make_unique<Memento>(*this); 
    }
    void restoreMemento(std::unique_ptr<Memento> memento);
private:
    int state;
};
```
{% endcode %}
{% endtab %}

{% tab title="Memento" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Memento
{
    friend class Originator;
public:
    Memento(Originator o) : originator(o) {}
private:
    void setOriginator(Originator o) 
    { 
        originator = o; 
    }
    Originator getOriginator() 
    { 
        return originator; 
    }
private:
    Originator originator;
};
```
{% endcode %}
{% endtab %}

{% tab title="Methods Caretaker" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# pragma region Methods Caretaker
void Caretaker::setMemento(unique_ptr<Memento> memento)
{
    mementos.push_back(move(memento));
}
unique_ptr<Memento> Caretaker::getMemento() 
{
    unique_ptr<Memento> last = move(mementos.back());
    mementos.pop_back();
    return last;
}
# pragma endregion
```
{% endcode %}
{% endtab %}

{% tab title="Method Originator" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# pragma region Method Originator
void Originator::restoreMemento(std::unique_ptr<Memento> memento)
{
    *this = memento->getOriginator();
}
# pragma endregion
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    auto originator = make_unique<Originator>(1);
    auto caretaker = make_unique<Caretaker>();

    cout << "State = " << originator->getState() << endl;
    caretaker->setMemento(originator->createMemento());

    originator->setState(2);
    cout << "State = " << originator->getState() << endl;
    caretaker->setMemento(originator->createMemento());
    originator->setState(3);
    cout << "State = " << originator->getState() << endl;
    caretaker->setMemento(originator->createMemento());

    originator->restoreMemento(caretaker->getMemento());
    cout << "State = " << originator->getState() << endl;
    originator->restoreMemento(caretaker->getMemento());
    cout << "State = " << originator->getState() << std::endl;
    originator->restoreMemento(caretaker->getMemento());
    cout << "State = " << originator->getState() << std::endl;
}
```
{% endcode %}

## Преимущества

* отделение логики сохранения и восстановления состояния от логики самого объекта: позволяет снять с класса задачу о сохранении своих предыдущих состояний
* предоставление возможности откатиться к предыдущему состоянию объекта
* возможность предоставлять различные алгоритмы сохранения состояния, позволяя выбрать наиболее подходящий подход для конкретной ситуации

## Недостатки&#x20;

* затраты памяти на сохранение снимков состояний
* потребность в реализации механизма очистки и определения того, нужны снимки или нет
* увеличение сложности кода из-за потребности создания дополнительных классов и методов

## Связь с другими паттернами

* [Команда](command.md): команды помещают информацию о состоянии, необходимую для отмены выполненных действий, в хранители.
* [Итератор](iterator.md): хранитель может быть использован вместе с итератором для сохранения текущего состояния итератора в снимке. Это позволяет итератору восстановить свое состояние после некоторых операций или переходов.
