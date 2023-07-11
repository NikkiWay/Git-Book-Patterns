---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Фабричный метод

## Назначение

Фабричный метод (Factory method) - пораждающий шаблон проектирования, определяющий общий интерфейс создания объектов в родительском классе и позволяющий изменять создаваемые объекты в дочерних классах. Шаблон позволяет классу делегировать создание объектов подклассам. Используется когда заранее неизвестны типы и зависимости объектов, с которыми должен работать код. При его использовании код очищается от создания конкретных объектов(от оператора new).

## Решаемые задачи

* Отедление принятия решения от создания объекта

Решение о том какой продукт надо создавать происходит в одном месте, а создание происходит в другом месте кода.

* Добавление новых классов без изменения написанного кода

Добавление, модификация и расширение классов происходит путем добавления новых классов, а не изменения старых.

* Подмена объектов

Появляется возможность легко подменить один объект другим.

* Повторное использование объектов

Повторное использование одних и тех же объектов, которые были созданы разных местах программы.

## Общая реализация на языке с++

{% hint style="info" %}
Класс Solution предоставляет метод для регистрации в данном случае Creator'ов для карты (map), состоящий из пар (pair): ключ + значение.
{% endhint %}

{% code lineNumbers="true" %}
```cpp
# include <iostream> 
# include <memory> 
# include <map> 
using namespace std; 

class Product;

class Creator
{
public:
      virtual ~Creator() = 0;
      
      virtual unique_ptr<Product> createProduct() = 0;
};
Creator::~Creator() = default;
 
template <typename Tprod>
class ConCreator : public Creator
{
public:
      virtual unique_ptr<Product> createProduct() override
      {
            return unique_ptr<Product>(new Tprod());
      }
};  
class Product
{
public:
      virtual ~Product() = 0;
      
      virtual void run() = 0;
};
 
Product::~Product() {}
 
class ConProd1 : public Product
{
public:
      virtual ~ConProd1() override 
      { cout << "Destructor;" << endl; }
      
      virtual void run() override 
      { cout << "Method run;" << endl; }
      
unique_ptr<Creator> createConcreteCreator()
      {
      return unique_ptr<Creator>(new ConcreteCreator<ConcreteProd1>());
      }
};
 
class Solution
{
public:
      typedef unique_ptr<Creator> (*CreateCreator)();
 
      bool registration(size_t id, CreateCreator createfun)
      {
            return callbacks.insert(CallBackMap::value_type(id, createfun)).second;
      }
      
      bool check(size_t id) 
      { return callbacks.erase(id) == 1; }
 
      unique_ptr<Creator> create(size_t id)
      {
            CallBackMap::const_iterator it = callbacks.find(id);
 
            if (it == callbacks.end())
            {
//                throw IdError();
            }
            return unique_ptr<Creator>((it->second)());
      }
private:
      using CallBackMap = map<size_t, CreateCreator>;
      
      CallBackMap callbacks;
};
 
int main()
{
      Solution solution;
      solution.registration(1, createConcreteCreator);
      shared_ptr<Creator> cr(solution.create(1));
      shared_ptr<Product> ptr = cr->createProduct();
      ptr->run();
}
```
{% endcode %}

## Связь с другими паттернами

* Фабричный метод может быть использован внутри [Абстрактной фабрики](abstract-factory.md) для создания конкретных объектов. Вместо того, чтобы создавать объекты напрямую, Абстрактная фабрика может использовать Фабричный метод для создания экземпляров объектов определенного типа.
* Фабричный метод может быть использован для создания [прототипов](prototype.md) объектов вместо создания новых объектов с помощью конструктора. Вместо создания объекта с нуля, Фабричный метод может использовать существующий объект-прототип и его клонирование для создания новых экземпляров. Это позволяет избежать накладных расходов на создание объектов с нуля и обеспечивает гибкость при создании новых объектов на основе существующих.
