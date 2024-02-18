---
description: Factory method
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

#### Реализация классов машин

{% tabs %}
{% tab title="ICar" %}
{% code fullWidth="true" %}
```kotlin
interface ICar {
    val model: String
    fun drive()
    fun getDescription(): String
}
```
{% endcode %}
{% endtab %}

{% tab title="Audi" %}
{% code fullWidth="true" %}
```kotlin
class Audi: ICar {
    override val model: String = "A7"
    override fun drive() {
        println("Rides like an iron")
    }

    override fun getDescription(): String = "Brand: Audi, model: A7, revision: ${this.hashCode()}"
}
```
{% endcode %}
{% endtab %}

{% tab title="Bmw" %}
{% code fullWidth="true" %}
```kotlin
class Bmw: ICar {
    override val model: String = "3 series"

    override fun drive() {
        println("No one knows when the driver will change lanes")
    }

    override fun getDescription(): String = "Brand: Bmw, model: 3 series, revision: ${this.hashCode()}"
}
```
{% endcode %}
{% endtab %}

{% tab title="Bugatti" %}
{% code fullWidth="true" %}
```kotlin
open class Bugatti protected constructor(): ICar {
    override val model: String = "Chiron"

    override fun drive() {
        println("Drive like a boss")
    }

    override fun getDescription(): String = "Brand: Bugatti, model: Chiron, revision: ${this.hashCode()}"
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Реализация классов фабрик машин

{% tabs %}
{% tab title="CarFactory" %}
{% code fullWidth="true" %}
```kotlin
abstract class CarFactory {
    abstract fun createCar(): ICar
}
```
{% endcode %}
{% endtab %}

{% tab title="AudiFactory" %}
{% code fullWidth="true" %}
```kotlin
class AudiFactory: CarFactory() {
    override fun createCar(): Audi = Audi()
}
```
{% endcode %}
{% endtab %}

{% tab title="BmwFactory" %}
{% code fullWidth="true" %}
```kotlin
class BmwFactory: CarFactory() {
    override fun createCar(): Bmw {
        return Bmw()
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="BugattiFactory" %}
{% code fullWidth="true" %}
```kotlin
class BugattiFactory : CarFactory() {
    override fun createCar(): Bugatti {
        class ConcreteBugatti : Bugatti()
        return ConcreteBugatti()
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val audi = Audi() // можно создавать объект там где необходимо
    val audiFactory: CarFactory = AudiFactory()
    val audi1 = audiFactory.createCar()
    println(audi1.getDescription())
    audi1.drive()

    val audi2 = audiFactory.createCar()
    println(audi2.getDescription())
    audi2.drive()

    val bmw = Bmw() // можно создавать объект там где необходимо
    val bmwFactory: CarFactory = BmwFactory()
    val bmw1 = bmwFactory.createCar()
    println(bmw1.getDescription())
    bmw1.drive()

//    val bugatti = Bugatti() // ошибка компиляции, нельзя вызвать protected constructor
    val bugattiFactory = BugattiFactory()
    val bugatti1: Bugatti = bugattiFactory.createCar()
    println(bugatti1.getDescription())
    bugatti1.drive()

    val bugatti2: Bugatti = bugattiFactory.createCar()
    println(bugatti2.getDescription())
    bugatti2.drive()
}
```
{% endcode %}
