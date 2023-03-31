---
title: "Visitor"
permalink: /design/visitor/
excerpt: "Visitor"
toc: true
sidebar:
  nav: "docs"
---

> A pattern where a component (visitor) is allowed to traverse the entire hierarchy of types. Implemented by propagating a single `Accept()` method throughout the entire hierachy.

### Motivation

- Need to define a new operation on an entire type hierarchy.
- Do not want to keep modifying every type in the hierarchy.
- Want to have the new functionality separate (SRP).

### Scenario

We want a way to represent numeric expressions i.e.

`(1+2)+3`

Let's define the following interface:

```golang
type Expression interface {}
```

If we break the above expression into it's constituent parts, we see we either have a double-precision expression (the number itself), or we have a binary expression (two double-precision expressions on either side).

We can represent the example expression as these two constructs.

```golang
type DoubleExpression struct {
    value float64
}

type AdditionExpression struct {
    left, right Expression
}
```

Let's create the example expression in our client:

```golang
func main() {
    e := &AdditionExpression{
        left: &DoubleExpression{1},
        right: &AdditionExpression{
            left: &DoubleExpression{2},
            right: &DoubleExpression{3},
        },
    }
}
```

What we've created here is essentially an [abstract syntax tree](https://www.twilio.com/blog/abstract-syntax-trees).

We want to be able to print this data structure so naturally we'll need a way to traverse it.

We can do this using various implementations of the visitor pattern. 

### Intrusive Visitor

*Note: As the name suggests, this pattern implies a violation of the open-closed principal.*

```golang
type Expression interface {
    Print(sb *strings.Builder)
}
...
func (d *DoubleExpression) Print(sb *strings.Builder) {
    sb.WriteString(fmt.Sprintf("%g", d.value))
}
...
func (d *AdditionExpression) Print(sb *strings.Builder) {
    sb.WriteRune('(')
    a.left.Print(sb)
    sb.WriteRune('+')
    a.right.Print(sb)
    sb.WriteRune(')')
}

func main() {
    e := &AdditionExpression{
        left: &DoubleExpression{1},
        right: &AdditionExpression{
            left: &DoubleExpression{2},
            right: &DoubleExpression{3},
        },
    }
    sb := strings.Builder{}
    e.Print(&sb)
    fmt.Println(sb.String())
}
```

```
//output
(1+(2+3))
```

So which component is "Visiting" in this pattern? In this instance it's the `strings.Builder` which is passed as an argument to the `Print()` method.

#### Comments

We mentioned earlier that this is an intrusive pattern - implementing it implies you modify the behavior of the interface and also the elements themselves. In general, this is a bad idea. 

Imagine now you wanted to add another visitor to calculate the final value. Using this pattern, you'd have to modify the expression interface again and then implement each method across the different elements in the hierarchy. This can get nasty.

You could also argue this is a violation of the `Single Responsibility Principal`. It would make much more sense to have a dedicated component which knew how to print every type of expression in your program.

### Reflective Visitor

In this pattern, we split out the Print behavior into it's own function and use reflection to determine what expression we want to print.

```golang
func Print(e Expression, sb *strings.Builder) {
    if de, ok := e.(*DoubleExpression); ok {
        sb.WriteString(fmt.Sprintf("%g", de.value))
    } else if ae, ok := e.(*AdditionExpression); ok {
        sb.WriteRune('(')
        Print(ae.left, sb)
        sb.WriteRune('+')
        Print(ae.right, sb)
        sb.WriteRune(')')
    }
}

func main() {
    e := &AdditionExpression{
        left: &DoubleExpression{1},
        right: &AdditionExpression{
            left: &DoubleExpression{2},
            right: &DoubleExpression{3},
        },
    }
    sb := strings.Builder{}
    Print(e, &sb)
    fmt.Println(sb.String())
}
```

#### Comments

This approach is somewhat better than the intrusive visitor because we adhering to the SRP. If we wanted to extend this behavior to support another type, we wouldn't need to modify the Expression hierachy.

We would however have to break OCP on the `Print()` function to support more types which isn't the best. Remember, we ideally want to *extend* components, not *modify* existing code that has been tested.

### Classic Visitor

#### Note on Dispatch

> dispatch is to do with knowing which function to call

Let's say that to solve our proposed scenario we want to implement individual `Print()` functions that take our concrete Expression types as arguments.

```golang
func Print(e AdditionExpression, sb *strings.Builder) {
    sb.WriteString("(")
    Print(e.left, sb) //Expression
    sb.WriteString("+")
    Print(e.right, sb)
    sb.WriteString(")")
}

func Print(e DoubleExpression, sb *strings.Builder) {
    sb.WriteString(fmt.Sprintf("%g", e.value))
}
```

There are multiple reasons why this won't work.
- Overloading functions are not supported in golang.
- Even if overloading was supported however, you wouldn't be able to do this because of the nested `Print()` statements in the AdditionExpression function. This is because the compiler does not know what the concrete types of `e.left` and `e.right` at compilation time. It only knows they are `Expressions`. So it has no idea which function to call as a result. This is a limitation of *dispatch*.

We can use a method called *double dispatch* to solve this problem. 

> double dispatch is a mechanism whereby you choose the correct method to invoke based both on the receiver and argument types.

#### Implementation

Technically we break OCP here b/c we're modifying the interface, however it is only modified **once**. We don't have to do it multiple times when we add multiple visitors.

```golang
type ExpressionVisitor interface {
    VisitDoubleExpression(e *DoubleExpression)
    VisitAdditionExpression(e *AdditionExpression)
}

type Expression interface {
    Accept(ev ExpressionVisitor) //Our single addition
}

func (d *DoubleExpression) Accept(ev ExpressionVisitor) {
    ev.VisitDoubleExpression(d)
}

func (d *AdditionExpression) Accept(ev ExpressionVisitor) {
    ev.VisitAdditionExpression(d)
}

type ExpressionPrinter struct {
    sb strings.Builder
}

func NewExpressionPrinter() *ExpressionPrinter {
    return &ExpressionPrinter{strings.Builder{}}
}

func (ep *ExpressionPrinter) VisitDoubleExpression(e *DoubleExpression) {
    ep.sb.WriteString(fmt.Sprintf("%g", e.value))
}

func (ep *ExpressionPrinter) VisitAdditionExpression(e *AdditionExpression) {
    ep.sb.WriteRune('(')
    e.left.Accept(ep)
    ep.sb.WriteRune('+')
    e.right.Accept(ep)
    sb.WriteRune(')')
}

func (ep *ExpressionPrinter) String() string {
    retrurn ep.sb.String()
}

func main() {
    e := &AdditionExpression{
        left: &DoubleExpression{1},
        right: &AdditionExpression{
            left: &DoubleExpression{2},
            right: &DoubleExpression{3},
        },
    }
    ep := NewExpressionPrinter()
    e.Accept(ep)
    fmt.Println(ep.String())
}
```

#### Comments

OCP is significantly better with this approach. There may still be some modification required if, for example, you added a `SubstitutionExpression` you'd have to go into the `ExpressionVisitor` interface and add a `VisitSubtractionExpression`. However the advantage of this would be that all your other visitors would be forced to implement this method. You wouldn't be able to forget to implement a visitor like in the previous example.

The addition of new visitors using this pattern is vastly improved and follows OCP as they'd just need to implement the `ExpressionVisitor` interface.
