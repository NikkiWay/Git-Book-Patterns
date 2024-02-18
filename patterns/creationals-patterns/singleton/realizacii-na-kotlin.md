---
description: Singleton
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="Singleton" %}
{% code fullWidth="true" %}
```kotlin
object Singleton
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val single1 = Singleton
    val single2 = Singleton
    println("Are the same object: ${single1 == single2}")
}
```
{% endcode %}

## Использование ленивой инициализации&#x20;

{% tabs %}
{% tab title="LazySingleton" %}
{% code fullWidth="true" %}
```kotlin
object LazySingleton {
    val lazyField: Teacher by lazy(LazyThreadSafetyMode.PUBLICATION) {
        Teacher(name = "Ivan", surname = "Ivanov")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Teacher" %}
{% code fullWidth="true" %}
```kotlin
data class Teacher(
    val name: String,
    val surname: String
)
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val single3 = LazySingleton.lazyField
    val single4 = LazySingleton.lazyField

    println("Are the same object: ${single3.name == single4.name && single3.surname == single4.surname}")
}
```
{% endcode %}
