---
description: Proxy
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="HashMapProxy" %}
{% code fullWidth="true" %}
```kotlin
class HashMapProxy<K, V>(
    private val hashMap: MutableMap<K, V>
) : MutableMap<K, V> by hashMap {
    override fun get(key: K): V? {
        val result = hashMap[key]
        if (result == null)
            println("No value under this key found.")
        return result
    }

    override fun put(key: K, value: V): V? {
        val result = hashMap.put(key, value)
        if (result == null)
            println("Value successfully added. No previous values under this key.")
        return result
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
private const val TEST_KEY = "Zero"
private const val TEST_EXCEPTION_KEY = "One"
private const val EXCEPTION_DESCRIPTION = "No such key or value under this key"
private const val TEST_VALUE = 56

fun main() {
    val hashMap = mutableMapOf<String, Int>()
    val proxy: MutableMap<String, Int> = HashMapProxy(hashMap)

    println("Is has map empty: ${proxy.isEmpty()}")
    proxy.put(TEST_KEY, TEST_VALUE)

    try {
        println("Trying to get value: ${proxy.get(TEST_EXCEPTION_KEY)}")
    } catch (e: NullPointerException) {
        println(EXCEPTION_DESCRIPTION)
    }

    println("Is hash map contains key = ${TEST_KEY} : ${proxy.containsKey(TEST_KEY)}")
    println("Is hash map contains value = $TEST_VALUE : ${proxy.containsValue(TEST_VALUE)}")
}
```
{% endcode %}
