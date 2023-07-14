---
description: Visitor
---

# Посетитель

## Назначение

Поведенческий паттерн проектирования посетитель предоставляет способ разделить сущность объекта и способ работы с этой сущностью. Он позволяет определить новую операцию, не изменяя классы объектов, с которыми работает.

Посетитель позволяет выполнить нужные действия в зависимости от типов объектов.

Паттерн посетитель основывается на идее объединения нескольких деревьев стратегий в один класс с множеством методов для каждой сущности.

## Решаемые задачи

* возможность выполнять операции (зависящие от конкретных классов) над объектами многих классов с различными интерфейсами
* объединение родственных операций в один класс

{% hint style="info" %}
Если структура объектов является общей для нескольких приложений, то паттерн посетитель позволит в каждое приложение включить только относящиеся к нему операции.
{% endhint %}

* определение новых операций без изменения классов
* разделение ответственности между классами: вынесение операций, находящихся в самих классах, в отдельные классы-посетители

## Общая реализация на C++

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

{% tab title="Visitor" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Circle;
class Rectangle;

class Visitor
{
public:
    virtual ~Visitor() = default;
    virtual void visit(Circle& ref) = 0;
    virtual void visit(Rectangle& ref) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Shape" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Shape
{
public:
    virtual ~Shape() = default;
    virtual void accept(shared_ptr<Visitor> visitor) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Circle" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Circle : public Shape
{
public:
    void accept(shared_ptr<Visitor> visitor) override 
    { 
        visitor->visit(*this); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Rectangle" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Rectangle : public Shape
{
public:
    void accept(shared_ptr<Visitor> visitor)  override 
    { 
        visitor->visit(*this); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteVisitor" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class ConcreteVisitor : public Visitor
{
public:
    void visit(Circle& ref) override 
    { 
        cout << "Circle;" << endl; 
    }
    void visit(Rectangle& ref) override 
    { 
        cout << "Rectangle;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Figure" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Figure : public Shape
{
    using Shapes = vector<shared_ptr<Shape>>;
private:
    Shapes shapes;
public:
    Figure(initializer_list<shared_ptr<Shape>> list)
    {
        for (auto&& elem : list)
            shapes.emplace_back(elem);
    }
    void accept(shared_ptr<Visitor> visitor)  override
    {
        for (auto& elem : shapes)
            elem->accept(visitor);
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    shared_ptr<Shape> figure = make_shared<Figure>(
        initializer_list<shared_ptr<Shape>>(
            { make_shared<Circle>(), make_shared<Rectangle>(), make_shared<Circle>() }
        )
    );
    shared_ptr<Visitor> visitor = make_shared<ConVisitor>();
    figure->accept(visitor);
}
```
{% endcode %}

## Преимущества

* объединение разных иерархий в одну (решение проблемы [стратегии](../strategy.md))
* значительное упрощение схемы использования
* отсутствие оберточных функций (решение проблемы [адаптера](../../structural-patterns/adapter/))
* открытость кода для расширения: добавление новых классов-посетителей позволяет расширять функциональность без изменения существующего кода
* упрощение поддержки разных типов объектов

## Недостатки

* расширение иерархии, добавление новых классов приводит к необходимости модификации посетителей

{% hint style="info" %}
Неудобство при добавлении новых типов объектов: требуется изменение всех классов посетителей.
{% endhint %}

* необходимость установления дружественных связей для обеспечения доступа к реализации

{% hint style="info" %}
Посетитель получает доступ к внутреннему состоянию объекта, с которым взаимодействует. Это может привести к потенциальному раскрытию деталей реализации объектов и нарушению принципа инкапсуляции.
{% endhint %}

* проблема связи на уровне базовых классов

{% hint style="info" %}
Происходит из-за сильной зависимости между посетителем и элементами, которые он посещает.
{% endhint %}

## Связь с другими паттернами

* [Компоновщик](../../structural-patterns/composite.md): посетители могут использоваться для выполнения операции над всеми объектами структуры, определенной с помощью паттерна компоновщик.
* [Посредник](../opekun.md): посетитель может быть связан с паттерном посредник для обмена информацией между различными объектами в системе. Посредник может передавать посетителям необходимую информацию о состоянии объектов, чтобы они могли выполнить соответствующие операции.
* [Стратегия](../strategy.md): посетитель может быть связан с паттерном стратегия для предоставления различных стратегий обработки элементов структуры. Посетитель может быть реализован как одна из стратегий, которая может быть выбрана во время выполнения в зависимости от требуемой операции.
