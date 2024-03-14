---
description: Factory method
---

# Возможные реализации для решения конкретных задач на Kotlin

## Фабричный метод с использованием Generics

([посмотреть реализацию на С++](concrete-implementation.md#fabrichnyi-metod-s-shablonnym-carfactory))

{% tabs %}
{% tab title="ICar" %}
{% code fullWidth="true" %}
```kotlin
interface ICar {
    fun drive()
}
```
{% endcode %}
{% endtab %}

{% tab title="Sedan" %}
{% code fullWidth="true" %}
```kotlin
class Sedan : ICar {
    init {
        println("Sedan constructor called")
    }

    override fun drive() {
        println("Driving sedan")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="CarFactory" %}
{% code fullWidth="true" %}
```kotlin
class CarFactory<T : ICar>(private val carClass: Class<T>) {
    fun produce(): T = carClass.getDeclaredConstructor().newInstance()
}

inline fun <reified T : ICar> createCarFactory(): CarFactory<T> = CarFactory(T::class.java)
```
{% endcode %}
{% endtab %}

{% tab title="User" %}
{% code fullWidth="true" %}
```kotlin
class User {
    fun <T : ICar> use(factory: CarFactory<T>) {
        val car = factory.produce()
        car.drive()
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code fullWidth="true" %}
```kotlin
fun main() {
    val sedanFactory = createCarFactory<Sedan>()
    val user = User()
    user.use(sedanFactory)
}
```
{% endcode %}
