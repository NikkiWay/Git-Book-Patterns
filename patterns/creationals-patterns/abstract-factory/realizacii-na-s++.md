---
description: Abstarct factory
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="BaseGraphics" %}
{% code fullWidth="true" %}
```cpp
class BaseGraphics 
{
public:
    virtual ~BaseGraphics() = 0;
};

BaseGraphics::~BaseGraphics() {}
```
{% endcode %}
{% endtab %}

{% tab title="AbstractGraphFactory" %}
{% code fullWidth="true" %}
```cpp
class AbstractGraphFactory
{
public:
    virtual unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) = 0;
    
    virtual unique_ptr<BasePen> createPen(shared_ptr<Color> cl) = 0;
    
    virtual unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="QtGraphFactory" %}
{% code fullWidth="true" %}
```cpp
class QtGraphFactory : public AbstractGraphFactory
{
    virtual unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im)
    { 
        return make_unique<QtGraphics>(im); 
    }

    virtual unique_ptr<BasePen> createPen(shared_ptr<Color> cl)
    { 
        return make_unique<QtPen>(); 
    }

    virtual unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl)
    { 
         
        return make_unique<QtBrush>();
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Pen" %}
{% code fullWidth="true" %}
```cpp
class BasePen {};

class QtPen : public BasePen {};
```
{% endcode %}
{% endtab %}

{% tab title="Brush" %}
{% code fullWidth="true" %}
```cpp
class BaseBrush {};

class QtBrush : public BaseBrush {};
```
{% endcode %}
{% endtab %}

{% tab title="Image" %}
{% code fullWidth="true" %}
```cpp
class Image {};
```
{% endcode %}
{% endtab %}

{% tab title="Color" %}
{% code fullWidth="true" %}
```cpp
class Color {};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>

using namespace std;

int main()
{
    shared_ptr<AbstractGraphFactory> graphfactory = make_shared<QtGraphFactory>();
    shared_ptr<Image> image = make_shared<Image>();
    shared_ptr<BaseGraphics> graphics1 = grfactory->createGraphics(image);
    shared_ptr<BaseGraphics> graphics2 = grfactory->createGraphics(image);
}
```
{% endcode %}
