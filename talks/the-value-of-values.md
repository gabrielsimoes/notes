---
title: Notes on "The Value of Values" by Rich Hickey
subtitle: A talk by Rich Hickey about using immutable values over "places".
date: 2018-05-30
tags:
  - clojure
  - functional-programming
  - talks
---

- *Values x Places*
- *Place has no role in an information model*, only in implementation
- Values are:
  - Immutable
  - Don't need methods (to be sent, for example)
- Value advantages:
  - Can be shared freely
  - Reproducible results
    - *Reproduce failures without replicating states*
  - Easy to fabricate (generate compliant values)
  - Refuse to help you program imperatively
  - Language independent
  - Values aggregate to values (value + value = value)
  - more stuff I didn't take note...
