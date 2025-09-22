# Boost.Tokenizer

* lib: `boost/libs/tokenizer`
* repo: `boostorg/tokenizer`
* commit: `8f6316a`, 2025-07-03

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

```c++
struct escaped_list_error : public std::runtime_error { ctor(const std::string& what); };

class escaped_list_separator<Ch,Tr=std::char_traits<Tr>> {
  using string_type = basic_string<Ch,Tr>;
  string_type escape_, c_, quote_;
  bool last_; // state
  bool is_escape(Ch e); bool is_c(Ch e); bool is_quote(Ch e); // whether e is in respective string
  void do_escape<It,Token>(It& next, It end, Token& tok);
public:
  explicit ctor(Ch e='\\', Ch c=',', Ch q='\"');
  ctor(string_type e, string_type c, string_type q);
  void reset();
  bool operator() <It,Token> (It& next, It end, Token& tok); // consume all of [next,end)
};

class offset_separator {
  std::vector<int> offsets_;
  unsigned int current_offset_{0}; // state
  bool wrap_offsets_{true}, return_partial_last_{true};
public:
  ctor() : offsets_(1,1);
  ctor<It> (It b, It e, bool wrap_offsets=true, bool return_partial_last=true);
  void reset();
  bool operator() <It,Token> (It& next, It end, Token& tok); // consume all of [next,end)
};

enum empty_token_policy { drop_empty_tokens, keep_empty_tokens };
class char_separator<Ch,Tr=std::char_traits<Ch>> {
  std::basic_string<Ch,Tr> m_kept_delims, m_dropped_delims;
  bool m_use_ispunct{false}, m_use_isspace{false};
  empty_token_policy m_empty_tokens{drop_empty_tokens};
  bool m_output_done; // state
  bool is_kept(Ch e) const; bool is_dropped(Ch e) const;
public:
  explicit ctor(const Ch* dropped_delims, const Ch* kept_delims =nullptr, empty_token_policy empty_tokens=drop_empty_tokens);
  explicit ctor() : m_use_ispunct{true}, m_use_isspace{true}{};
  void reset();
  bool operator() <It,Token> (It& next, It end, Token& tok); // consume all of [next,end)
};

class char_delimiters_separator<Ch,Tr=std::char_traits<Ch>> {
  std::basic_string<Ch,Tr> returnable_, nonreturnable_;
  bool return_delims_, no_ispunct_, no_isspace_; // no_ispunct = returnable != ""; no_isspace = nonreturnable != ""
  bool is_ret(Ch e) const; bool is_nonret(Ch e) const;
public:
  explicit ctor(bool return_delims=false, const Ch* returnable="", const Ch* nonreturnable="");
  void reset();
  bool operator() <It,Token> (It& next, It end, Token& tok); // consume all of [next,end)
};
```

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
auto make_token_iterator(Iterator begin, Iterator end, const TokenizerFunc&)
  -> token_iterator_generator<TokenizerFunc, Iterator, Type>::type;
```

------
#### Class `tokenizer`

```c++
template <class TokenizerFunc = char_delimiters_separator<char>,
  class Iterator = std::string::const_iterator,
  class Type = std::string
>
class tokenizer {
  using iter = token_iterator_generator<TokenizerFunc,Iterator,Type>::type;
  Iterator first_, last_; TokenizerFunc f_;
public:
  using iterator = iter; using const_iterator = iter;
  using value_type = Type; // [const_]reference and [const_]pointer
  using size_type = void; using difference_type = void;
  ctor(Iterator first, Iterator last, const TOkenizerFunc& f={});
  ctor<Container>(const Container& c, const TokenizerFunc& f={});
  void assign(Iteratir first, Iterator last[, const TokenizerFunc& f]);
  void assign<Container>(const Container& c[, const TokenizerFunc& f]);
  iter begin() const; iter end() const;
};
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>`
* `<boost/iterator/minimum_category.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_pointer.hpp>`
* `<boost/type_traits/conditional.hpp>`

------
### Standard Facilities
