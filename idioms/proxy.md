---
description: Заместитель
---

# Proxy

## Назначение

Идиома Proxy позволяет клиентскому коду работать с заместителем таким же образом, как с реальным объектом, не внося в него изменений.

Реализуется идиома при помощи шаблонов и наследования.

Заместитель может быть реализован как класс, который имеет тот же интерфейс, что и оригинальный объект, но может выполнять дополнительные действия перед вызовом методов оригинального объекта.

{% hint style="info" %}
Паттерн Proxy и идиома Proxy на C++ являются разными концепциями.

* Паттерн Proxy

Паттерн Proxy является одним из классических порождающих паттернов проектирования. Он предоставляет структуру для создания объекта-посредника, который выступает в роли замены или обертки для реального объекта. Прокси имитирует интерфейс реального объекта, позволяя себе встраиваться в клиентский код без изменения его логики.

Главная цель паттерна Proxy - контроль доступа к реальному объекту и предоставление дополнительной функциональности при его обращении.

* Идиома Proxy

Идиома Proxy является практикой или подходом к реализации классов-оберток или прокси-классов для достижения определенных целей. Идиома Proxy используется для предоставления прокси-объектов, которые оборачивают реальные объекты и добавляют дополнительное поведение без изменения интерфейса реальных объектов.
{% endhint %}

## Решаемые задачи

* управление доступом к ресурсам или сервисам
* кэширование данных&#x20;
* проведение логирования и аудита
* обеспечение защищенного соединения между клиентом и сервером

## Пример реализации идиомы Proxy

{% tabs %}
{% tab title="Object" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
// Интерфейс для оригинального объекта и прокси
class Object 
{
public:
    virtual void DoSomething() = 0;
};
```
{% endcode %}
{% endtab %}

{% tab title="RealObject" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
// Реализация оригинального объекта
class RealObject : public Object 
{
public:
    RealObject() 
    {
    // Здесь может быть дорогостоящая инициализация объекта
    std::cout << "RealObject: Created." << std::endl;
    }
    void DoSomething() override 
    {
    // Здесь может быть дорогостоящая операция
    std::cout << "RealObject: Doing something." << std::endl;
    }
};
```
{% endcode %}
{% endtab %}

{% tab title="ObjectProxy" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
// Прокси для оригинального объекта
class ObjectProxy : public Object {
public:
    ObjectProxy() : real_object_(nullptr) {}
    void DoSomething() override 
    {
        // Создаем оригинальный объект только при первом вызове его метода
        if (!real_object_) 
        {
            std::cout << "ObjectProxy: Creating real object." << std::endl;
            real_object_ = new RealObject();
        }    
    // Выполняем операцию на оригинальном объекте
    std::cout << "ObjectProxy: Doing something." << std::endl;
    real_object_ -> DoSomething();
    }
    ~ObjectProxy() 
    {
        delete real_object_;
    }
private:
    RealObject * real_object_;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main() 
{
    std::vector<ObjectProxy > objects(10);
    // Выполняем операцию для каждого объекта
    for (Object & obj : objects) 
        obj.DoSomething();    
    return 0;
}
```
{% endcode %}
