---
title: "Design Patterns"
permalink: /golang/patterns/
excerpt: "Design Patterns"
toc: true
sidebar:
  nav: "docs"
---

## Solid Design Principles

...

## Creational Patterns

### Builder

> When piecewise object construction is complicated, provide an API for doing it succinctly.

This design pattern aims to solve the problem of labourious object creation that can result in monstrous constructors or subtype breeding.

#### Example

Imagine you're writing a web server and you want to build up strings of html from ordinary text elements. You could do something like...

```golang
// creating an unordered list
func main() {
    sb := strings.Builder{}
    words := []string{"hello", "world"}
    sb.WriteString("<ul>")
    for _, v := range words {
    	sb.WriteString("<li>")
    	sb.WriteString(v)
    	sb.WriteString("</li>")
    }
    sb.WriteString("</ul")
    fmt.Println(sb.String())
}
```

However this is pretty laborious and not very flexible should you want to output a paragraph or a heading etc.

We want to make it convenient for the client to build HTML elements instead of having to create them piecewise like above.

Luckily, we can represent the whole HTML construct as a tree data structure. We can then delegate the building of said data structure to a Builder component that the client can interact with.

We start with defining the HTML element...

```golang
// htmlElement.go: Product
type HtmlElement struct {
    name, text 	string 	    // name is the tag, text is what's in between
    element     HtmlElement // html elements can also include other html elements
}

// We also want to be able to print the HtmlElement and set the correct 
// indentation etc. We won't show the internals of this because it's not really 
// relevant for the pattern but just for completeness sake...

func (e *HtmlElement) String() string {
    ...
}
```

Now to to create the Builder..

```golang
// htmlBuilder: Concrete builder
type HtmlBuilder struct {
    root HtmlElement // root element is all that's required to return the whole data structure
    rootName string // required to reset the builder (an html element always needs a root)
}

func NewHtmlBuilder(rootName string) *HtmlBuilder {
    return &HtmlBuilder{rootName, 
        HtmlElement{rootName, "", []HtmlElement{}}}
}

// Utility method if we prefer to only interact with the
// Builder and not return the HtmlElement object to the client.
func (b *HtmlBuilder) String() string {
    return b.root.String()
}

func (b *HtmlBuilder) AddChild(childName, childText string) {
    e := HtmlElement{childName, childText,
      []HtmlElement{}}
    b.root.elements = append(b.root.elements, e)
}
```

Refactoring the client code...

```golang
// creating an unordered list
func main() {
    b := NewHtmlBuilder("ul")
    b.AddChild("li", "hello")
    b.AddChild("li", "world")
    fmt.Println(b.String())
}
```

As we can see, we have now successfully decoupled the building of the Html product from the client code. The client only has to care about the utlity calls and not the particulars of Html creation.

#### Fluent Interfaces

A fluent interface allows you to chain calls together. This makes sense in the context of "building" because oftentimes you want to call several methods on the same receiver. We can do this simply by returning the receiver in the method call..

```golang
func (b *HtmlBuilder) AddChildFluent(childName, childText string) *HtmlBuilder {
    e := HtmlElement{childName, childText,
      []HtmlElement{}}
    b.root.elements = append(b.root.elements, e)
}
```

The client code then becomes..

```golang
// creating an unordered list
func main() {
    b := NewHtmlBuilder("ul")
    b.AddChild("li", "hello").
	  AddChild("li", "world")
    fmt.Println(b.String())
}
```

It's not so obvious from the above example but it makes more sense in the context of builder facets.

#### Builder Facets

The faceted builder can be used when you have a very complicated object that requires multiple individual builders to create. The pattern essentially provides a facade over these individual builders so they can be accessed/coordinated in a convenient way.

Imagine we have the following Person type..

```golang
type Person struct {
	// address
	StreetAddress, Postcode, City string

	// job
	CompanyName, Position string
	AnnualIncome int
}
```

Let's start with our "facade" which will be what the client interacts with directly `PersonBuilder`..

```golang
type PersonBuilder struct {
	person *Person
}

func NewPersonBuilder() *PersonBuilder {
	return &PersonBuilder{ &Person{} }
}
```

Now let's create separate builders for each of the different aspects of the `Person` type..

```golang
type PersonAddressBuilder struct {}
type PersonJobBuilder struct {}
```

We want the client not to necessarily know they're accessing underlying builders when they create a Person. That's the point of the facade. To do this we aggregate `PersonBuilder` in these other types.

> Aggregation is a "uses" realtionship. Object A uses Object B, however Object B can still exist meaningully without object A. This differs subtly from compostion which is an "owns" relationship. If Object A "owns" Object B, then once it's deleted, so is  Object B.

```golang
type PersonAddressBuilder struct {
	PersonBuilder
}
type PersonJobBuilder struct {
	PersonBuilder
}
```

Now let's create some utility methods so we can access the various builders from our PersonBuilder.

```golang
func  (b *PersonBuilder) Lives() *PersonAddressBuilder {
	// because PersonAddressBuilder aggregates PersonBuilder we need to provide
	// the initialised object to our constructor.
	return &PersonAddressBuilder{*b}
}

func  (b *PersonBuilder) Works() *PersonJobBuilder {
	return &PersonJobBuilder{*b}
}
```

The above means we can jump around between the various builders with ease.

Let's add the actual builder methods now..

```golang
func (pab *PersonAddressBuilder) At(streetAddress string) *PersonAddressBuilder {
	pab.person.StreetAddress = streetAddress
	return pab
}

func (pab *PersonAddressBuilder) In(city string) *PersonAddressBuilder {
	pab.person.City = city
	return pab
}

func (pab *PersonAddressBuilder) WithPostcode(postcode string) *PersonAddressBuilder {
	pab.person.Postcode = postcode
	return pab
}

func (pjb *PersonJobBuilder) At(companyName string) *PersonJobBuilder {
	pjb.person.CompanyName = companyName
	return pjb
}

func (pjb *PersonJobBuilder) AsA(position string) *PersonJobBuilder {
	pjb.person.Position = position
	return pjb
}

func (pjb *PersonJobBuilder) Earning(annualIncome int) *PersonJobBuilder {
	pjb.person.AnnualIncome = annualIncome
	return pjb
}
```

With the above methods we've effectively setup a tiny DSL (Domain Specific Language) for building up a person's information.

Lastly we need to add some sort of `Build()` method so we can yield the constructed object.

```golang
func (b *PersonBuilder) Build() *Person {
	return b.person
}
```

Now let's actually use all these builders to create a person..

```golang
func main() {
  pb := NewPersonBuilder()
  pb.
    Lives().
    At("123 London Road").
    In("London").
    WithPostcode("SW12BC").
  Works().
    At("Fabrikam").
    AsA("Programmer").
    Earning(123000)
  person := pb.Build()
  fmt.Println(person)
}
```

#### Builder Parameter

> How do you force the client to use your builders?

When you have an API that performs an action on a built object, how do you ensure the client has used your builder to create said object?

We can do this using a **builder parameter**.

As an example, let's say you're building an email API that exposes a `SendEmail(email email)` function. How do you ensure the client uses your `EmailBuilder` to correctly build the email object before passing it to the function? Let's start with..

```golang
package email

// low-level object is exposed
type Email struct {
	from, to, subject, body string
}

// the builder you want the client to use
type EmailBuilder struct {
	email Email
}

// utility method that performs proper validation
func (b *EmailBuilder) From(from string) *EmailBuilder {
	// example validation
	if !strings.Contains(from, "@") {
		panic("email shjould contain @")
	}
	b.email.from = from
	return b
}

// ... and so on for other utility methods

// public API for the client to use
func SendEmail(email Email) {
	// ...
}
```

A client using the above package has to interact with an email object directly in order to pass it to the `SendEmail()` function. They could of course use the provided builder but there's nothing stopping them from initialising an `Email` object directly.

Let's fix this by implementing the builder parameter.

```golang
package email

// let's hide the low-level object so it doesn't bleed out of the pkg
type email struct {
	from, to, subject, body string
}

// expose the builder to the client
type EmailBuilder struct {
	email email
}

// ... utility methods

// hide the underlying function that interacts with the private object
func sendMailImpl(email *email) {
	// ... interaction with email object
}

// this is the builder parameter - essentially just a function that applies to your builder.
type build func(*EmailBuilder)

// public API that your client can interact with to send emails. The client must supply
// a function that takes a pointer to an EmailBuilder as an argument.
func SendEmail(action build) {
	builder := EmailBuilder{}
	action(&builder)
	sendMailImpl(&builder.email)
}
```

The client code becomes..

```golang
package main

func main() {
	SendEmail(func(b *EmailBuilder) {
		b.From("foo@bar.com").
		  To("bar@baz.com").
		  Subject("foobar").
		  Body("Hello, how are you")
	})
}
```

So the idea is that the client **must** provide a function that works on the EmailBuilder to `SendEmail()`, thereby forcing them to initialise an email object correctly.

### Strategy

> Separates an algorithm into its 'skeleton' and concrete implementation steps, which can be varied at run-time.

This pattern leverages interfaces to achieve subtype polymorphism and ultimately LSP (Liskov substitution principle).

#### Example

Outputing a table in markdown and html, the high-level procedure is the same, but the implementation is different.

1. Start by defining the strategy as an interface.

```golang
// typically, the creation of a table involves the below steps. b/c we
// are outputting the table as strings we utilise the inbuilt strings 
// builder to accumulate the text as its being outputted.
type TableStrategy interface {
	Start(builder *strings.Builder, items []string)
	End(builder *strings.Builder)
	AddRow(builder *strings.Builder, item []string)
}
```

2. Implement each strategy

```golang
type MarkdownTableStrategy struct{}

func (m *MarkdownTableStrategy) Start(builder *strings.Builder, items []string) {
	m.AddRow(builder, items)
	for i := 0; i < len(items); i++ {
		builder.WriteString("| " + "--- ")
	}
	builder.WriteString("|\n")
}
func (m *MarkdownTableStrategy) End(builder *strings.Builder) {
}

func (m *MarkdownTableStrategy) AddRow(builder *strings.Builder, items []string) {
	for _, item := range items {
		builder.WriteString("| " + item + " ")
	}
	builder.WriteString("|\n")
}

type HtmlTableStrategy struct{}

func (h *HtmlTableStrategy) Start(builder *strings.Builder, items []string) {
	builder.WriteString("<table>\n")
	builder.WriteString("\t<tr>\n")
	for _, item := range items {
		builder.WriteString("\t\t<th>" + item + "</th>\n")
	}
	builder.WriteString("\t</tr>\n")

}

func (h *HtmlTableStrategy) End(builder *strings.Builder) {
	builder.WriteString("</table>\n")
}

func (h *HtmlTableStrategy) AddRow(builder *strings.Builder, items []string) {
	builder.WriteString("\t<tr>\n")
	for _, item := range items {
		builder.WriteString("\t\t<td>" + item + "</td>\n")
	}
	builder.WriteString("\t</tr>\n")
}
```

3. Create the high-level configurable component which will utilise the strategy.

```golang
type TextProcessor struct {
	builder       strings.Builder
	tableStrategy TableStrategy
}

func NewTextProcessor(tableStrategy TableStrategy) *TextProcessor {
	return &TextProcessor{strings.Builder{}, tableStrategy}
}

func (t *TextProcessor) String() string {
	return t.builder.String()
}
```

4. Implement the high-level algorithm, this is where the LSP shines.

```golang
func (t *TextProcessor) CreateTable(headers []string, rows [][]string) error {
	// should not create table with empty headers
	if len(headers) == 0 {
		return ErrEmptyHeaders
	}
	// length of all rows should be identical
	for _, row := range rows {
		if len(headers) != len(row) {
			return ErrDimensionMismatch
		}
	}
	s := t.tableStrategy
	s.Start(&t.builder, headers)
	for _, row := range rows {
		s.AddRow(&t.builder, row)
	}
	s.End(&t.builder)
	return nil
}
```

#### Summary

- Define an algorithm at a high level.
- Define the interface you expect each strategy to follow.
- Support the injection of the strategy into the high-level algorithm.