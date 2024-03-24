---
description: Singleton
---

# Возможные реализации для решения конкретных задач на С++

## Шаблон одиночка (Singleton)

{% tabs %}
{% tab title="NotAbstractClass" %}
{% code fullWidth="true" %}
```cpp
template <typename T>
concept NotAbstractClass = is_class_v<T> && !is_abstract_v<T>;
```
{% endcode %}
{% endtab %}

{% tab title="CopyConstructible" %}
{% code fullWidth="true" %}
```cpp
template <typename T>
concept CopyConstructible = requires(T t)
{
	T(t);
};
```
{% endcode %}
{% endtab %}

{% tab title="Assignable" %}
{% code fullWidth="true" %}
```cpp
template <typename T>
concept Assignable = requires(T t1, T t2)
{
	t1 = t2;
};
```
{% endcode %}
{% endtab %}

{% tab title="OnlyObject" %}
{% code fullWidth="true" %}
```cpp
template <typename T>
concept OnlyObject = NotAbstractClass<T> && !CopyConstructible<T> && !Assignable<T>;
```
{% endcode %}
{% endtab %}

{% tab title="Singleton" %}
{% code fullWidth="true" %}
```cpp
template <OnlyObject Type>
class Singleton
{
private:
	static unique_ptr<Type> inst;

public:
	template <typename... Args>
	static Type& instance(Args&& ...args)
	{
		struct Proxy : public Type
		{
			Proxy(Args&& ...args) : Type(forward<Args>(args)...) {}
		};

		if (!inst)
			inst = make_unique<Proxy>(forward<Args>(args)...);

		return *inst;
	}

	Singleton() = delete;
	Singleton(const Singleton&) = delete;
	Singleton<Type>& operator =(const Singleton&) = delete;
};

template <OnlyObject Type>
unique_ptr<Type> Singleton<Type>::inst{};
```
{% endcode %}
{% endtab %}

{% tab title="Sun" %}
{% code fullWidth="true" %}
```cpp
class Sun
{
private:
	int age;
	double brightness;

protected:
	Sun() = default;
	Sun(int n, double d) : age(n), brightness(d)
	{
		cout << "Calling the constructor;" << endl;
	}

public:
	~Sun() 
	{ 
		cout << "Calling the destructor;" << endl; 
	}

	void f() 
	{ 
		cout << "age = " << age << "; brightness = " << brightness << endl; 
	}

	Sun(const Sun&) = delete;
	Sun& operator =(const Sun&) = delete;
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>

using namespace std;

int main()
{
	decltype(auto) d1 = Singleton<Sun>::instance(1, 2.);
	decltype(auto) d2 = Singleton<Sun>::instance();

	d2.f();
}
```
{% endcode %}
