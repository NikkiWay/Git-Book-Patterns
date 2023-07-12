---
description: Composite
---

# Компоновщик

## Назначение

Компонует объекты в древовидные структуры для представления иерархий часть-целое. Позволяет клиентам единообразно трактовать индивидуальные и составные объекты.

#### Используется в случаях, когда:

* Необходимо объединять группы схожих объектов и управлять ими.
* Объекты могут быть как примитивными (элементарными), так и составными (сложными). Составной объект может включать в себя коллекции других объектов, образуя сложные древовидные структуры.&#x20;

{% hint style="info" %}
Пример: директория файловой системы состоит из элементов, каждый их которых также может быть директорией.
{% endhint %}

* Код клиента работает с примитивными и составными объектами единообразно.

## Решаемые задачи

* композиция объектов

Дает возможность представления иерархии объектов вида часть-целое. С помощью композиции можно создавать древовидные структуры объектов, группировать объекты в контейнеры и управлять ими единообразно.

* предоставление единообразного интерфейса

Единообразная трактовка клиентами составных и индивидуальных объектов, то есть паттерн компоновщик позволяет клиентам взаимодействовать с отдельными объектами и группами объектов (составными объектами) через единый интерфейс.

{% hint style="info" %}
В качестве клиентов может выступать пользовательский код или другие классы, взаимодействующие с объектами иерархии Composite.
{% endhint %}

* создание рекурсивных операций&#x20;

Паттерн компоновщик позволяет выполнять операции на составных объектах рекурсивно. Когда операция вызывается на составном объекте, он автоматически распространяет операцию на все объекты в иерархии, включая объекты, содержащие другие объекты. Просиходит рекурсивный проход по иерархии. Таким образом, можно легко применять операции как к отдельным объектам, так и ко всему дереву объектов.

## Общая реализация на языке С++

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <initializer_list>
# include <memory>
# include <vector>
```
{% endcode %}
{% endtab %}

{% tab title="usings" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
using namespace std;

class Component

using PtrComponent = shared_ptr<Component>;
using VectorComponent = vector<PtrComponent>;
```
{% endcode %}
{% endtab %}

{% tab title="Component" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Component
{
public:
    using value_type = Component;
    using size_type = size_t;
    using iterator = VectorComponent::const_iterator;
    using const_iterator = VectorComponent::const_iterator;

    virtual ~Component() = default;

    virtual void operation() = 0;

    virtual bool isComposite() const 
    { 
        return false; 
    }
    virtual bool add(initializer_list<PtrComponent> comp) 
    { 
        return false; 
    }
    virtual bool remove(const iterator& it) 
    { 
        return false; 
    }
    virtual iterator begin() const 
    { 
        return iterator(); 
    }
    virtual iterator end() const 
    { 
        return iterator(); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Figure" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Figure : public Component
{
public:
    virtual void operation() override 
    { 
        cout << "Figure method;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Camera" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Camera : public Component
{
public:
    virtual void operation() override 
    { 
        cout << "Camera method;" << endl; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Composite" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Composite : public Component
{
private:
    VectorComponent vec;
public:
    Composite() = default;
    Composite(PtrComponent first, ...);

    void operation() override;

    bool isComposite() const override 
    { 
        return true; 
    }
    bool add(initializer_list<PtrComponent> list) override;
    bool remove(const iterator& it) override 
    { 
        vec.erase(it); return true; 
    }
    iterator begin() const override 
    { 
        return vec.begin(); 
    }
    iterator end() const override 
    { 
        return vec.end(); 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="Composite Methods" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# pragma region Methods
Composite::Composite(PtrComponent first, ...)
{
    for (shared_ptr<Component>* ptr = &first; *ptr; ++ptr)
        vec.push_back(*ptr);
}

void Composite::operation()
{
    cout << "Composite method:" << endl;
    for (auto elem : vec)
        elem->operation();
}

bool Composite::add(initializer_list<PtrComponent> list)
{
    for (auto elem : list)
        vec.push_back(elem);
return true;
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
    using Default = shared_ptr<Component>;
    PtrComponent fig = make_shared<Figure>(), cam = make_shared<Camera>();
    auto composite1 = make_shared<Composite>(fig, cam, Default{});

    composite1->add({ make_shared<Figure>(), make_shared<Camera>() });
    composite1->operation();
    cout << endl;

    auto it = composite1->begin();

    composite1->remove(++it);
    composite1->operation();
    cout << endl;

    auto composite2 = make_shared<Composite>(make_shared<Figure>(), composite1, Default());

    composite2->operation();
}
```
{% endcode %}

## Преимущества

* упрощение архитектуры клиентского кода

Предоставление клиентскому коду удобного единого интерфейса для работы как с отдельными, так и со составными объектами позволяет ему работать с иерархией объектов без необходимости проверять их типы и выбирать разные пути обработки.

* повышение гибкости и расширяемости системы за счет добавления новых компонентов в иерархию без изменения существующего кода
* возможность выполнения рекурсивных операций на составных объектах
* предоставление удобных методов для добавления, удаления и обхода компонентов в иерархии объектов

## Недостатки



## Связь с другими паттернами
