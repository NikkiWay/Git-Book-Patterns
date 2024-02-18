---
description: Prototype
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="Candy" %}
{% code fullWidth="true" %}
```kotlin
data class Candy(
    val name: String,
    val taste: String,
    val manufacture: String
)
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val myCandy = Candy("Mishka","delicious", "Noname")
    val sistersCandy = myCandy.copy()

    println("Is candies equals: ${myCandy == sistersCandy}")
    println("My candy: ${System.identityHashCode(myCandy)}")
    println("Sisters candy: ${System.identityHashCode(sistersCandy)}")
}
```
{% endcode %}
