---
description: Proxy
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="incldes" %}
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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

{% tab title="RealSubject methods" %}
{% code fullWidth="true" %}
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

pair<bool, double> RealSubject::request(size_t index)
{
    random_device rd;
    mt19937 gen(rd());

    return pair<bool, double>(true, generate_canonical<double, 10>(gen));
}
```
{% endcode %}
{% endtab %}

{% tab title="ConcreteProxy methods" %}
{% code fullWidth="true" %}
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
