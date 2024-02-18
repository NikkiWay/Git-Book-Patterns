---
description: Mediator
---

# Реализации на Kotlin

## Общая реализация на языке Kotlin

#### Реализация класса медиатора

{% tabs %}
{% tab title="SmartHomeMediator" %}
{% code fullWidth="true" %}
```kotlin
class SmartHomeMediator {
    private val devices = mutableListOf<SmartHomeDevice>()

    fun add(device: SmartHomeDevice) {
        devices.add(device)
        device.mediator = this
    }

    fun send(message: Message) {
        message.markSent()
        val currentMsgState = message.messageState
        require(currentMsgState is MessageState.Sent)
        println("${message.sender.name} send the message: ${message.content} at time: ${currentMsgState.timeSent}")
        devices
            .filter { smartHomeDevice ->  smartHomeDevice.deviceType == message.recipientType }
            .forEach { smartHomeDevice ->
                message.markReceived()
                smartHomeDevice.receive(message)
            }
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Реализация класса сообщений&#x20;

{% tabs %}
{% tab title="Message" %}
{% code fullWidth="true" %}
```kotlin
class Message(
    val sender: SmartHomeDevice,
    val recipientType: DeviceType,
    val content: String
) {
    private var _messageState: MessageState = MessageState.Initial
    val messageState: MessageState
        get() = _messageState

    fun markSent() {
        val currentState = messageState
        if (currentState is MessageState.Initial) {
            _messageState = MessageState.Sent()
        } else {
            error("Illegal state for sending message: $currentState")
        }
    }

    fun markReceived() {
        val currentState = messageState
        if (currentState is MessageState.Sent) {
            _messageState = MessageState.Received(
                timeSent = currentState.timeSent
            )
        } else {
            error("Illegal state for receiving message: $currentState")
        }
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="MessageState" %}
{% code fullWidth="true" %}
```kotlin
sealed class MessageState {
    data object Initial: MessageState()
    data class Sent(val timeSent: LocalTime = LocalTime.now()): MessageState()
    data class Received(
        val timeSent: LocalTime,
        val timeReceived: LocalTime = LocalTime.now()
    ): MessageState()
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Реализация классов устройств&#x20;

{% tabs %}
{% tab title="SmartHomeDevice" %}
{% code fullWidth="true" %}
```kotlin
abstract class SmartHomeDevice(val name: String) {
    abstract val deviceType: DeviceType
    var mediator: SmartHomeMediator? = null

    abstract fun send(message: Message)
    abstract fun receive(message: Message)
}
```
{% endcode %}
{% endtab %}

{% tab title="SmartLight" %}
{% code fullWidth="true" %}
```kotlin
class SmartLight(name: String) : SmartHomeDevice(name) {
    override val deviceType: DeviceType = DeviceType.LIGHT

    override fun send(message: Message) {
        mediator?.send(message) ?: println("Device $name is not tied to the mediator.")
    }

    override fun receive(message: Message) {
        val currentMsgState = message.messageState
        require(currentMsgState is MessageState.Received)
        println("$name receive the message: ${message.content} at time: ${currentMsgState.timeReceived}")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="SmartCoffeeMachine" %}
{% code fullWidth="true" %}
```kotlin
class SmartCoffeeMachine(name: String) : SmartHomeDevice(name) {
    override val deviceType: DeviceType = DeviceType.COFFEE_MACHINE

    override fun send(message: Message) {
        mediator?.send(message) ?: println("Device $name is not tied to the mediator.")
    }

    override fun receive(message: Message) {
        val currentMsgState = message.messageState
        require(currentMsgState is MessageState.Received) // smart cast to Received
        println("$name receive the message: ${message.content} at time: ${currentMsgState.timeReceived}")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="SmartTv" %}
{% code fullWidth="true" %}
```kotlin
class SmartTv(name: String) : SmartHomeDevice(name) {
    override val deviceType: DeviceType = DeviceType.TV

    override fun send(message: Message) {
        mediator?.send(message) ?: println("Device $name is not tied to the mediator.")
    }

    override fun receive(message: Message) {
        val currentMsgState = message.messageState
        require(currentMsgState is MessageState.Received)
        println("$name receive the message: ${message.content} at time: ${currentMsgState.timeReceived}")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Untitled" %}
{% code fullWidth="true" %}
```kotlin
enum class DeviceType {
    LIGHT,
    COFFEE_MACHINE,
    TV
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Main

{% code lineNumbers="true" fullWidth="true" %}
```kotlin
fun main() {
    val smartCoffeeMachine = SmartCoffeeMachine("Coffee Machine")
    val smartLivingRoomLight = SmartLight("Living Room Light")
    val smartKitchenLight1 = SmartLight("Kitchen Light1")
    val smartKitchenLight2 = SmartLight("Kitchen Light2")
    val smartHallwayLight = SmartLight("Hallway Light")
    val smartTV = SmartTv("Living Room TV")

    println("The owner of the house is back!")
    val houseMediator = SmartHomeMediator()
    houseMediator.add(smartHallwayLight)
    houseMediator.add(smartKitchenLight1)
    houseMediator.add(smartKitchenLight2)
    houseMediator.add(smartLivingRoomLight)
    houseMediator.add(smartCoffeeMachine)
    houseMediator.add(smartTV)

    smartHallwayLight.send(Message(smartHallwayLight, DeviceType.TV, "Set light on in house!"))
    smartKitchenLight1.send(Message(smartKitchenLight1, DeviceType.COFFEE_MACHINE, "Set light on! Let's make coffee"))

    val list = listOf(1, 2, 3)
    list.indexOf(1)
}
```
{% endcode %}
