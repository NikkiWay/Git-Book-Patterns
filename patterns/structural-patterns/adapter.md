---
description: Adapter
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

# Адаптер

## Назначение

Паттерн адаптер (Adapter), представляющий собой программную обертку над существующими классами, преобразует их интерфейсы к виду, пригодному для последующего использования. То есть он обеспечивает совместную работу классов с несовместимыми интерфейсами, которая без него была бы невозможна.

{% hint style="info" %}
Контейнеры queue, priority\_queue и stack библиотеки стандартных шаблонов STL реализованы на базе последовательных контейнеров list, deque и vector, адаптируя их интерфейсы к нужному виду. Именно поэтому эти контейнеры называют контейнерами-адаптерами.
{% endhint %}

## Решаемые задачи

* Разнесение ответсвенностей

Когда существует сущность на которой 2 ответственности, одна из ответсвенностей выносится в адаптер. Ответсвенности делаются независимыми друг от друга.

* Подмена одного интерфейса на другой

Класс с интерфейсом, отличным от интерфейса, выделенного под этот класс в требуемой системе, стоит использовать через адаптер.

* Использование сторонних библиотек

Использовать класс из нестанадртной библиотеки следует через адаптер.

* Разделение ролей

Если у одного класса несколько ролей, можно создать простой класс, а роли внести в адаптеры

* Расширение базового интерфейса

{% hint style="info" %}
Интерфейс базового класса в общем случае плохо расширять, так как в определенный момент нужен будет функционал расширенного класса, в то время как работа всегда будет производится с указателем на базовый класс, то есть придется приводить типы с постоянными проверками, чтобы понять что за класс перед нами.
{% endhint %}

При использовании адаптера расширение может относиться не к одному классу ,а к целой иерархии классов

## Общая реализация на языке С++

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>

using namespace std;

class BaseAdaptee
{
public:
    virtual ~BaseAdaptee() = default;

    virtual void specificRequest() = 0;
};

class ConAdaptee : public BaseAdaptee
{
public:
    virtual void specificRequest() override { cout << "Method ConAdaptee;" << endl; }
};

class Adapter
{
public:
    virtual ~Adapter() = default;

    virtual void request() = 0;
};

class ConAdapter : public Adapter
{
private:
    shared_ptr<BaseAdaptee>  adaptee;

public:
    ConAdapter(shared_ptr<BaseAdaptee> ad) : adaptee(ad) {}

    virtual void request() override;
};

# pragma region Methods
void ConAdapter::request()
{
    cout << "Adapter: ";

    if (adaptee)
    {
        adaptee->specificRequest();
    }
    else
    {
        cout << "Empty!" << endl;
    }
}

# pragma endregion

int main()
{
    shared_ptr<BaseAdaptee> adaptee = make_shared<ConAdaptee>();
    shared_ptr<Adapter> adapter = make_shared<ConAdapter>(adaptee);

    adapter->request();
}
```
{% endcode %}

## Возможные реализации для решения конкретных задач

{% tabs %}
{% tab title="Адаптер (Adapter)" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>

using namespace std;

class BaseAdaptee
{
public:
    virtual ~BaseAdaptee() = default;

    virtual void specificRequest() = 0;
};

class ConAdaptee : public BaseAdaptee
{
public:
    virtual void specificRequest() override { cout << "Method ConAdaptee;" << endl; }
};

class Adapter
{
public:
    virtual ~Adapter() = default;

    virtual void request() = 0;
};

class ConAdapter : public Adapter
{
private:
    shared_ptr<BaseAdaptee>  adaptee;

public:
    ConAdapter(shared_ptr<BaseAdaptee> ad) : adaptee(ad) {}

    virtual void request() override;
};

# pragma region Methods
void ConAdapter::request()
{
    cout << "Adapter: ";

    if (adaptee)
    {
        adaptee->specificRequest();
    }
    else
    {
        cout << "Empty!" << endl;
    }
}

# pragma endregion

int main()
{
    shared_ptr<BaseAdaptee> adaptee = make_shared<ConAdaptee>();
    shared_ptr<Adapter> adapter = make_shared<ConAdapter>(adaptee);

    adapter->request();
}
```
{% endcode %}
{% endtab %}

{% tab title="Шаблон Адаптер" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <vector>

using namespace std;

class Interface
{
public:
    virtual ~Interface() = default;

    virtual void request() = 0;
};

template <typename Type>
class Adapter : public Interface
{
public:
    using MethodPtr = void (Type::*)();

    Adapter(shared_ptr<Type> o, MethodPtr m) : object(o), method(m) {}

    void request() override { ((*object).*method)(); }

private:
    shared_ptr<Type> object;
    MethodPtr method;
};

class AdapteeA
{
public:
    ~AdapteeA() { cout << "Destructor class AdapteeA;" << endl; }

    void specRequestA() { cout << "Method AdapteeA::specRequestA;" << endl; }
};

class AdapteeB
{
public:
    ~AdapteeB() { cout << "Destructor class AdapteeB;" << endl; }

    void specRequestB() { cout << "Method AdapteeB::specRequestB;" << endl; }
};

auto initialize()
{
    using InterPtr = shared_ptr<Interface>;

    vector<InterPtr> vec{
        make_shared<Adapter<AdapteeA>>(make_shared<AdapteeA>(), &AdapteeA::specRequestA),
        make_shared<Adapter<AdapteeB>>(make_shared<AdapteeB>(), &AdapteeB::specRequestB)
    };

    return vec;
}

int main()
{
    auto v = initialize();

    for (const auto& elem : v)
        elem->request();
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Связь с другими паттернами

* [Мост](bridge.md): структура паттерна мост аналогична структуре адаптера, но у моста иное назначение. Он отделяет интерфейс от реализации, чтобы то и другое можно было изменять независимо. Адаптер же призван изменить интерфейс существующего объекта.
* [Фасад](facade.md): адаптер может использоваться внутри фасада для обеспечения совместимости между подсистемами, имеющими несовместимые интерфейсы.
* [Заместитель](proxy.md): адаптер может служить в качестве простого заместителя для объекта, предоставляя тот же интерфейс, но с другой реализацией.
* [Декоратор](dekorator.md): оба паттерна имеют сходную структуру, но разные цели. Адаптер изменяет интерфейс объекта, в то время как декоратор расширяет функциональность объекта, не изменяя его интерфейса

## Достоинства

* отделяет и скрывает от клиента подробности преобразования различных интерфейсов
* позволяет адаптировать интерфейс к требуемому
* позволяет разделить роли сущности
* дает возможность независимо развивать различные ответственности сущности
* расширение интерфейса

## Недостатки

* Необходимость плодить много классов приводит к увеличению количества времени и памяти,необходимых для исполнения программы
* Дублирование кода (в различных конкретных адаптерах может требоваться одна и та же реализация методов)
* Различные сущности с одной базой могут работать с данными, с которыми адаптер может не уметь работать. Возникает поребность переносить зависимости на один уровень ниже
* Часто адаптер должен иметь доступ к реализации класса
