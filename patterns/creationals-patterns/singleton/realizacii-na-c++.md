---
description: Singleton
---

# Реализации на C++

## Общая реализация на языке С++

{% tabs %}
{% tab title="Product" %}
{% hint style="info" %}
Конструктор помечается модификатором private, чтобы объект класса нельзя было создать извне
{% endhint %}

{% code fullWidth="true" %}
```cpp
class Product
{
public:
   static shared_ptr<Product> instance()
   {
       static shared_ptr<Product> myInstance(new Product());
       return myInstance;
   }
   
   ~Product() 
   {
        cout << "Destructor;" << endl; 
   }

   void f() 
   {
        cout << "Method f;" << endl; 
   }

   Product(const Product&) = delete; 
   Product& operator=(const Product&) = delete; 

private:
   Product() 
   { 
        cout << "Default constructor;" << endl; 
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
   shared_ptr<Product> ptr(Product::instance());

   ptr->f();
}
```
{% endcode %}
