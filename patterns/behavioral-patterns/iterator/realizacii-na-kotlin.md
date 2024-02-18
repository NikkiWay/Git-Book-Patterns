---
description: Iterator
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="IIterator" %}
{% code fullWidth="true" %}
```kotlin
interface IIterator<T> {
    fun hasNext(): Boolean
    fun next(): T
}
```
{% endcode %}
{% endtab %}

{% tab title="ReverseIterator" %}
{% code fullWidth="true" %}
```kotlin
class ReverseIterator<T>(private val collection: List<T>) : IIterator<T> {
    private var currentIndex: Int = collection.size - 1

    override fun hasNext(): Boolean = currentIndex >= 0

    override fun next(): T {
        val item = collection[currentIndex--]
        return item
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="ListUtil" %}
{% code fullWidth="true" %}
```kotlin
object ListUtil {
    fun <T>List<T>.reverseIterator(): ReverseIterator<T> = ReverseIterator(this)
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val digits = listOf(1, 3, 5, 9, 7, 3)
    val iterator = digits.reverseIterator()
    while (iterator.hasNext()) {
        println(iterator.next())
    }

    // default iterator
    for (digit in digits) {
        println(digit)
    }
}
```
{% endcode %}
