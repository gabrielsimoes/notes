---
title: Notes on "Simple Made Easy" by Rich Hickey
subtitle: A talk on functional programming ideas by Rich Hickey. Simple != Easy.
date: 2018-05-29
tags:
  - clojure
  - functional-programming
  - talks
---

# Notes on "Simple Made Easy" by Rich Hickey

- We should aim for simplicity because simplicity is a prerequisite for reliability.
- Simple != Easy.
  - "Easy" means "to be at hand", "to be approachable".
  - "Simple" is the opposite of "complex" which means "being intertwined", "being tied together".
  - "Simple" is about doing a single thing, not about doing it only once (cardinality).
- What matters in software is: does the software do what is supposed to do? Is it of high quality? Can we rely on it? Can problems be fixed along the way? Can requirements change over time? The answers to these questions is what matters in writing software not the look and feel of the experience writing the code or the cultural implications of it.
- The benefits of simplicity are: ease of understanding, ease of change, ease of debugging, flexibility.
- Complex constructs: State, Object, Methods, Syntax, Inheritance, Switch/matching, Vars, Imperative loops, Actors, ORM, Conditionals.
- Simple constructs: Values, Functions, Namespaces, Data, Polymorphism, Managed refs, Set functions, Queues, Declarative data manipulation, Rules, Consistency.
- The following constructs are simpler:
  - Values: use final, persistent collections
  - Functions: use stateless methods
  - Namespaces: use a language with good support for namespaces
  - Data: use maps, arrays, sets, XML, JSON, etc.
  - Polymorphism: through protocols, type classes
  - Managed Refs: Clojure, Haskell
  - Set functions: via libraries
  - Queues: via libraries
  - Declarative data manipulation: via SQL, LINQ, Datalog
  - Rules: via libraries or natively in Prolog
- Build simple systems by:
  - Abstracting - design by answering questions related to what, who, when, where, why, and how.
  - Choosing constructs that generate simple artifacts.
  - Simplify by encapsulation.
