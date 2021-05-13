# ABI 

Of course, C functions returning C valstat structs are eminently usable in adding a value when solving difficult C++ ABI situations. Usually solved with C API in front of the C++ component. Using valstat enabled C API there are no ABI boundaries to a valstat concept.

Valstat ABI malleability, is useful in [WASM](https://en.wikipedia.org/wiki/WebAssembly), or in shared libraries exhibiting valstat enabled API.
