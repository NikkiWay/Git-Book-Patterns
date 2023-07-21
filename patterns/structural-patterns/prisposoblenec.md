---
description: Flyweight
---

# Приспособленец

## Назначение

Паттерн Приспособленец (Flyweight) — это структурный паттерн проектирования, который применяется для эффективной поддержки большого числа мелких объектов засчет разделения объектов на две части: внутреннее состояние (уникальная информация, специфичная для каждого объекта) и внешнее состояние (общая информация, которая может быть разделяемой между объектами)

## Решаемые задачи

* потребность в создании огромного количество объектов, каждый из которых  содержит часть информации, которая является общей для множества других объектов
* экономия времени на создание и уничтожение объектов

#### Данный паттерн проектирования будет эффективен при выполнении всех последующих условий:

* в приложении используется большое число объектов
* накладные расходы на хранение высоки
* большую часть состояния объектов можно вынести вовне
* многие группы объектов можно заменить относительно небольшим количеством разделяемых объектов, поскольку внешнее состояние вынесено
* приложение не зависит от идентичности объекта

## Общая реализация на языке С++

{% tabs %}
{% tab title="Glyph" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Glyph 
{
public:
    virtual ~Glyph();
    virtual void Draw(Window*, GlyphContext&);
    virtual void SetFont(Font*, GlyphContext&);
    virtual Font* GetFont(GlyphContext&);
    virtual void First(GlyphContext&);
    virtual void Next(GlyphContext&);
    virtual bool IsDone(GlyphContext&);
    virtual Glyph* Current(GlyphContext&);
    virtual void Insert(Glyph*, GlyphContext&);
    virtual void Remove(GlyphContext&);
protected:
    Glyph();
};
```
{% endcode %}
{% endtab %}

{% tab title="Character" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Character : public Glyph 
{
public:
    Character(char);
    virtual void Draw(Window*, GlyphContext&);
private:
    char _charcode;
};
```
{% endcode %}
{% endtab %}

{% tab title="GlyphContext " %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class GlyphContext 
{
public:
    GlyphContext();
    virtual ~GlyphContext();
    virtual void Next(int step = 1);
    virtual void Insert(int quantity = 1);
    virtual Font* GetFont();
    virtual void SetFont(Font*, int span = 1);
private:
    int _index;
    BTree* _fonts;
};
```
{% endcode %}
{% endtab %}

{% tab title="GlyphFactory " %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
const int NCHARCODES = 128;

class GlyphFactory 
{
public:
    GlyphFactory();
    virtual ~GlyphFactory();
    virtual Character* CreateCharacter(char);
    virtual Row* CreateRow();
    virtual Column* CreateColumn();
private:
    Character* _character[NCHARCODES];
};

# pragma region Methods
GlyphFactory::GlyphFactory () 
{
    for (int i = 0; i < NCHARCODES; ++i) 
        _character[i] = 0;
}
Character* GlyphFactory::CreateCharacter (char c) 
{
    if (!_character[c]) 
        _character[c] = new Character(c);
    return _character[c];
}
Row* GlyphFactory::CreateRow () 
{
    return new Row;
}
Column* GlyphFactory::CreateColumn () 
{
    return new Column;
}
# pragma endregion
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Преимущества

* создание более гибкого и поддерживаемого кода
* увеличение скорости выполнения ресурсоемких операций: за счет снижения нагрузки на создание и удаление объектов
* экономия оперативной памяти

## Недостатки

* расход процессорного времени на поиск/вычисление контекста
* усложнение кода программы из-за введения множества дополнительных классов

## Связь с другими паттернами

* [Компоновщик](composite.md): паттерн приспособленец часто используется в сочетании с компоновщиком для реализации иерархической структуры в виде ациклического направленного графа с разделяемыми листовыми вершинами.
* [Стратегия](../behavioral-patterns/strategy.md): часто наилучшим способом реализации объектов стратегии является паттерн приспособленец.
