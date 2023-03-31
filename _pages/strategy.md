---
title: "Strategy"
permalink: /design/strategy/
excerpt: "Strategy"
toc: true
sidebar:
  nav: "docs"
---

> Separates an algorithm into its 'skeleton' and concrete implementation steps, which can be varied at run-time.

This pattern leverages interfaces to achieve subtype polymorphism and ultimately LSP (Liskov substitution principle).

### Example

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

### Summary

- Define an algorithm at a high level.
- Define the interface you expect each strategy to follow.
- Support the injection of the strategy into the high-level algorithm.