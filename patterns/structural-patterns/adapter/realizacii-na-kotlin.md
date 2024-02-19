---
description: Adapter
---

# Реализации на Kotlin

## UML диаграмма

<figure><img src="../../../.gitbook/assets/adapter.png" alt=""><figcaption><p>UML диаграмма для общей реализации паттерна "Адаптер" на Kotlin</p></figcaption></figure>

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="IThermometer" %}
{% code fullWidth="true" %}
```kotlin
interface IThermometer {
    val temperature: Double
}
```
{% endcode %}
{% endtab %}

{% tab title="CelsiusThermometer" %}
{% code fullWidth="true" %}
```kotlin
class CelsiusThermometer : IThermometer {
    override val temperature: Double = DEFAULT_TEMPERATURE

    companion object {
        private const val DEFAULT_TEMPERATURE: Double = 15.0
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="ITempAdapter" %}
{% code fullWidth="true" %}
```kotlin
interface ITempAdapter {
    val temperature: Double
}
```
{% endcode %}
{% endtab %}

{% tab title="FahrenheitTempAdapter" %}
{% code fullWidth="true" %}
```kotlin
class FahrenheitTempAdapter(
    private val thermometer: IThermometer
) : ITempAdapter {
    override val temperature: Double = thermometer.temperature * FACTOR + LINEAR_COEFFICIENT

    companion object {
        private const val FACTOR = 1.8
        private const val LINEAR_COEFFICIENT = 32.0
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val thermometer: IThermometer = CelsiusThermometer()
    println("Temperature in celsius: ${thermometer.temperature}")
    val adapter: ITempAdapter = FahrenheitTempAdapter(thermometer)
    println("Temperature in fahrenheit: ${adapter.temperature}")
}
```
{% endcode %}
