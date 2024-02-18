---
description: Publisher-subscriber
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="ISubscriber" %}
{% code fullWidth="true" %}
```kotlin
interface ISubscriber {
    fun writeComment()
}
```
{% endcode %}
{% endtab %}

{% tab title="GoodSubscriber" %}
{% code fullWidth="true" %}
```kotlin
class GoodSubscriber : ISubscriber {
    override fun writeComment() {
        println("Subscriber writes positive comment in comment section.")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="NeutralSubscriber" %}
{% code fullWidth="true" %}
```kotlin
class NeutralSubscriber : ISubscriber {
    override fun writeComment() {
        println("Subscriber writes nothing.")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="BadSubscriber" %}
{% code fullWidth="true" %}
```kotlin
class BadSubscriber : ISubscriber {
    override fun writeComment() {
        println("Subscriber writes negative comment in comment section.")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Publisher" %}
{% code fullWidth="true" %}
```kotlin
class Publisher {
    private val subs: MutableSet<ISubscriber> = mutableSetOf()

    fun subscribe(subscriber: ISubscriber): Boolean = subs.add(subscriber)

    fun unsubscribe(subscriber: ISubscriber): Boolean = subs.remove(subscriber)

    fun doAction() {
        println("Creating post")
        for (sub in subs)
            sub.writeComment()
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val publisher = Publisher()

    val goodSubscriber = GoodSubscriber()
    val badSubscriber = BadSubscriber()
    val neutralSubscriber = NeutralSubscriber()

    publisher.subscribe(goodSubscriber)
    publisher.subscribe(badSubscriber)
    publisher.subscribe(neutralSubscriber)

    publisher.doAction()

    publisher.unsubscribe(badSubscriber)

    publisher.doAction()
}
```
{% endcode %}
