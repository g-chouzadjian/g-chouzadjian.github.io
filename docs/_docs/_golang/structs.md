---
layout: post
title: Structs
---

## Structs

Struct stands for Data structure. Essentially they represent a collection of related properties or fields. Similar to dictionary in python or object in JS.

### Declaration

```golang
type person struct {
    firstName string
    lastName string
}

func main() {
...
```

### Initialisation

```golang
// There are three ways to initialise a struct:

// 1. Implicit - The fields are filled positionally
func main() {
    garen := person{"Garen", "Chouzadjian"}
}

// 2. Explicit assignment
func main() {
    garen := person{firstName: "Garen", lastName: "Chouzadjian"}
}

// 3. fields inside the struct are 'zero valued'
func main() {
    var garen person
    // to print fields
    fmt.Printf("%+v", garen)
    // to assign to fields
    garen.firstName = "Garen"
}
```

### Zero Valuing

Go doesn't assign Null/Nil to uninitialised fields, it 'zero values' variable like so:

string -> ""
int -> 0
float -> 0
bool -> false
struct -> {}

### Embedding structs

Structs can be embedded within other structs for eg.:

```golang
type contactInfo struct {
  email string
  postCode int
}

// Note in the below declaration you don't need to stipulate a name for the 
// contactInfo field. If you don't include the name it will default to the name 
// of the type

type person struct {
  firstName string
  lastName string
  contact contactInfo # can also just be contactInfo
}

func main() {
  jim := person{
    firstName: "Jim",
    lastName: "Party",
    contact: contactInfo{
        email: "jim@gmail.com",
        postCode: 3103,
    },
}
```

*Note: You **must always** include a trialing ',' when assigning attributes in the above multiline way.*

### Receiver functions

You can use receiver functions on structs in the same way that you use them on other types.

```golang
func (p person) print() {
	fmt.Printf("%+v", p)
}

func main() {
  jim := person{
  ...
  jim.print()
```
