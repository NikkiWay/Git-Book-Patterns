---
description: Resourse Acquisition Is Initialization
---

# RAII

## Альтернативные названия

* CADR (Constructor Acquires, Destructor Releases)
* SBRM (Scope-Bound Resourse Management)

## Назначение

Resource Acquisition Is Initialization (RAII) – программная идиома, смысл которой заключается в том, чтобы с помощью тех или иных программных механизмов получение некоторого ресурса неразрывно совмещалось с инициализацией, а освобождение – с уничтожением объекта.

{% hint style="info" %}
C++ не имеет встроенного процесса автоматической сборки мусора. Вместо этого программа должна вручную освобождать ресурсы, чтобы избежать утечек. Современный C++ позволяет управлять ресурсами объектов, используя принцип получения ресурсов при инициализации (RAII). Это позволяет избежать утечек памяти, утечек, вызванных некорректной работой с ресурсами (файловых, сетевых, потоков, процессов, баз данных, устройств).
{% endhint %}

## Решаемые задачи

* необходимость освобождения ресурсов: например памяти или дескрипторов ресурсов

{% hint style="info" %}
Идиома RAII широко используется в стандартной библиотеке C++, чтобы управлять различными ресурсами, такими как память, файлы, потоки
{% endhint %}

* гарантированное освобождение ресурсов при выходе из области видимости
* позволяет избежать потери данных при освобождении ресурсов

## Общая реализация на С++

В приведенном примере класс File использует идиому RAII для открытия файла в конструкторе и закрытия файла в деструкторе. Это гарантирует, что если возникнет исключительная ситуация, файл будет закрыт корректно, а данные не будут утеряны.

{% hint style="info" %}
Согласно идиоме RAII, нам следует «обернуть» файловый дескриптор в объект специального класса. Тогда открытие файла соответствовало бы инициализации объекта в его конструкторе, а закрытие файла — уничтожению объекта в деструкторе. В случае ошибки при открытии файла мы бы сгенерировали исключение, и объект нашего класса не был бы создан.
{% endhint %}

{% tabs %}
{% tab title="includes" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
#include <cstdio>
#include <exception>
#include <string>
```
{% endcode %}
{% endtab %}

{% tab title="CannotOpenFileException" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class CannotOpenFileException {};
```
{% endcode %}
{% endtab %}

{% tab title="File" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class File 
{
private:
    std::FILE* f; // файловый дескриптор, который мы оборачиваем
public:
    File(const std::string& name) 
    {
        if (f = std::fopen(name.c_str(), "r"); f == nullptr) 
        {
            throw CannotOpenFileException();
        }
    }
    File(const File&) = delete;
    File& operator = (const File&) = delete;
// Конструктор перемещения
    File(File&& other) noexcept // File&& — ссылка на временный объект
    { 
        f = other.f;
        other.f = nullptr; // забираем владение дескриптором у временного объекта other
    }
// Оператор присваивания с семантикой перемещения
    File& operator = (File&& other) noexcept 
    {
        if (f != nullptr && f != other.f) 
        {
            std::fclose(f); // закрываем файл у текущего объекта
        }
        f = other.f; // забираем владение у временного объекта other
        other.f = nullptr;
        return *this;
    }
    ~File() noexcept 
    {
        if (f != nullptr) 
        {
            std::fclose(f);
        }
    }
    std::string Read() const 
    {
        ...
    }
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```cpp
int main() 
{
    try 
    {
        File file("input.txt");
        auto str = file.Read();
        // ...
    } 
    catch (const CannotOpenFileException&) 
    {
        std::cout << "File open failure!\n";
    }
}
```
{% endcode %}

## RAII в стандартной библиотеке

#### Некоторые примеры применения:

* <mark style="color:orange;">std::unique\_ptr</mark> и <mark style="color:orange;">std::shared\_ptr</mark> - умные указатели, которые используют идиому RAII для автоматического освобождения памяти в деструкторе объекта.
* <mark style="color:orange;">std::ifstream</mark> и <mark style="color:orange;">std::ofstream</mark> - классы, которые используют идиому RAII для автоматического закрытия файловых потоков в деструкторе объекта.
* <mark style="color:orange;">std::lock\_guard</mark> и <mark style="color:orange;">std::unique\_lock</mark> - классы, которые используют идиому RAII для автоматического освобождения блокировок при выходе из области видимости объекта.
* <mark style="color:orange;">std::thread</mark> - класс, который использует идиому RAII для автоматического завершения потока при выходе из области видимости объекта.

{% hint style="info" %}
Использование RAII в стандартной библиотеке C++ позволяет создавать более безопасный и понятный код, а также уменьшить количество ошибок и утечек памяти.
{% endhint %}
