---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Абстрактная фабрика

## Назначение

Абстрактная фабрика(Abstarct factory) — это порождающий паттерн проектирования, который позволяет создавать семейства связанных объектов, без указания конкретных классов создаваемых объектов. Паттерн подразумевает создание абстрактной фабрики, которая определяет интерфейс для создания семейств связанных объектов. Затем создаются конкретные фабрики, которые реализуют этот интерфейс и отвечают за создание конкретных объектов определенного семейства. Шаблон позволяет классу делегировать создание семейства объектов подклассам. Используется когда заранее не известны типы объектов, с которыми должен работать код. Использование этого шаблона помогает избавить код от явного создания конкретных объектов (от оператора new). Вместо этого, код будет работать с абстрактными интерфейсами, определенными в фабрике.

{% hint style="info" %}
Семейством связанных объектов могут быть, например, разные виды кистей (brash, pen, marker) в графических редакторах.
{% endhint %}

### Решаемые задачи

* Отделение принятия решения о том, какой объект нужно создать, от самого процесса создания объекта

Решение о выборе конкретного типа объекта принимается в части кода, который использует фабрику. Во время выполнения этот код определяет, какой тип объекта требуется создать, и вызывает соответствующий метод фабрики. Фактическое создание объекта происходит внутри фабрики.

* Возможность создавать и подменять семейства объектов

Появляется возможность создавать группы объектов, которые взаимосвязаны между собой и имеют общую логику, без привязки к конкретным классам. Это позволяет легко заменять или добавлять новые семейства объектов, не изменяя другие части кода, которые используют эти объекты.

* Повторное использование объектов

Появляется возможность повтороного использования одних и тех же объектов, которые были созданы в разных местах программы. Когда требуется создание объекта, вызывается метод фабрики, которая создает требуемый объект. Если объект уже был создан и сохранен в памяти, фабрика может вернуть ссылку на этот объект, вместо создания нового экземпляра.

## Общая реализация на языке с++

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
```cpp
class AbstractGraphFactory
{
public:
    virtual unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) = 0;
    
    virtual unique_ptr<BasePen> createPen(shared_ptr<Color> cl) = 0;
    
    virtual unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) = 0;
};
```
{% endtab %}

{% tab title="QtGraphFactory" %}
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
{% endtab %}

{% tab title="BasePen" %}
```cpp
class BasePen {};
```
{% endtab %}

{% tab title="BaseBrush" %}
```cpp
class BaseBrush {};
```
{% endtab %}

{% tab title="QtPen" %}
```cpp
class QtPen : public BasePen {};
```
{% endtab %}

{% tab title="QtBrush" %}
```cpp
class QtBrush : public BaseBrush {};
```
{% endtab %}

{% tab title="Image" %}
```cpp
class Image {};

```
{% endtab %}

{% tab title="Color" %}
```cpp
class Color {};
```
{% endtab %}

{% tab title="main" %}
```cpp
int main()
{
    shared_ptr<AbstractGraphFactory> graphfactory = make_shared<QtGraphFactory>();
    shared_ptr<Image> image = make_shared<Image>();
    shared_ptr<BaseGraphics> graphics1 = grfactory->createGraphics(image);
    shared_ptr<BaseGraphics> graphics2 = grfactory->createGraphics(image);
}
```
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" %}
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

## Преимущества

1. Избавляение от необходимости создания конкретного объекта в коде.
2. Создание семейства объектов в одном место упрощает поддержку и изменение кода.
3. Возможность во время выполнения программы принимать решение, какой объект создавать.
4. Возможность во время выполнения программы подменять создание одного объекта созданием другого.
5. Упрощается добавление новых классов без изменения написанного кода.

## Недостатки

1. Увеличение количества кода за счёт появления параллельных иерархий классов, усложнение разработки и проектирования.
2. Увеличение времени компиляции и исполнения.
3. Необходимость перекомплиировать один и тот же код при добавлении новых типов объектов.
4. У абстрактной фабрики должен быть базовый интерфейс, если у стронних библиотек используются различные понятия, выделить базовое для них не всегда возможно.

## Связь с другими паттернами

* Абстрактная фабрика может использовать [Фабричный метод](factory-method.md) для создания конкретных объектов. Вместо того, чтобы иметь только один метод для создания объектов, абстрактная фабрика может использовать фабричный метод в своей реализации.
* Абстрактная фабрика может быть комбинирована с паттерном [Строитель](builder.md) для создания сложных объектов.
* Абстрактная фабрика может использовать [Прототип](prototype.md) для создания объектов на основе существующих прототипов. Абстрактная фабрика может использовать уже существующие прототипы и их клонирование для создания новых экземпляров объектов.
