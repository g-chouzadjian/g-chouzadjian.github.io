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

Now want to create the Builder...

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

...
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