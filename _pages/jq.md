---
title: "jq"
permalink: /bash/jq/
excerpt: "jq"
toc: true
sidebar:
  nav: "docs"
---

...

### Filtering Objects

Oftentimes you need to iterate through objects and filter based on some key's value.

Let's take the below JSON:

```json
// fruits.json
{
    "fruits": {
        "lemon": {
            "details": {
                "colour": "yellow"
            },
            "is_citrus": true 
        },
        "banana": {
            "details": {
                "colour": "yellow"
            },
            "is_citrus": false
        },
        "lime": {
            "details": {
                "colour": "green"
            },
            "is_citrus": true
        }
    }
}
```

And extract the colours of all the citrus fruits:

```bash
jq '.fruits | .[] | select(.is_citrus==true) | .details.colour' "./fruits.json"
```

First, we access the fruits property (`.fruits`), which returns all the children of this key. We then pipe the children to the object value iterator (`.[]`), which will iterate through each fruit object. The `select()` function is used to select only the fruits with `.is_citrus==true`. Finally, the `.details.colour` key is queried and the value returned:

```json
"yellow"
"green"
```

