# Boost.Functional/Forward

* lib: `boost/libs/functional/forward`
* repo: `boostorg/functional`
* commit: `3c468dd`, 2025-06-26

------
### Forwarding Wrappers

Header `<boost/functional/forward_adapter.hpp>` and `<boost/functional/lightweight_forward_adapter.hpp>`

* Handling _perfect forwarding_ problem via preprocessor generated code, superceded by rvalue-reference
* Both adapters wraps a functor/function pointer into a functor
* `lightweight_forward_adapter` requires passing `ref(x)` for lvalues, and unwraps the argument when invoking.
* Handles 0 argument case specially because SFINAE cannot work for that.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.Core

* `<boost/ref.hpp>` - required by `lightweight_forward_adapter`

#### Boost.Utility

* `<boost/utility/result_of.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/iteration/iterate.hpp>`
* `<boost/preprocessor/repetition/*.hpp>`,
* `<boost/preprocessor/cat.hpp>`
* `<boost/preprocessor/arithmetic/dec.hpp>`, `<boost/preprocessor/facilities/intercept.hpp>`

------
### Standard Facilities

Language: Rvalue reference (C++11)
Standard Library: `forward` (C++11)
