---
title: "Searching"
permalink: /algo/searching/
excerpt: "Searching"
toc: true
sidebar:
  nav: "docs"
---

### Linear Search

> Given an array, the simplest way to search for a value is to look at every element in the array and check if it's the value we want.

This is the most basic searching algorithm and, as the name suggests, has an `O(n)` time complexity.

*example*

```golang
// searches a slice of strings
func linearSearch(toSearch []string, matchVal string) int {
	for index, element := range toSearch {
		if element == matchVal {
			return index
		}
	}
	return -1
}
```