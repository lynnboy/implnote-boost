# Boost.Utility

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### `string_view`

```c++
class basic_string_view<Ch,Tr> {
    const Ch* ptr_; size_t len_;
public: using traits_type = Tr; using value_type = Ch;
    using <const>_pointer = <const> Ch*; using <const>_reference = <const> Ch&;
    using <const>_iterator = <const>_pointer; using <const>_reverse_iterator = std::reverse_iterator<const_iterator>;
    using size_type = size_t; using difference_type = ptrdiff_t;
    static constexpr size_type npos = size_type{-1};

    constepxr ctor() noexcept; constexpr ctor(const self&) noexcept; self& operator=(const self&) noexcept;
    ctor<A>(const std::basic_string<Ch,Tr,A>& str) noexcept;
    constexpr ctor(const Ch* str); constexpr ctor(const Ch* str, size_type len);

    constexpr const_iterator <c>{begin|end} () const noexcept;
    const_reverse_iterator <c>r{begin|end} () const noexcept;
    constexpr size_type {size|length|max_size}() const noexcept;
    constexpr bool empty() const noexcept;
    constexpr const_reference operator[](size_type pos) const noexcept;
    constexpr const_reference at(size_type pos) const;
    constexpr const_reference {front|back} () const;
    constexpr const_pointer data() const noexcept;

    constexpr void clear() noexcept;
    constexpr void remove_{prefix|suffix} (size_type n);
    constexpr void swap(self& s) noexcept;

    explicit operator std::basic_string<Ch,Tr,A> () const;
    std::basic_string<Ch,Tr,A> to_string<A=std::allocator<Ch>(const A& a={}) const;
    size_type copy(Ch* s, size_type n, size_type pos=0) const;
    constexpr self substr() const; constexpr self substr(size_type pos, size_type n=npos) const;
    constexpr int compare({self|const Ch*} x) const noexcept;
    constexpr int compare(size_type pos1, size_type n1, {self|const Ch*} x) const;
    constexpr int compare(size_type pos1, size_type n1, {self|const Ch*} x, size_type pos2, size_type n2) const;
    constexpr bool {starts|ends}_with({Ch|self} c) const noexcept;
    constexpr bool contains({self|Ch|const Ch*} s) const noexcept;
    constexpr size_type find({self|Ch|const Ch*} s, size_type pos=0) const noexcept;
    constexpr size_type find(const Ch* s, size_type pos, size_type n) const noexcept;
    constexpr size_type rfind({self|Ch|const Ch*} s, size_type pos=npos) const noexcept;
    constexpr size_type rfind(const Ch* s, size_type pos, size_type n) const noexcept;
    constexpr size_type find_first_<not>_of({self|Ch|const Ch*} s, size_type pos=0) const noexcept;
    constexpr size_type find_first_<not>_of(const Ch* s, size_type pos, size_type n) const noexcept;
    constexpr size_type find_last_<not>_of({self|Ch|const Ch*} s, size_type pos=npos) const noexcept;
    constexpr size_type find_last_<not>_of(const Ch* s, size_type pos, size_type n) const noexcept;

    friend constexpr bool operator==(self x, self y) noexcept; // and !=, <, >, =, >=
    friend constexpr bool operator== <A> (self x, const std::basic_string<Ch,Tr,A>& y) noexcept; // and !=, <, >, =, >=, and inversed
    friend constexpr bool operator==(self x, const Ch* y) noexcept; // and !=, <, >, =, >=, and inversed
    friend std::basic_ostream<Ch,Tr>& operator<<(std::basic_ostream<Ch,Tr>& os, const self& str);
    friend size_t hash_value(self s);
};

using string_view = basic_string_view<char, std::char_traits<char>>;
using wstring_view = basic_string_view<wchar_t, std::char_traits<wchar_t>>;
using u16string_view = basic_string_view<char16_t, std::char_traits<char16_t>>;
using u32string_view = basic_string_view<char32_t, std::char_traits<char32_t>>;
```

------
### `string_ref`

```c++
class basic_string_ref<Ch,Tr>; // similar to string_ref

using string_ref = basic_string_ref<char, std::char_traits<char>>;
using wstring_ref = basic_string_ref<wchar_t, std::char_traits<wchar_t>>;
using u16string_ref = basic_string_ref<char16_t, std::char_traits<char16_t>>;
using u32string_ref = basic_string_ref<char32_t, std::char_traits<char32_t>>;
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.IO

* `<boost/io/ostream_put.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

------
### Standard Facilities
