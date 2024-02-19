---
description: Command
---

# Реализации на Kotlin

## UML диаграмма

<figure><img src="../../../.gitbook/assets/command.png" alt=""><figcaption><p>UML диаграмма для общей реализации паттерна "Команда" на Kotlin</p></figcaption></figure>

## Общая реализация на языке Kotlin

{% tabs %}
{% tab title="ICommand" %}
{% code fullWidth="true" %}
```kotlin
fun interface ICommand {
    operator fun invoke()
}
```
{% endcode %}
{% endtab %}

{% tab title="CopyCommand" %}
{% code fullWidth="true" %}
```kotlin
class CopyCommand : ICommand {
    override operator fun invoke() {
        println("Copying document.")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="PrintCommand" %}
{% code fullWidth="true" %}
```kotlin
class PrintCommand : ICommand {
    override operator fun invoke() {
        println("Printing document.")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="ScanCommand" %}
{% code fullWidth="true" %}
```kotlin
class ScanCommand : ICommand {
    override operator fun invoke() {
        println("Scanning document.")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Invoker" %}
{% code fullWidth="true" %}
```kotlin
class Invoker(var command: ICommand) {
    fun execute() {
        println("Executing...")
        command()
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val copyCommand = CopyCommand()
    val scanCommand = ScanCommand()
    val printCommand = PrintCommand()

    val printer = Invoker(copyCommand)
    printer.execute()

    printer.command = scanCommand
    printer.execute()

    printer.command = printCommand
    printer.execute()
}
```
{% endcode %}
