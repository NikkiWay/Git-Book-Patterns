---
description: Facade
---

# Реализации на С++

## Пример реализации паттерна фасад для подсистемы компиляции на языке С++

{% tabs %}
{% tab title="Scanner" %}
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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

{% tab title="CodeGenerator" %}
{% code fullWidth="true" %}
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
{% code fullWidth="true" %}
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
