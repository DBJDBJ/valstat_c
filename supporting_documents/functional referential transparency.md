# Functional referential transparency

## What is FRT?

`f(42)` in purely functional code, can be evaluated once, and the value it returns can be replaced for every other instance of `f(42)` in the same program. 

This has been done before and elsewhere, but not uniformly across many languages, on the foundations of one unifying core idea.

## Valstat and FRT

valstat adoption makes C functions with no side effects easier to achieve. Using valstat allows for much cleaner C functional programming.

Honourable readership of this text is of course well aware, in pure functional languages functions cannot have side effects. That is, they are completely determined by what they return, and cannot do anything else. Most notably they cannot throw exceptions, or set globals. 

