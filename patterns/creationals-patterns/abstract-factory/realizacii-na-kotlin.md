---
description: Abstract factory
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

#### Определение общего интерфейса инструментов

{% code fullWidth="true" %}
```kotlin
interface ITool {
    fun use()
}
```
{% endcode %}

#### Непосредственная реализация конкретных видов инструментов

{% tabs %}
{% tab title="AbstractHacksaw" %}
{% code fullWidth="true" %}
```kotlin
abstract class AbstractHacksaw(
    private val type: String,
    private val name: String = SAW_NAME
) : ITool {
    override fun use() {
        println("Instrument: $name can be used for $type")
    }

    companion object {
        protected const val SAW_NAME: String = "Saw"
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="MetalHacksaw" %}
{% code fullWidth="true" %}
```kotlin
class MetalHacksaw : AbstractHacksaw(SAW_TYPE) {
    companion object {
        private const val SAW_TYPE = "Metal"
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="WoodHacksaw" %}
{% code fullWidth="true" %}
```kotlin
class WoodHacksaw : AbstractHacksaw(SAW_TYPE) {
    companion object {
        private const val SAW_TYPE = "Wood"
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="AbstractHammer" %}
{% code fullWidth="true" %}
```kotlin
abstract class AbstractHammer(
    private val type: String,
    private val name: String = HAMMER_NAME
) : ITool {
    override fun use() {
        println("Instrument: $name can be used for $type")
    }

    companion object {
        protected const val HAMMER_NAME: String = "Hammer"
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="MetalHammer" %}
{% code fullWidth="true" %}
```kotlin
class MetalHammer : AbstractHammer(HAMMER_TYPE) {
    companion object {
        private const val HAMMER_TYPE = "Metal"
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="WoodenHammer" %}
{% code fullWidth="true" %}
```kotlin
class WoodenHammer : AbstractHammer(HAMMER_TYPE) {
    companion object {
        private const val HAMMER_TYPE = "Wood"
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Реализация абстрактных фабрик инструментов

{% tabs %}
{% tab title="AbstractFactory" %}
{% code fullWidth="true" %}
```kotlin
interface AbstractFactory {
    fun createHacksaw(): AbstractHacksaw
    fun createHammer(): AbstractHammer
}
```
{% endcode %}
{% endtab %}

{% tab title="MetalFactory" %}
{% code fullWidth="true" %}
```kotlin
class MetalFactory : AbstractFactory {
    override fun createHammer(): MetalHammer {
        return MetalHammer()
    }

    override fun createHacksaw(): MetalHacksaw {
        return MetalHacksaw()
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="WoodFactory" %}
{% code fullWidth="true" %}
```kotlin
class WoodFactory : AbstractFactory {
    override fun createHammer(): WoodenHammer {
        return WoodenHammer()
    }

    override fun createHacksaw(): WoodHacksaw {
        return WoodHacksaw()
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Main

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val metalFactory = MetalFactory()
    val metalHammer: MetalHammer = metalFactory.createHammer()
    metalHammer.use()
    val metalHacksaw: MetalHacksaw = metalFactory.createHacksaw()
    metalHacksaw.use()

    val woodFactory = WoodFactory()
    val woodenHammer: WoodenHammer = woodFactory.createHammer()
    woodenHammer.use()
    val woodHacksaw: WoodHacksaw = woodFactory.createHacksaw()
    woodHacksaw.use()
}
```
{% endcode %}
