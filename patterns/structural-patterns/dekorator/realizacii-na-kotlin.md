---
description: Decorator
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="ICoffee" %}
{% code fullWidth="true" %}
```kotlin
interface ICoffee {
    val cost: Int
    fun makeCoffee()
}
```
{% endcode %}
{% endtab %}

{% tab title="SimpleCoffee" %}
{% code fullWidth="true" %}
```kotlin
class SimpleCoffee : ICoffee {

    override val cost: Int = BASE_COST

    override fun makeCoffee() {
        println("Simple coffee was made.")
    }

    companion object {
        private const val BASE_COST = 5
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="CoffeeDecorator" %}
{% code fullWidth="true" %}
```kotlin
abstract class CoffeeDecorator(private val coffee: ICoffee) : ICoffee {

    override val cost: Int = coffee.cost

    override fun makeCoffee() {
        coffee.makeCoffee()
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="MilkDecorator" %}
{% code fullWidth="true" %}
```kotlin
class MilkDecorator(coffee: ICoffee) : CoffeeDecorator(coffee) {

    override val cost: Int = super.cost + MILK_COST

    override fun makeCoffee() {
        println("Milk was added.")
    }

    companion object {
        private const val MILK_COST = 2
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="SugarDecorator" %}
{% code fullWidth="true" %}
```kotlin
class SugarDecorator(coffee: ICoffee) : CoffeeDecorator(coffee) {

    override val cost: Int = super.cost + SUGAR_COST

    override fun makeCoffee() {
        println("Sugar was added.")
    }

    companion object {
        private const val SUGAR_COST = 1
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val simpleCoffee: ICoffee = SimpleCoffee()
    simpleCoffee.makeCoffee()
    println("Current cost of coffee is: ${simpleCoffee.cost}")

    val milkCoffee: ICoffee = MilkDecorator(simpleCoffee)
    milkCoffee.makeCoffee()
    println("Current cost of coffee is: ${milkCoffee.cost}")

    val milkAndSugarCoffee: ICoffee = SugarDecorator(milkCoffee)
    milkAndSugarCoffee.makeCoffee()
    println("Current cost of coffee is: ${milkAndSugarCoffee.cost}")
}
```
{% endcode %}
