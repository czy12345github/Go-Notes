Bad Smell | Refactorings
-- | --
Mysterious Name | Change Function Declaration; Rename Variable; Rename Field
Duplicated Code | Slide Statements then Extract Function; Pull Up Method
Long Function | Extract Function; Replace Temp with Query; Introduce Parameter Object; Preserve Whole Object; [Replace Function with Command](#replace-function-with-command); Decompose Conditional; Replace Conditional with Polymorphism
Long Parameter List | Replace Parameter with Query; Preserve Whole Object; Introduce Parameter Object; [Remove Flag Argument](#remove-flag-argument); Combine Functions into Class
Global Data | Encapsulate Variable
Mutable Data | Encapsulate Variable; Split Variable; Slide Statements then Extract Function; Separate Query from Modifier; Remove Setting Method; [Replace Derived Variable with Query](#replace-derived-variable-with-query); Combine Functions into Class; Combine Functions into Transform; Change Reference to Value
[Divergent Change](#divergent-change) | Split Phase; Move Function; Extract Function; Extract Class
[Shotgun Surgery](#shortgun-surgery) | Move Function; Move Field; Combine Functions into Class; Combine Functions into Transform; Split Phase; Inline Function; Inline Class
Feature Envy | Extract Function; Move Function
Data Clumps | Extract Class; Introduce Parameter Object; Preserve Whole Object
Primitive Obsession | Replace Primitive with Object; Replace Type Code with Subclasses; Replace Conditional with Polymorphism; Extract Class and Introduce Parameter Object
Repeated Switches | Replace Conditional with Polymorphism
[Lazy Element](#laze-element) | Inline Function; Inline Class; Collapse Hierarchy
[Speculative Generality](#speculative-generality) | Inline Function; Inline Class; Collapse Hierarchy; Change Function Declaration; Remove Dead Code
[Temporary Field](#temporary-field) | Extract Class; Move Function; [Introduce Special Case](#introduce-special-case)
Message Chains | Hide Delegate; Extract Function and Move Function
Middle Man | Remove Middle Man; Inline Function; Replace Superclass as Delegate; Replace Subclass with Delegate
Insider Trading | Move Function; Move Field; Hide Delegate; Replace Superclass as Delegate; Replace Subclass with Delegate
Large Class | Extract Class; Extract Superclass; Replace Type Code with Subclasses
Alternative Classes with Different Interfaces | Change Function Declaration; Move Function; Extract Superclass
Data Class | Encapsulate Record; Remove Setting Method; Move Function; Extract Function; Split Phase
Refused Bequest | Push Down Method; Push Down Field; Replace Superclass as Delegate; Replace Subclass with Delegate
Comments | Extract Function; Change Function Declaration; Introduce Assertion

## 坏味道

### Divergent Change

* 当某个条件发生改变时，需要修改一个类中多个不相关的方法。
* 示例：
```go
type OrderRecorder struct {
    order string
    priceList []float64
}
func (o OrderRecorder) Price() float64 {
    values := strings.Split(o.order, " ")
    productId := parseInt(strings.Split(values[0], "-")[1])
    return float64(parseInt(values[1]))*o.priceList[productId]
}
func (o OrderRecorder) ToString() string {
    values := strings.Split(o.order, " ")
    temp := strings.Split(values[0])
    productName, productId := temp[0], temp[1]
    quantity := values[1]
    return fmt.Sprintf("name: %s\tid:%s\tquantity: %s\n", productName, productId, quantity)
}
```
当order格式改变时，比如在最前面附加时间戳信息，Price和ToString方法均需要改变。这是因为这两个方法都进行了解析order的工作，重构方式是将order的解析工作分离出来，让Price和ToString都使用结构化的数据。

### Shortgun Surgery

* 当某个条件发生改变时，需要对多个类做出改变。
* 示例：
```go
type Teacher struct {
    telePhone TelePhone
    ...
}
func (t *Teacher) Format() string {
    contact := fmt.Sprintf("%s-%s", t.telePhone.AreaCode, t.telePhone.Number)
    ...
}
type Student struct {
    phone TelePhone
    ...
}
func (s *Student) Format() string {
    contact := fmt.Sprintf("%s-%s", t.telePhone.AreaCode, t.telePhone.Number)
    ...
}
```
当想要改变电话的输出格式时，要同时修改Teacher和Student。应该为TelePhone添加格式化输出方法供需要格式化输出TelePhone的所有结构体使用。

### Laze Element

* 过于简单的方法或类。
* 示例：
```go
func largerThanFive(x int) bool {
    return x > 5
}
```

### Speculative Generality

* 过度设计，过多地考虑今后将进行的演进而使得当前的程序过于复杂。

### Temporary Field

* 一个结构体中的某个值只有在一些特殊情况下才会被设置。

## 重构方法

### Replace Function with Command

当一个函数过于复杂或需要撤销等附加功能时，可以考虑将其转化为只对外提供一个方法的类。

### Remove Flag Argument

To be a flag argument, the callers must be setting the boolean value to a literal value, not data that's flowing through the program. Also, the implementation function must be using the argument to influence its control flow, not as data that it passes to further functions.

### Replace Derived Variable with Query

对于源数据会改变的情况，如果某个值由对源数据的简单计算得到，那么尽量通过方法获取该值，而不要在结构体中添加该元素。

### Introduce Special Case

用特殊情况的结构体实现普通结构体所实现的接口，Factory Method根据情况决定输出哪种类型的结构体。一般情况下，使用者可以像使用普通结构体一样使用特殊情况下的结构体(Unknow, Null等)；但当不同使用者需要以不同方式处理同一个特殊情况时，应该在结构体中添加判别方法(IsUnknow, IsNull)。

### Remove Flag Argument

### Replace Derived Variable with Query
