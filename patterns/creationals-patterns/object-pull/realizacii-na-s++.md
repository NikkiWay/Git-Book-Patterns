---
description: Object pool
---

# Реализации на С++

## Общая реализация на языке С++

{% tabs %}
{% tab title="EmployeePoolObject" %}
{% code fullWidth="true" %}
```cpp
template <typename T>
concept EmployeePoolObject = requires(T t)
{
	t.clockIn();
};
```
{% endcode %}
{% endtab %}

{% tab title="Employee" %}
{% code fullWidth="true" %}
```cpp
class Employee
{
private:
	static size_t count;

public:
	Employee() 
	{ 
		cout << "Constructor(" << ++count << ");" << endl; 
	}
	
	~Employee() 
	{ 
		cout << "Destructor(" << count-- << ");" << endl; 
	}

	void clockIn() 
	{ 
		cout << "Employee clocked in: 0x" << this << endl; 
	}
};

size_t Employee::count = 0;
```
{% endcode %}
{% endtab %}

{% tab title="EmployeePool" %}
{% code fullWidth="true" %}
```cpp
template <EmployeePoolObject Type>
class EmployeePool
{
public:
	static shared_ptr<EmployeePool<Type>> instance();

	shared_ptr<Type> hireEmployee();
	bool fireEmployee(shared_ptr<Type>& employee);
	size_t count() const { return pool.size(); }

	EmployeePool(const EmployeePool&) = delete;
	EmployeePool& operator =(const EmployeePool&) = delete;

private:
	vector<pair<bool, shared_ptr<Type>>> pool;

	EmployeePool() {}

	pair<bool, shared_ptr<Type>> createEmployee();

	template <typename Type>
	friend ostream& operator << (ostream& os, const EmployeePool<Type>& pl);
};
```
{% endcode %}
{% endtab %}

{% tab title="instance()" %}
{% code fullWidth="true" %}
```cpp
template <EmployeePoolObject Type>
shared_ptr<EmployeePool<Type>> EmployeePool<Type>::instance()
{
	static shared_ptr<EmployeePool<Type>> myInstance(new EmployeePool<Type>());

	return myInstance;
}
```
{% endcode %}
{% endtab %}

{% tab title="hireEmployee()" %}
{% code fullWidth="true" %}
```cpp
template <EmployeePoolObject Type>
shared_ptr<Type> EmployeePool<Type>::hireEmployee()
{
	size_t i;
	for (i = 0; i < pool.size() && pool[i].first; ++i);

	if (i < pool.size())
	{
		pool[i].first = true;
	}
	else
	{
		pool.push_back(createEmployee());
	}

	return pool[i].second;
}
```
{% endcode %}
{% endtab %}

{% tab title="fireEmployee()" %}
{% code fullWidth="true" %}
```cpp
template <EmployeePoolObject Type>
bool EmployeePool<Type>::fireEmployee(shared_ptr<Type>& employee)
{
	size_t i;
	for (i = 0; pool[i].second != employee && i < pool.size(); ++i);

	if (i == pool.size()) return false;

	employee.reset();
	pool[i].first = false;
	pool[i].second->clockIn();

	return true;
}
```
{% endcode %}
{% endtab %}

{% tab title="createEmployee()" %}
{% code fullWidth="true" %}
```cpp
template <EmployeePoolObject Type>
pair<bool, shared_ptr<Type>> EmployeePool<Type>::createEmployee()
{
	return { true, make_shared<Type>() };
}
```
{% endcode %}
{% endtab %}

{% tab title="operator <<" %}
{% code fullWidth="true" %}
```cpp
template <typename Type>
ostream& operator << (ostream& os, const EmployeePool<Type>& pl)
{
	for (auto elem : pl.pool)
		os << "{" << elem.first << ", 0x" << elem.second << "} ";

	return os;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
# include <iostream>
# include <memory>
# include <iterator>
# include <vector>

using namespace std;

int main()
{
	shared_ptr<EmployeePool<Employee>> pool = EmployeePool<Employee>::instance();

	vector<shared_ptr<Employee>> vec(4);

	for (auto& elem : vec)
		elem = pool->hireEmployee();

	pool->fireEmployee(vec[1]);

	cout << *pool << endl;

	shared_ptr<Employee> ptr = pool->hireEmployee();
	vec[1] = pool->hireEmployee();

	cout << *pool << endl;
}
```
{% endcode %}
