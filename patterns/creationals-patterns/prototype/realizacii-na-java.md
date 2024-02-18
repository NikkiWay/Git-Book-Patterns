---
description: Prototype
---

# Реализации на Java

## Общая реализация на языке Java

{% tabs %}
{% tab title="CandyJava" %}
{% code fullWidth="true" %}
```java
public class CandyJava implements Cloneable {
    private final String name;
    private final String taste;
    private final String manufacture;

    public CandyJava(String name, String taste, String manufacture) {
        this.name = name;
        this.taste = taste;
        this.manufacture = manufacture;
    }

    public String getName() {
        return name;
    }

    public String getTaste() {
        return taste;
    }

    public String getManufacture() {
        return manufacture;
    }

    @Override
    public CandyJava clone()  {
        try {
            return (CandyJava) super.clone();
        } catch (CloneNotSupportedException e) {
            return null;
        }

    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        CandyJava candyJava = (CandyJava) o;
        return Objects.equals(name, candyJava.name) && Objects.equals(taste, candyJava.taste) && Objects.equals(manufacture, candyJava.manufacture);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, taste, manufacture);
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code lineNumbers="true" fullWidth="true" %}
```java
fun main() {
    val myJavaCandy = CandyJava("Kitty","delicious", "Well known")
    val brothersJavaCandy = myJavaCandy.clone()

    println("Is java candies equals: ${myJavaCandy == brothersJavaCandy}")
    println("My java candy: ${System.identityHashCode(myJavaCandy)}")
    println("Brothers java candy: ${System.identityHashCode(brothersJavaCandy)}")
}
```
{% endcode %}
