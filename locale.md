# Boost.Locale

* lib: `boost/libs/locale`
* repo: `boostorg/locale`
* commit: `3466ac7`, 2025-09-19

------
### Locale Categories

#### Convert

#### Collation

```c++
enum class collate_level { primary, secondary, tertiary, quaternary, identical };
struct collator<Ch> : std::locale::facet, facet_id<self> {
    using char_type = Ch; using string_type = std::basic_string<Ch>;
    int compare(<collate_level level=identical>, const char_type* b1, e1, b2, e2) const;
    int compare(collate_level level, const string_type& l, const string_type& r) const;
    string_type transform(<collate_level level=identical>, const char_type* b, e) const;
    string_type transform(collate_level level, const string_type& s) const;
    long hash(<collate_level level=identical>, const char_type* b, e) const;
    long hash(collate_level level, const string_type& s) const;
protected: ctor(size_t refs=0);
    virtual int do_compare(collate_level level, const char_type* b1, e1, b2, e2) const =0;
    virtual string_type do_transform(collate_level level, const char_type* b, e) const =0;
    virtual long do_hash(collate_level level, const char_type* b, e) const =0;
};
struct comparator<Ch,default_level=identical> {
    ctor(const std::locale& l={},collate_level level=default_level);
    bool operator()(const std::basic_string<Ch>& l, r) const;
private: std::locale locale_; const collator<Ch>& collator_; collate_level level_;
};
```

#### Formatting

#### Parsing

#### Codepage

#### Calendar

#### Message

#### Information

#### Boundary

```c++
namespace boundary;

enum boundary_type { character, word, sentence, line };
using rule_type = uint32_t;
constexpr rule_type word_none=0x0000F, word_number=0x000F0, word_letter=0x00F00, word_kana=0x0F000, word_ideo=0xF0000,
    word_any=0xFFFF0, word_letters=0xFFF00, word_kana_ideo=0xFF000, word_mask=0xFFFFF;
constexpr rule_type line_soft=0x0F, line_hard=0xF0, line_any=0xFF, line_mask=0xFF;
constexpr rule_type sentence_term=0x0F, sentence_sep=0xF0, sentence_any=0xFF, sentence_mask=0xFF;
constexpr rule_type character_type=0xF, character_mask=0xF;
rule_type boundary_rule(boundary_type t); // XX -> XX_mask

struct boundary_point<It> {
    using iterator_type = It;
    ctor(); ctor(It p, rule_type r);
    void iterator(It i); It iterator() const;
    void rule(rule_type r); rule_type rule() const;
    bool operator{==,!=}(const {self,It}& other) const;
    operator It() const;
private: It iterator_; rule_type = rule_{0};
};
bool operator{==,!=}<BaseIt> (const BaseIt& l, const boundary_point<BaseIt>& r);
using <w,u8,u16,u32>sboundary_point = boundary_point<std::<w,u8,u16,u32>string::const_iterator>;
using <w,u8,u16,u32>cboundary_point = boundary_point<const {char,wchar_t,char8_t,char16_t,char32_t}*>;

struct break_info { ctor(); ctor(size_t v); size_t offset{0}; rule_type rule{0}; bool operator<(const self& other) const; };
using index_type = std::vector<break_info>;
struct boundary_indexing<Ch> : public std::locale::facet, facet_id<self> {
    ctor(size_t refs=0);
    virtual index_type map(boundary_type t, const Ch* begin, const Ch* end) const =0;
};

int compare_text<LIt,RIt>(LIt l_b, LIt l_e, RIt r_b, RIt r_e);
int compare_text<L,R>(const L& l, const R& r);
int compare_string<L,Ch>(const L& l, const Ch* begin);
int compare_string<R,Ch>(const Ch* begin, const R& r);

struct segment<It> : public std::pair<It,It> {
    using char_type = std::iterator_traits<It>::value_type; using string_type = std::basic_string<char_type>;
    using value_type = char_type; using <const>_iterator = It;
    using difference_type = std::iterator_traits<It>::difference_type;
    ctor(); ctor(iterator b, iterator e, rule_type r);
    void {begin,end}(const iterator& v); It {begin,end}() const;
    operator std::basic_string<char_type,T,A> <T,A> () const;
    string_type str() const; size_t length() const; bool empty() const;
    rule_type rule() const; void rule(rule_type r);
    bool operator{==,!=}(const self& other) const;
private: rule_type rule_;
};
bool operator{==,!=,<,<=,>,>=} <ItL,ItR> (const segment<ItL>& l, const segment<ItR>& r);
bool operator{==,!=,<,<=,>,>=} <Ch,Tr,A,ItR> (const std::basic_string<Ch,Tr,A>& l, const segment<ItR>& r);
bool operator{==,!=,<,<=,>,>=} <It,Ch,Tr,A> (const segment<It>& l, const std::basic_string<Ch,Tr,A>& r);
bool operator{==,!=,<,<=,>,>=} <Ch,ItR> (const Ch* l, const segment<ItR>& r);
bool operator{==,!=,<,<=,>,>=} <It,Ch> (const segment<It>& l, const Ch* r);
using <w,u8,u16,u32>ssegment = segment<std::<w,u8,u16,u32>string::const_iterator>;
using <w,u8,u16,u32>csegment = segment<const {char,wchar_t,char8_t,char16_t,char32_t}*>;
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,It> (std::basic_ostream<Ch,Tr>& out, const segment<It>& seg);

const boundary_indexing<Ch>& detail::get_boundary_indexing<Ch>(const std::locale& l);
struct detail::linear_iterator_traits<Ch,It> {
    static constexpr bool is_linear = ...; // It is <const>Ch*, basic_string<Ch>::<const>_iterator, or vector<Ch>::<const>_iterator
};
struct detail::mapping_traits<It,Cat=std::iterator_traits<It>::iterator_category> {
    using char_type = std::iterator_traits<It>::value_type;
    static index_type map(boundary_type t, It b, It e, const std::locale& l);
};
struct detail::mapping_traits<It,std::random_access_iterator_tag> {
    using char_type = std::iterator_traits<It>::value_type;
    static index_type map(boundary_type t, It b, It e, const std::locale& l);
};
struct detail::mapping<BaseIt> {
    using base_iterator = BaseIt; using char_type = std::iterator_traits<BaseIt>::value_type;
    ctor(boundary_type type, BaseIt b, BaseIt e, const std::locale& loc); ctor();
    const index_type& index() const; BaseIt {begin,end}() const;
private: std::shared_ptr<index_type> index_; BaseIt begin_, end_;
};
struct detail::segment_index_iterator<BaseIt> : iterator_facade<self, segment<BaseIt>,bidirectional_traversal_tag,const segment<BaseIt>&> {
    using base_iterator = BaseIt; using mapping_type = mapping<BaseIt>; using segment_type = segment<BaseIt>;
    ctor();
    ctor(BaseIt p, const mapping_type* map, rule_type mask, bool full_select);
    ctor(bool is_begin, const mapping_type* map, rule_type mask, bool full_select);
    const segment_type& dereference() const;
    bool equal(const self& other) const;
    void increment(); void decrement();
private: segment_type value_; std::pair<size_t,size_t> current_{0,0};
    const mapping_type* map_{nullptr}; rule_type mask_{0}; bool full_select_{false};
};
struct detail::boundary_point_index_iterator<BaseIt> : iterator_facade<self, segment<BaseIt>,bidirectional_traversal_tag,const segment<BaseIt>&> {
    using base_iterator = BaseIt; using mapping_type = mapping<BaseIt>; using boundary_point_type = boundary_point<BaseIt>;
    ctor();
    ctor(BaseIt p, const mapping_type* map, rule_type mask);
    ctor(bool is_begin, const mapping_type* map, rule_type mask);
    const segment_type& dereference() const;
    bool equal(const self& other) const;
    void increment(); void decrement();
private: boundary_point_type value_; size_t current_{0}; const mapping_type* map_{nullptr}; rule_type mask_{0};
};

struct segment_index<BaseIt> {
    using base_iterator = BaseIt; using <const>_iterator = segment_index_iterator<BaseIt>; using value_type = segment<BaseIt>;
    ctor(); ctor(boundary_type type, BaseIt b, BaseIt e, <rule_type mask>, const std::locale& loc={});
    ctor(const boundary_point_index<BaseIt>&); self& operator=(const boundary_point_index<BaseIt>&);
    void map(boundary_type type, BaseIt b, BaseIt e, const std::locale& loc={});
    iterator begin() const; iterator end() const;
    iterator find(BaseIt p) const;
    rule_type rule() const; void rule(rule_type v);
    bool full_select() const; void full_select(bool v);
private: using mapping_type = mapping<BaseIt>;
    mapping_type map_; rule_type mask_{0xFFFFFFFFu}; bool full_select_{false};
};
struct boundary_point_index<BaseIt> {
    using base_iterator = BaseIt; using <const>_iterator = boundary_point_index_iterator<BaseIt>; using value_type = boundary_point<BaseIt>;
    ctor(); ctor(boundary_type type, BaseIt b, BaseIt e, <rule_type mask>, const std::locale& loc={});
    ctor(const segment_index<BaseIt>&); self& operator=(const segment_index<BaseIt>&);
    void map(boundary_type type, BaseIt b, BaseIt e, const std::locale& loc={});
    iterator begin() const; iterator end() const;
    iterator find(BaseIt p) const;
    rule_type rule() const; void rule(rule_type v);
private: using mapping_type = mapping<BaseIt>;
    mapping_type map_; rule_type mask_{0xFFFFFFFFu};
};

using <w,u8,u16,u32>ssegment_index = segment_index<std::<w,u8,u16,u32>string::const_iterator>;
using <w,u8,u16,u32>csegment_index = segment_index<const {char,wchar_t,char8_t,char16_t,char32_t}*>;
using <w,u8,u16,u32>sboundary_point_index = boundary_point_index<std::<w,u8,u16,u32>string::const_iterator>;
using <w,u8,u16,u32>cboundary_point_index = boundary_point_index<const {char,wchar_t,char8_t,char16_t,char32_t}*>;

// ICU implementation
index_type map_direct(boundary_type t, icu::BreakIterator* it, int reverse);
std::unique_ptr<icu::BreakIterator> get_iterator(boundary_type t, const icu::Locale& loc);
index_type do_map<Ch>(boundary_type t, const Ch* b, const Ch* e, const icu::Locale& loc, const std::string& encoding);
struct boundary_indexing_impl<Ch> : boundary_indexing<Ch> {
    ctor(const cdata& data);
    index_type map(boundary_type t, const Ch* b, const Ch* e) const;
private: icu::Locale locale_; std::string encoding_;
};
std::locale create_boundary(const std::locale& in, const cdata& cd, char_facet_t type);
```

------
### Details

```c++
struct detail::facet_id<Derived> { static std::locale::id id; };
struct detail::is_supported_char<Ch>; // true_type for char/wchar_t/char{8,16,32}_t
```

conversion, date_time_<facet>, encoding_<errors,utf>, format<ting>, generator,
generic_codecvt, gnu_gettext, hold_ptr, info, localization_backend, message, time_zone, utf, utf8_codecvt, util

detail/: allocator_traits, any_string, encoding
util/: locale_data, string

encoding/: codepage, {i,u,w}conv_converter
icu/: all_generator, cdata, codecvt, collator, conversion, date_time, formatter<s_cache>, icu_backend, icu_util, numeric, time_zone, uconv
posix/: all_generator, codecvt, collate, converter, numeric, posix_backend
shared/: date_time, format<ting>, generator, iconv_codecvt, ids, ios_prop, localization_backend, message, mo_hash, mo_lambda, std_collate_adapter
std/: all_generator, codecvt, collate, converter, numeric, std_backend
util/: codecvt_converter, default_locale, encoding, foreach_char, gregorian, iconv, info, locale_data, make_std_unique, numeric_<conversion>, timezone, win_codepages
win32/: all_generator, api, collate, converter, lcid, numeric, win_backend


------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.CharConv

* `<boost/charconv/limits.hpp>`
* `<boost/charconv/from_chars.hpp>`
* `<boost/charconv/to_chars.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/config/auto_link.hpp>`

#### Boost.Core

* `<boost/core/detail/string_view.hpp>`
* `<boost/core/exchange.hpp>`
* `<boost/core/ignore_unused.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_facade.hpp>`

#### Boost.Thread

* `<boost/predef/os.h>`

#### Boost.Thread

* `<boost/thread.hpp>`
* `<boost/thread/locks.hpp>`
* `<boost/thread/mutex.hpp>`
* `<boost/thread/tss.hpp>`

------
### Standard Facilities
