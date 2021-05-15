# C++ ABI issue

If nothing else, the ABI will finally sink the Vasa.

- [The Day The Standard Library Died](https://cor3ntin.github.io/posts/abi/)
- ["Remember the Vasa!"](https://www.stroustrup.com/P0977-remember-the-vasa.pdf)

Before that day, C functions returning C valstat structs are eminently usable in adding a value when solving difficult C++ ABI situations. Usually solved with C API in front of the C++ component. Using valstat enabled C API there are no ABI boundaries to a valstat concept.

## Concrete and Immediate

Before "might never happen" problems and solutions, valstat is immediately and concretely usable in many other domains.

As an example, valstat ABI malleability, is useful in [WASM](https://en.wikipedia.org/wiki/WebAssembly), or in shared libraries exhibiting valstat enabled API.
