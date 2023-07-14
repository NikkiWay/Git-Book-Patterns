---
description: Proxy
---

# Заместитель

## Назначение

Паттерн проектирования заместитель имитирует интерфейс и поведение оригинального объекта и контролирует доступ к нему.

{% hint style="info" %}
Заместитель обычно создается для того, чтобы предоставить дополнительные функции или управлять доступом к оригинальному объекту без изменения его основной логики или структуры.
{% endhint %}

Назначение паттерна заместитель заключается в добавлении дополнительного уровня абстракции между клиентом и реальным объектом.&#x20;

## Решаемые задачи

* предоставление локального объекта, который работает с удаленным объектом, скрывая детали удаленного взаимодействия \[удаленный заместитель (Remote Proxy)]
* создание заглушки для ресурсоемкого объекта, позволяющей откладывать его создание или загрузку до момента реального использования \[виртуальный прокси (Virtual Proxy)]
* контроль доступа к объекту, ограничение различных операций в зависимости от прав доступа \[защищающий заместитель (Protection Proxy)]
* выполнение дополнительных действий при доступе к объекту, например, подсчет ссылок на объект или отслеживание изменений \["умная" ссылка (Smart Reference)]
* кэширование результатов операций объекта для избегания повторных вычислений или загрузки \[кэширующий заместитель (Caching Proxy)]

## Общая реализация на С++

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <map>
# include <random>

using namespace std;
```
{% endcode %}
{% endtab %}

{% tab title="Subject" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Subject
{
public:
    virtual ~Subject() = default;

    virtual pair<bool, double> request(size_t index) = 0;
    virtual bool changed() 
    { 
        return true; 
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="RealSubject" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class RealSubject : public Subject
{
private:
    bool flag{ false };
    size_t counter{ 0 };
public:
    virtual pair<bool, double> request(size_t index) override;
    virtual bool changed() override;
};
```
{% endcode %}
{% endtab %}

{% tab title="Proxy" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class Proxy : public Subject
{
protected:
    shared_ptr<RealSubject> realsubject;
public:
    Proxy(shared_ptr<RealSubject> real) : realsubject(real) {}
};

```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProxy" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class ConcreteProxy : public Proxy
{
private:
    map<size_t, double> cache;
public:
    using Proxy::Proxy;
    virtual pair<bool, double> request(size_t index) override;
};
```
{% endcode %}
{% endtab %}

{% tab title="RealSubject::change() " %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
bool RealSubject::changed()
{
    if (counter == 0)
    {
        flag = true;
    }
    if (++counter == 7)
    {
        counter = 0;
        flag = false;
    }
    return flag;
}
```
{% endcode %}
{% endtab %}

{% tab title="RealSubject::request()" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
pair<bool, double> RealSubject::request(size_t index)
{
    random_device rd;
    mt19937 gen(rd());

    return pair<bool, double>(true, generate_canonical<double, 10>(gen));
}
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProxy::request()" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
pair<bool, double> ConcreteProxy::request(size_t index)
{
    pair<bool, double> result;

    if (!realsubject)
    {
        cache.clear();
        result = pair<bool, double>(false, 0.);
    }
    else if (!realsubject->changed())
    {
        cache.clear();
        result = realsubject->request(index);
        cache.insert(map<size_t, double>::value_type(index, result.second));
    }
    else
    {
        map<size_t, double>::const_iterator it = cache.find(index);
        if (it != cache.end())
        {
            result = pair<bool, double>(true, it->second);
        }
        else
        {
            result = realsubject->request(index);
            cache.insert(map<size_t, double>::value_type(index, result.second));
        }
    }
    return result;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main()
{
    shared_ptr<RealSubject> subject = make_shared<RealSubject>();
    shared_ptr<Subject> proxy = make_shared<ConcreteProxy>(subject);

    for (size_t i = 0; i < 21; ++i)
    {
        cout << "( " << i + 1 << ", " << proxy->request(i % 3).second << " )" << endl;
    if ((i + 1) % 3 == 0)
        cout << endl;
    }
}
```
{% endcode %}

## Преимущества

* контроль доступа к объекту

{% hint style="info" %}
Позволяет реализовать механизмы безопасности, аутентификации и авторизации в системе.
{% endhint %}

* возможность работы при отсутствии самого объекта

{% hint style="info" %}
В этом случае поведение заместителя может быть неоднозначным. Необходимо предупреждение о возможной недостоверности ответа
{% endhint %}

* упрощение работы с ресурсоемкими объектами, "разгрузка" тяжелого оригинального объекта
* добавление дополнительного функционала перед или после обращения к оригинальному объекту
* возможность отвечать за жизненный цикл объекта
* кэширование: возможность хранить результаты операций и предоставлять их повторно без обращения к оригинальному объекту

## Недостатки

* увеличение накладных расходов: увеличение времени выполнения операций
* усложнение кода: необходимость в добавлении дополнительных классов и методов

## Связь с другими паттернами

* [Адаптер](adapter/): предоставляет другой интерфейс к адаптируемому объекту. Напротив, заместитель в точности повторяет интерфейс своего субъекта.

{% hint style="info" %}
Интерфейс заместителя может быть и подмножеством интерфейса субъекта.
{% endhint %}

* [Декоратор](dekorator.md): реализация паттерна декоратор похожа на реализацию заместителя, но назначение совершенно иное. Декоратор добавляет объекту новые обязанности, а заместитель контролирует доступ к объекту.
* [Фабричный метод](../creationals-patterns/factory-method.md): заместитель может быть создан с помощью фабричного метода. Фабричный метод позволяет инкапсулировать процесс создания заместителя и предоставляет гибкость в выборе типа заместителя, который будет создан.
* [Одиночка](../creationals-patterns/singleton.md): заместитель может быть реализован как одиночка, гарантируя, что будет создан только один объект заместителя для доступа к оригинальному объекту
* [Стратегия](../behavioral-patterns/strategy.md): заместитель может использоваться для введения различных стратегий доступа к объекту.
