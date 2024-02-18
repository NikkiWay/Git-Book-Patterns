---
description: Visitor
---

# Возможные реализации для решения конкретных задач

## Посетитель с приведением типа между базовыми классами

Реализация посетителя с приведением типа между базовыми классами использует пустой абстрактный класс посетителя, который дополняется посетителями конкретных типов при помощи реализации идиомы MixIn.

Подход может быть использован, в случае, если требуется создавать посетители с сильно разнящимся поведением относительно посещаемых типов, когда мы не хотим размещать разную логику обработки сущностей рядом в одном классе.

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
#include <iostream>
#include <vector>
#include <memory>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="AbstractVisitor" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class AbstractVisitor 
{
public:
    virtual ~AbstractVisitor() = default;
};
```
{% endcode %}
{% endtab %}

{% tab title="Visitor" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <typename T>
class Visitor 
{
public:
    virtual ~Visitor() = default;
    virtual void visit(const T&) const = 0;
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
    Shape() = default;
    virtual ~Shape() = default;
    virtual void accept(const AbstractVisitor&) const = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="Circle" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Circle : public Shape 
{
private:
    double radius;
public:
    Circle(double radius) : radius(radius) {}
    void accept(const AbstractVisitor& v) const override 
    {
        auto cv = dynamic_cast<const Visitor<Circle>*>(&v);
        if (cv)
            cv->visit(*this);
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Square" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Square : public Shape 
{
private:
    double side;
public:
    Square(double side) : side(side) {}
    void accept(const AbstractVisitor& v) const override 
    {
        auto cv = dynamic_cast<const Visitor<Square>*>(&v);
        if (cv)
            cv->visit(*this);
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="DrawCircle" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class DrawCircle : public Visitor<Circle> 
{
    void visit(const Circle& circle) const override 
    {
        cout << "Circle" << endl;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="DrawSquare" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class DrawSquare : public Visitor<Square> 
{
    void visit(const Square& circle) const override 
    {
        cout << "Square" << endl;
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
    void accept(const AbstractVisitor& visitor) const override 
    {
        for (auto& elem : shapes)
            elem->accept(visitor);
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Draw" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Draw : public AbstractVisitor, public DrawCircle, public DrawSquare {};
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
            { make_shared<Circle>(1), make_shared<Square>(2) }
        )
    );
    figure->accept(Draw{});
}
```
{% endcode %}

## Посетитель с использованием шаблона variant ("безопасный" union)

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
#include <iostream>
#include <vector>
#include <variant>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="Circle" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Circle {};
```
{% endcode %}
{% endtab %}

{% tab title="Square" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Square {};
```
{% endcode %}
{% endtab %}

{% tab title="Shape" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
using Shape = std::variant<Circle, Square>;
```
{% endcode %}
{% endtab %}

{% tab title="Formation" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Formation
{
public:
    static vector<Shape> initialization(initializer_list<Shape> list)
    {
        vector<Shape> vec;
        for (auto&& elem : list)
            vec.emplace_back(elem);
        return vec;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Draw" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Draw
{
public:
    void operator ()(const Circle&) const 
    { 
        cout << "Circle" << endl; 
    }
    void operator ()(const Square&) const 
    { 
        cout << "Square" << endl; 
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
    using Shapes = vector<Shape>;
    Shapes fiqure = Formation::initialization({ Circle{}, Square{} });
    for (const auto& elem : fiqure)
        std::visit(Draw{}, elem);
}
```
{% endcode %}

## Шаблонный посетитель (Template visitor) с использованием паттерна CRTP (Curiously Recurring Template Pattern)

Реализация шаблонного посетителя прибегает к вариативным шаблонам классов для определения типа посетителя, который посещает заданные классы. За счёт использования CRTP можно избавиться от реализации метода accept в каждой сущности, перенеся ответственность за это на класс Visitable.

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <initializer_list>
# include <vector>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="Visitor" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <typename... Types>
class Visitor;

template <typename Type>
class Visitor<Type>
{
public:
    virtual void visit(Type& t) = 0;
};

template <typename Type, typename... Types>
class Visitor<Type, Types...> : public Visitor<Types...>
{
public:
    using Visitor<Types...>::visit;
    virtual void visit(Type& t) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="ShapeVisitor" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
using ShapeVisitor = Visitor<class Figure, class Camera>;
```
{% endcode %}
{% endtab %}

{% tab title="Point" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Point {};
```
{% endcode %}
{% endtab %}

{% tab title="Shape" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Shape
{
public:
    Shape(const Point& pnt) : point(pnt) {}
    virtual ~Shape() = default;
    const Point& getPoint() const 
    { 
        return point; 
    }
    void setPoint(const Point& pnt) 
    { 
        point = pnt; 
    }
    virtual void accept(shared_ptr<ShapeVisitor> v) = 0;
private:
    Point point;
};
```
{% endcode %}
{% endtab %}

{% tab title="Visitable" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
template <typename Derived>
class Visitable : public Shape
{
public:
    using Shape::Shape;
    void accept(shared_ptr<ShapeVisitor> v) override
    {
        v->visit(*static_cast<Derived*>(this));
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Figure" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Figure : public Visitable<Figure>
{
    using Visitable<Figure>::Visitable;
};
```
{% endcode %}
{% endtab %}

{% tab title="Camera" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Camera : public Visitable<Camera>
{
    using Visitable<Camera>::Visitable;
};
```
{% endcode %}
{% endtab %}

{% tab title="Composite" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Composite : public Shape
{
    using Shapes = vector<shared_ptr<Shape>>;
private:
    Shapes shapes{};
public:
    Composite(initializer_list<shared_ptr<Shape>> list) : Shape(Point{})
    {
        for (auto&& elem : list)
            shapes.emplace_back(elem);
    }
    void accept(shared_ptr<ShapeVisitor> visitor)  override
    {
        for (auto& elem : shapes)
            elem->accept(visitor);
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="DrawVisitor" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class DrawVisitor : public ShapeVisitor
{
public:
    void visit(Figure& fig) override 
    { 
        cout << "Draws a figure;" << endl; 
    }
    void visit(Camera& fig) override 
    { 
        cout << "Draws a camera;" << endl; 
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
    Point p;
    shared_ptr<Composite> figure = make_shared<Composite>(
        initializer_list<shared_ptr<Shape>>(
            { make_shared<Figure>(p), make_shared<Camera>(p), make_shared<Figure>(p) }
        )
    );
    shared_ptr<ShapeVisitor> visitor = make_shared<DrawVisitor>();
    figure->accept(visitor);
}
```
{% endcode %}
