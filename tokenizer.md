# Boost.Tokenizer

* lib: `boost/libs/tokenizer`
* repo: `boostorg/tokenizer`
* commit: `8b3ab1f8`, 2015-5-15

------
### Tokenizer Library

#### Header

* `<boost/token_functions.hpp>`
* `<boost/token_iterator.hpp>`
* `<boost/tokenizer.hpp>`

#### Concept `TokenizerFunction`

```c++
template <class X, class Token, class Iterator>
concept bool TokenizerFunction = Assignable<X> && CopyConstructible<X> &&
  requires (X func, Token tok, Iterator next, Iterator end) {
    { func(next, end, tok) } -> bool;
    func.reset();
  };
```

##### Models of `TokenizerFunction`

* `class escaped_list_separator<Char, Traits=std::char_traits<Char>>`
  * Parameters: `escape='\'`, `comma=','`, `quote='"'`
  * Each parameter can be _one of_ string.
  * When encountered escape error, throw `escaped_list_error`.
* `class offset_separator`
  * Initialized with a range of `offset` integer values.
  * Parameters: `wrap_offsets=true`, `return_partial_last=true`
* `class char_separator<Char, Traits=std::char_traits<Char>>`
  * Parameters: `dropped_delims`, `kept_delims=""`, `empty_tokens=drop_empty_tokens'
  * If default constructed, it use `ispunct` for kept delimiters, and use
    `isspace` for dropped delimiters.
* `class char_delimiters_separator<Char, Traits=std::char_traits<Char>>`
  * Deprecated by `char_separator`.

------
#### Tokenizer Iterator

```c++
template<class TokenizerFunc, class Iterator, class Type>
class token_iterator;

template <class TokenizerFunc = char_delimiters_separator<char>,
  class Iterator = std::string::const_iterator,
  class Type = std::string
>
class token_iterator_generator;

template<class Type, class Iterator, class TokenizerFunc>
make_token_iterator(Iterator begin, Iterator end, const TokenizerFunc&)
  -> token_iterator_generator<TokenizerFunc, Iterator, Type>::type;
```

------
#### Class `tokenizer`

```c++
template <class TokenizerFunc = char_delimiters_separator<char>,
  class Iterator = std::string::const_iterator,
  class Type = std::string
>
class tokenizer;
```

##### Members

* Typedefs like a container, `iterator`, `value_type`, `size_type`, etc.
* `tokenizer(Iterator first, Iterator last, const TokenizerFunc& f = TokenizerFunc())`
* `tokenizer<Container>(const Container&, const TokenizerFunc& f = TokenizerFunc())`
* `assign(Iterator first, Iterator last[, const TokenizerFunc& f])`
* `assign<Container>(const Container &[, const TokenizerFunc& f])`
* `begin() const -> tokenizer_iterator`
* `end() const -> tokenizer_iterator`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>`
* `<boost/iterator/minimum_category.hpp>`

------
### Standard Facilities
