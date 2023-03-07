---
title: "Factory"
permalink: /design/factory/
excerpt: "Design Patterns"
toc: true
sidebar:
  nav: "docs"
---

## Factory

> A component responsible solely for the wholesale (not piecewise) creation of object.

### Factory Function

Factory functions are an extremeley basic design pattern and ammount to nothing more than a freestanding function which returns an instance of the object you wish to create.

You see them all over the place when there is some logic that needs to be applied during the creation of an object.

#### Example

In this example we give a default value to the number of eyes a "person" has while leaving the other values configurable in the factory function. You can of course update the `EyeCount` after the factory function.

```golang
type Person {
	Name string
	Age int
	EyeCount int
}

// somewhat more efficient to return a pointer
// instead of returning by-value.
func NewPerson(name string, age int) *Person {
	return &Person{name, age, 2}
}

func main() {
	p := NewPerson("John", 33)
}
```

### Interface Factory

> Factory function returns an interface

This is a neat way of encapsulating some information aout your object and by having your factory function expose the interface that you want your client to work with.

You also gain the ability to return different underlying types.

#### Example

```golang
type Person interface {
	SayHello()
}

type person struct {
	name string
	age int
}

func (p *person) SayHello() {
	fmt.Printf("Hi, my name is %s, I am %d years old\n",
		p.name, p.age)
}

func (p *tiredPerson) SayHello() {
	fmt.Printf("Sorry, I'm too tired.")
}

type tiredPerson struct {
	name string
	age int
}

// factory func returns the interface
func NewPerson(name string, age int) Person {
	// can return different types given build parameters
	if age > 100 {
		return &tiredPerson{name, age}
	}
	return &person{name, age}
}

func main() {
	p := NewPerson("James", 34)
	p.SayHello()
}
```