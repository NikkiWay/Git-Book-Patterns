---
description: Singleton
---

# Реализации на C++

## Общая реализация на языке С++

{% tabs %}
{% tab title="Sun" %}
{% hint style="info" %}
Конструктор помечается модификатором private, чтобы объект класса нельзя было создать извне
{% endhint %}

{% code fullWidth="true" %}
```cpp
class Sun
{
public:
	static shared_ptr<Sun> instance()
	{
		class SunProxy : public Sun {};

		static shared_ptr<Sun> myInstance = make_shared<SunProxy>();

		return myInstance;
	}
	
	~Sun() 
	{ 
		cout << "Calling the destructor;" << endl; 
	}

	void shine() 
	{ 
		cout << "The sun is shining;" << endl; 
	}

	Sun(const Sun&) = delete;
	Sun& operator =(const Sun&) = delete;

private:
	Sun() 
	{ 
		cout << "Calling the default constructor;" << endl; 
	}
};
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
	shared_ptr<Sun> sun(Sun::instance());

	sun->shine();
}
```
{% endcode %}
