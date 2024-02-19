---
description: Builder
---

# Реализации на Kotlin

## UML диаграмма

<figure><img src="../../../.gitbook/assets/builder.png" alt=""><figcaption><p>UML диаграмма для общей реализации паттерна "Строитель" на Kotlin</p></figcaption></figure>

## Общая реализация на языке Kotlin

#### Реализация класса машины и ее компонентов

{% tabs %}
{% tab title="Car" %}
{% code fullWidth="true" %}
```kotlin
class Car(
    val body: Body,
    val engine: Engine,
    val suspension: Suspension
)
```
{% endcode %}
{% endtab %}

{% tab title="Body" %}
{% code fullWidth="true" %}
```kotlin
abstract class Body(val bodyType: String)

class SedanBody(bodyType: String) : Body(bodyType)
```
{% endcode %}
{% endtab %}

{% tab title="Engine" %}
{% code fullWidth="true" %}
```kotlin
abstract class Engine(val capacity: Double)

class InternalCombustionEngine(capacity: Double) : Engine(capacity)
```
{% endcode %}
{% endtab %}

{% tab title="Suspension" %}
{% code fullWidth="true" %}
```kotlin
abstract class Suspension(val type: String)

class AirSuspension(type: String): Suspension(type)
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Реализация Builder и Director

{% tabs %}
{% tab title="IBuilder" %}
{% code fullWidth="true" %}
```kotlin
interface IBuilder {
    fun withBody(body: Body): IBuilder
    fun withEngine(engine: Engine): IBuilder
    fun withSuspension(suspension: Suspension): IBuilder
    fun build(): Car
}
```
{% endcode %}
{% endtab %}

{% tab title="CamryBuilder" %}
{% code fullWidth="true" %}
```kotlin
class CamryBuilder : IBuilder {
    private var body: Body = SedanBody("sedan")
    private var engine: Engine = InternalCombustionEngine(2.5)
    private var suspension: Suspension = AirSuspension("mono-cylinder")

    override fun withBody(body: Body): CamryBuilder {
        this.body = body
        return this
    }

    override fun withEngine(engine: Engine): CamryBuilder {
        this.engine = engine
        return this
    }

    override fun withSuspension(suspension: Suspension): CamryBuilder {
        this.suspension = suspension
        return this
    }

    override fun build(): Car {
        return Car(
            body = body,
            engine = engine,
            suspension = suspension,
        )
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Director" %}
{% code fullWidth="true" %}
```kotlin
class Director(private val builder: IBuilder) {
    fun create(): Car = builder
        .withEngine(InternalCombustionEngine(3.5))
        .build()
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val director = Director(CamryBuilder())
    val camry = director.create()

    println(
        "Director created camry! " +
                "Body: ${camry.body.bodyType}, " +
                "engine: ${camry.engine.capacity}, " +
                "suspension: ${camry.suspension.type}"
    )
}
```
{% endcode %}
