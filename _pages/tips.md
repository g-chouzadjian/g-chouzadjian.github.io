---
title: "Tips"
permalink: /bash/tips/
excerpt: "Tips"
toc: true
sidebar:
  nav: "docs"
---

## How to load a template file and populate with vars

```
eval "cat << EOF > ${OUT_FILE}
  $(<"${IN_FILE}")
  " 2> /dev/null
```