---
description: Facade
---

# Фасад

## Назначение

Паттерн фасад (Facade) относится к структурным шаблонам проектирования и предоставляет унифицированный интерфейс вместо набора интерфейсов некоторой подсистемы. Он определяет интерфейс более высокого уровня, который упрощает использование подсистемы.

## Решаемые задачи

* предоставление простого интерфейса к сложной подсистеме
* ослабление связей между компонентами системы

## Общая реализация на языке С++

Рассмотрим более подробно, как реализовать паттерн фасад для подсистемы компиляции

{% tabs %}
{% tab title="Scanner" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
 class Scanner 
{
public:
    Scanner(istream&);
    virtual ~Scanner();
    virtual Token& Scan();
private:
    istream& _inputStream;
};
```
{% endcode %}
{% endtab %}

{% tab title="Parser" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
 class Parser 
{
public:
    Parser();
    virtual ~Parser();
    virtual void Parse(Scanner&, ProgramNodeBuilder&);
};
```
{% endcode %}
{% endtab %}

{% tab title="ProgramNodeBuilder" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class ProgramNodeBuilder 
{
public:
    ProgramNodeBuilder();
    virtual ProgramNode* NewVariable(const char* variableName) const;
    virtual ProgramNode* NewAssignment(ProgramNode* variable, ProgramNode* expression) const;
    virtual ProgramNode* NewReturnStatement(ProgramNode* value) const;
    virtual ProgramNode* NewCondition(ProgramNode* condition,ProgramNode* truePart, ProgramNode* falsePart) const;
    ProgramNode* GetRootNode();
private:
    ProgramNode* _node;
};
```
{% endcode %}
{% endtab %}

{% tab title="ProgramNode" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class ProgramNode
{
public:
    virtual void GetSourcePosition(int& line, int& index);
    virtual void Add(ProgramNode*);
    virtual void Remove(ProgramNode*);
    virtual void Traverse(CodeGenerator&);
protected:
    ProgramNode();
};
```
{% endcode %}
{% endtab %}

{% tab title="CodeGenerator " %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
class CodeGenerator 
{
public:
    virtual void Visit(StatementNode*);
    virtual void Visit(ExpressionNode*);
protected:
    CodeGenerator(BytecodeStream&);
protected:
    BytecodeStream& _output;
};
```
{% endcode %}
{% endtab %}

{% tab title="Compiler" %}
{% code lineNumbers="true" fullWidth="true" %}
```cpp
// класс Compiler, который будет служить фасадом, позволяющим 
// собрать приведенные классы подсистемы компиляции 

class Compiler 
{
public:
    Compiler();
    virtual void Compile(istream&, BytecodeStream&);
};
void Compiler::Compile (istream& input, BytecodeStream& output) 
{
    Scanner scanner(input);
    ProgramNodeBuilder builder;
    Parser parser;
    parser.Parse(scanner, builder);
    RISCCodeGenerator generator(output);
    ProgramNode* parseTree = builder.GetRootNode();
    parseTree->Traverse(generator);
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Преимущества

* упрощение взаимодействия: предоставление простого интерфейса к сложной подсистеме
* снижение зависимости клиентского кода от подсистемы: работа происходит с одним объектом

## Недостатки

* увеличение сложности и сопровождаемости фасада

Если подсистема имеет большое количество компонентов или сложную структуру, фасад рискует стать сложным, перегруженным объектом, что приведет к укреплению связей между компонентами вместо их ослабления.

## Связь с другими паттернами

* [Абстрактная фабрика](../creationals-patterns/abstract-factory.md): допустимо использовать вместе с фасадом, чтобы предоставить интерфейс для создания объектов подсистем способом, не зависимым от этих подсистем. Абстрактная фабрика может выступать и как альтернатива фасаду, чтобы скрыть платформенно-зависимые классы.
* [Посредник](../behavioral-patterns/opekun.md): аналогичен фасаду в том смысле, что абстрагирует функциональность существующих классов.
* [Одиночка](../creationals-patterns/singleton.md): обычно требуется только один фасад. Поэтому объекты фасадов часто бывают одиночками.
* [Подписчик-издатель](../behavioral-patterns/follower-publisher.md): фасад может применять паттерн наблюдатель для уведомления клиентов о событиях, происходящих в подсистеме.
* [Компоновщик](composite.md): фасад может использовать компоновщик для предоставления единого интерфейса к группе объектов внутри подсистемы.
* [Адаптер](adapter/): фасад может использовать адаптеры для преобразования интерфейсов подсистемы в интерфейс, ожидаемый клиентом
