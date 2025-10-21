# Boost.StaticString

* lib: `boost/libs/static_string`
* repo: `boostorg/static_string`
* commit: `7a53b7f`, 2025-04-17

------

* Header `<boost/static_string.hpp>`

```c++
class basic_static_string<n,Ch,Tr>;
using static_string<n> = basic_static_string<n,char,std::char_traits<char>>;
using static_wstring<n> = basic_static_string<n,wchar_t,std::char_traits<wchar_t>>;
using static_u16string<n> = basic_static_string<n,char16_t,std::char_traits<char16_t>>;
using static_u32string<n> = basic_static_string<n,char32_t,std::char_traits<char32_t>>;
using static_u8string<n> = basic_static_string<n,char8_t,std::char_traits<char8_t>>;

using detail::smallest_width<n> = conditional<n<= std::numeric_limits<unsigned char>::max(), unsigned char, ...>::type; // short, int, long, long long

struct detail::is_nothrow_convertible<From,To>;
struct detail::void_t<...>;
struct detail::is_string_like<T,Ch>;
struct detail::enable_if_viewable<n,T,Ch,Tr>;
using detail::enable_if_viewable_t<n,T,Ch,Tr> = enable_if_viewable<n,T,Ch,Tr>::type;
struct detail::common_string_view_type_impl<T,Ch,Tr>;
using detail::common_string_view_type<T,Ch,Tr> = common_string_view_type_impl<T,Ch,Tr>::type;
struct detail::is_iterator<T>;
struct detail::is_input_iterator<T>;
struct detail::is_forward_iterator<T>;
struct detail::is_subtractable<T>;

size_t detail::distance<FwdIt>(FwdIt first, FwdIt last) requires !is_subtractable<FwdIt>::value
{ size_t dist=0; for (; first!=last; ++first, ++dist) return dist; }
size_t detail::distance<RndIt>(RndIt first, RndIt last) requires is_subtractable<RndIt>::value { return last - first; }
constexpr void detail::copy_with_traits<Tr,InputIt,Ch>(InputIt first, InputIt last, Ch*out) { for (;first!=last;++first,++out) Tr::assign(*out,*first); }

class detail::static_string_base<n,Ch,Tr>{ // constexpr
    using size_type = smallest_width<n>; using value_type = Tr::char_type;
    using pointer = value_type*; using const_pointer = const value_type*;
    size_type size_=0; value_type data_[n+1]{};
public: ctor() noexcept {};
    <const>_pointer data_impl() <const> noexcept { return data_; }
    size_t size_impl() const noexcept { return size_; }  size_t set_size(size_t n) noexcept { return size_ = size_type{n}; }
    void term_impl() noexcept { Tr::assign(data_[size_], value_type{}); }
};
class detail::static_string_base<0,Ch,Tr>{ // constexpr
    using value_type = Tr::char_type; using pointer = value_type*;
    static constexpr const value_type null_{};
public: ctor() noexcept {};
    pointer data_impl() const noexcept { return const_cast<pointer>(&null_); }
    size_t size_impl() const noexcept { return 0; }  size_t set_size(size_t n) noexcept { return 0; }
    void term_impl() noexcept {}
};

int detail::lexicographical_compare<Ch,Tr>(const Ch* s1, size_t n1, const Ch* s2, size_t n2) noexcept {
    if (n1<n2) return Tr::compare(s1, s2, n1)<=0 ? -1:1;
    if (n1>n2) return Tr::compare(s1, s2, n2)>=0 ? 1:-1;
    return Tr::compare(s1,s2,n1);
}
char* detail::integer_to_string<Tr,Int>(char* str_end, Int value, std::true_type) noexcept;
char* detail::integer_to_string<Tr,Int>(char* str_end, Int value, std::false_type) noexcept;
wchar_t* detail::integer_to_wstring<Tr,Int>(wchar_t* str_end, Int value, std::true_type) noexcept;
wchar_t* detail::integer_to_wstring<Tr,Int>(wchar_t* str_end, Int value, std::false_type) noexcept;
static_string<n> detail::to_static_string_int_impl<n,Int>(Int value) noexcept;
static_wstring<n> detail::to_static_wstring_int_impl<n,Int>(Int value) noexcept;
int detail::count_digits(size_t vlaue) { return value<10 ? 1:count_digits(value/10)+1; }
static_string<n> to_static_string_float_impl<n>(<long> double value) noexcept;
static_wstring<n> to_static_wstring_float_impl<n>(<long> double value) noexcept;

FwdIt detail::find_not_of<Tr,Ch,FwdIt>(FwdIt first, FwdIt last, const Ch* str, size_t n) noexcept;
FwdIt1 detail::search<FwdIt1,FwdIt2,BPred>(FwdIt1 first, FwdIt2 last, FwdIt2 s_first, FwdIt2 s_last, BPred p);
InIt detail::find_first_of<InIt,FwdIt,BPred>(InIt first, InIt last, FwdIt s_first, FwdIt s_last, BPred p);
bool detail::ptr_in_range<T>(const T* src_first, const T* src_last, const T* ptr);
[[noreturn]] void detail::throw_exception(const char* msg);

class basic_static_string<n,Ch,Tr=std::char_traits<Ch>> : private static_string_base<n,Ch,Tr> {
public: using traits_type = Tr; using value_type = Tr::char_type;
    using size_type = size_t; using difference_type = ptrdiff_t;
    using <const>_pointer = <const> value_type*; using <const>_reference = <const> value_type&;
    using <const>_iterator = const value_type*; using <const>_reverse_iterator = std::reverse_iterator<<const>_iterator>;
    using string_view_type = basic_string_view<value_type, traits_type>;
    static constexpr size_type static_capacity=n, npos=-1sz;

    ctor() noexcept { term(); }
    ctor(size_type count, value_type ch) { assign(count,ch); }
    ctor<m>(const self<m,Ch,Tr>& other, size_type pos, [size_type count]) { assign(other, pos, [count]); }
    ctor(const_pointer s, [size_type count]) { assign(s, [count]); }
    ctor<InIt>(InIt first, InIt last) requires is_input_iterator<InIt>::value { assign(first, last); }
    ctor<m>(const self<m,Ch,Tr>& other) noexcept { assign(other); } // copy-ctor
    ctor(std::initializer_list<value_type> init) { assign(init.begin(), init.end()); }
    ctor<T>(const T& t, <size_type pos, size_type n>) requires enable_if_viewable_t<n,T,Ch,Tr> { assign(t, <pos, n>); }

    self& operator=<m>(const self<m,Ch,Tr>& s) { return assign(s); } // copy-assign
    self& operator=(const_pointer s) { return assign(s); }
    self& operator=(value_type ch) { return assign_char(ch, bool_constant<(n>0)>{}); }
    self& operator=(std::initializer_list<value_type> ilist) { return assign(ilist); }
    self& operator=<T>(const T& t) requires enable_if_viewable_t<n,T,Ch,Tr> { return assign(t); }
    self& assign(size_type count, value_type ch);
    self& assign<m>(self<m,Ch,Tr> const& s) { return assign_unchecked(s.data(), s.size()); }
    self& assign(self const& s) noexcept { if (data() == s.data()) return *this; return assign_unchecked(s.data(), s.size()); }
    self& assign<m>(self<m,Ch,Tr> const& s, size_type pos, size_type count=npos) { return assign(s.data() + pos, s.capped_length(pos, count)); }
    self& assign(const_pointer s, size_type count);
    self& assign(const_pointer s) { return assign(s, traits_type::length(s)); }
    self& assign<InIt>(InIt first, InIt last) requires is_input_iterator<InIt>::value;
    self& assign(std::initializer_list<value_type> ilist) { return assign(ilist.begin(), ilist.end()); }
    self& assign<T>(const T& t) requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t; return assign(sv.data(), sv.size()); }
    self& assign<T>(const T& t, size_type pos, size_type count=npos) requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t;
        if (pos>sv.size()) throw_exception<std::out_of_range>("pos >= t.size()");
        return assign(sv.data() + pos, std::min(count, sv.size()-pos)); }

    <const>_reference at(size_type pos) <const> { if (pos >= size()) throw_exception<std::out_of_range>("pos >= size()"); return data()[pos]; }
    <const>_reference operator[](size_type pos) <const> noexcept { return data()[pos]; }
    <const>_reference front() <const> noexcept { return data()[0]; }
    <const>_reference back() <const> noexcept { return data()[size()-1]; }
    <const>_pointer data() <const> noexcept { return this->data_impl(); }
    const_pointer c_str() const noexcept { return data(); }

    operator std::basic_string_view<Ch,Tr>() const noexcept { return {data(), size()}; } // and boost::core::string_view

    <const>_iterator <c>begin() <const> noexcept { return data(); }
    <const>_iterator <c>end() <const> noexcept { return data() + size(); }
    <const>_reverse_iterator <c>rbegin() <const> noexcept { return <const>_reverse_iterator{<c>end()}; }
    <const>_reverse_iterator <c>rend() <const> noexcept { return <const>_reverse_iterator{<c>begin()}; }

    bool empty() const noexcept { return size() == 0; }
    size_type size() const noexcept { return this->size_impl(); }
    size_type length() const noexcept { return size(); }
    size_type max_size() const noexcept { return n; }
    void reserve(size_type n) { if (n > max_size()) throw_exception<std::length_error>("n > max_size()"); }
    size_type capacity() const noexcept { return max_size(); }
    void shrink_to_fit() noexcept {}
    void clear() noexcept { this->set_size(0); term(); }

    self& insert(size_type index, size_type count, value_type ch)
    { if (index>size()) throw_exception<std::out_of_range>("index > size()");
        insert(begin() + index, count, ch); return *this; }
    self& insert(size_type index, const_pointer s) { return insert(index, s, traits_type::length(s)); }
    self& insert(size_type index, const_pointer s, size_type count)
    { if (index > size()) throw_exception<std::out_of_range>("index > size()");
        insert(data() + index, s, s + count); return *this; }
    self& insert<m>(size_type index, const self<m,Ch,Tr>& str) { return insert_unchecked(index, str.data(), str.size()); }
    self& insert(size_type index, const self& str) { return insert(index, str.data(), str.size()); }
    self& insert<m>(size_type index, const self<m,Ch,Tr>& str, size_type index_str, size_type count=npos)
    { return insert_unchecked(index, str.data()+index_str, str.capped_length(index_str,count)); }
    self& insert(size_type index, const self& str, size_type index_str, size_type count=npos)
    { return insert(index, str.data()+index_str, str.capped_length(index_str,count)); }
    iterator insert(const_iterator pos, value_type ch) { return insert(pos,1,ch); }
    iterator insert(const_iterator pos, size_type count, value_type ch);
    iterator insert<InIt> (const_iterator pos, InIt first, InIt last) requires is_input_iterator<InIt>::value
    { if constexpr (is_forward_iterator<InIt>::value) {...} else {...} }
    iterator insert(const_iterator pos, std::initializer_list<value_type> ilist) { return insert_unchecked(pos, ilist.begin(), ilist.end()); }
    self& insert(size_type index, const T& t) requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t; return insert(index, sv.data(), sv.size()); }
    self& insert(size_type index, const T& t, size_type index_str, size_type count=npos) requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t;
        if (index_str > sv.size()) throw_exception<std::out_of_range>("index_str > t.size()");
        return insert(index, sv.data(), sv.size()); }

    self& erase(size_type index=0, size_type count=npos) { erase(data() + index, data() + index + capped_length(index, count)); return *this; }
    iterator erase(const_iterator pos) { return erase(pos, pos+1); }
    iterator erase(const_iterator first, const_iterator last);

    void push_back(value_type ch);
    void pop_back() noexcept { this->set_size(size() - 1); term(); }

    self& append(size_type count, value_type ch);
    self& append<m>(const self<m,Ch,Tr>& s) { return append(s.data(), s.size()); }
    self& append<m>(const self<m,Ch,Tr>& s, size_type pos, size_type count=npos) { return append(s.data() + pos, s.capped_length(pos, count)); }
    self& append(const_pointer s, size_type count);
    self& append(const_pointer s) { return append(s, traits_type::length(s)); }
    self& append<InIt>(InIt first, InIt last) requires is_input_iterator<InIt>::value
    { this->set_size(size() + read_back(true, first, last)); return term(); }
    self& append(std::initializer_list<value_type> ilist) { return append(ilist.begin(), ilist.size()); }
    self& append<T>(const T& t) requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t; return append(sv.data(), sv.size()); }
    self& append(const T& t, size_type pos, size_type count=npos) requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t;
        if (pos > sv.size()) throw_exception<std::out_of_range>("pos > t.size()");
        return append(sv.data() + pos, std::min(sv.size() - pos, count)); }
    self& operator+=<m>(const self<m,Ch,Tr>& s) { return append(s); }
    self& operator+=(value_type ch) { push_back(ch); return *this; }
    self& operator+=(const_pointer s) { return append(s); }
    self& operator+=(std::initializer_list<value_type> ilist) { return append(ilist); }
    self& operator+=<T>(const T& t) requires enable_if_viewable_t<n,T,Ch,Tr> { return append(t); }

    int compare<m>(const self<m,Ch,Tr>& s) const noexcept
    { return lexicographical_compare<Ch,Tr>(data(), size(), s.data(), s.size()); }
    int compare<m>(size_type pos1, size_type count1, const self<m,Ch,Tr>& s) const
    { return lexicographical_compare<Ch,Tr>(data() + pos1, capped_length(pos1, count1), s.data(), s.size()); }
    int compare<m>(size_type pos1, size_type count1, const self<m,Ch,Tr>& s, size_type pos2, size_type count2=npos) const
    { return lexicographical_compare<Ch,Tr>(data() + pos1, capped_length(pos1, count1), s.data() + pos2, s.capped_length(pos2, count2)); }
    int compare(const_pointer s) const noexcept
    { return lexicographical_compare<Ch,Tr>(data(), size(), s, traits_type::length(s)); }
    int compare(size_type pos1, size_type count1, const_pointer s) const
    { return lexicographical_compare<Ch,Tr>(data() + pos1, capped_length(pos1, count1), s, traits_type::length(s)); }
    int compare(size_type pos1, size_type count1, const_pointer s, size_type count2) const
    { return lexicographical_compare<Ch,Tr>(data() + pos1, capped_length(pos1, count1), s, count2); }
    int compare<T>(const T& t) const noexcept requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t; return lexicographical_compare<Ch,Tr>(data(), size(), sv.data(), sv.size()); }
    int compare<T>(size_type pos1, size_type count1, const T& t) const requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t;
        return lexicographical_compare<Ch,Tr>(data() + pos1, capped_length(pos1, count1), sv.data(), sv.size()); }
    int compare<T>(size_type pos1, size_type count1, const T& t, size_type pos2, size_type count2=npos) const requires enable_if_viewable_t<n,T,Ch,Tr>
    { common_string_view_type<T,Ch,Tr> sv=t;
        if (pos > sv.size()) throw_exception<std::out_of_range>("pos2 > sv.size()");
        return compare(pos1, count1, sv.data() + pos2, min(sv.size() - pos2, count2)); }
};
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.ContainerHash

* `<boost/container_hash/hash.hpp>`

#### Boost.Core

* `<boost/core/detail/string_view.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Utility

* `<boost/utility/string_view.hpp>`

------
### Standard Facilities
