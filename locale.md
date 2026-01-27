# Boost.Locale

* lib: `boost/libs/locale`
* repo: `boostorg/locale`
* commit: `3466ac7`, 2025-09-19

------
### Common

```c++
std::string time_zone::global(<const std::string& new_tz>);
```

------
### Locale Categories

#### Convert

```c++
struct conversion_base { enum conversion_type { normalization, upper_case, lower_case, case_folding, title_case }; };
struct converter<Ch> : converter_base, std::locale::facet, facet_id<self> {
    ctor(size_t refs=0);
    virtual std::basic_string<Ch> convert(conversion_type how, const Ch* b, const Ch* e, int flags=0) const =0;
};
enum norm_type { norm_nfd, norm_nfc, norm_nfkd, norm_nfkc, norm_default=norm_nfc };
std::basic_string<Ch> normalize<Ch>(const Ch* b, const Ch* e, norm_type n=norm_default, const std::locale& loc={});
std::basic_string<Ch> normalize<Ch>(const std::basic_string<Ch>& str, norm_type n=norm_default, const std::locale& loc={});
std::basic_string<Ch> normalize<Ch>(const Ch* str, norm_type n=norm_default, const std::locale& loc={});
std::basic_string<Ch> {to_upper,to_lower,to_title,fold_case}<Ch>(const Ch* b, const Ch* e, const std::locale& loc={});
std::basic_string<Ch> {to_upper,to_lower,to_title,fold_case}<Ch>(const std::basic_string<Ch>& str, const std::locale& loc={});
std::basic_string<Ch> {to_upper,to_lower,to_title,fold_case}<Ch>(const Ch* str, const std::locale& loc={});

// ICU impl
void impl_icu::normalize_string(icu::UnicodeString& str, int flags);
struct impl_icu::converter_impl<Ch> : converter<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor(const cdata& d);
    string_type convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: icu::Locale locale_; std::string encoding_;
};
struct impl_icu::get_casemap_size_type<T>;
struct impl_icu::is_casemap_func_const<Func>;
struct impl_icu::raii_casemap<U8Char> {
    using string_type = std::basic_string<U8Char>;
    ctor(const std::string& locale_id); ~dtor(); // no copy
    string_type convert<Conv>(Conv func, const U8Char* b, const U8Char* e) <const>; // cons if is_casemap_func_const<Conv>
private: string_type do_convert<Conv>(Conv func, const U8Char* b, const U8Char* e) const;
    UCaseMap* map_;
};
struct impl_icu::utf8_converter_impl<U8Char> : converter<U8Char> {
    ctor(const cdata& d);
    std::basic_string<U8Char> convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: icu::Locale locale_; raii_casemap<U8Char> map_;
};
std::locale impl_icu::create_convert(const std::locale& in, const cdata& cd, char_facet_t type);

// POSIX impl
struct impl_posix::case_traits<Ch>{ static Ch {lower,upper}(Ch c, locale_t lc); }; // for char, wchar_t
struct impl_posix::std::converter<Ch> : converter<Ch> {
    using string_type = std::basic_string<Ch>; using ctype_type = std::ctype<Ch>;
    ctor(std::shared_ptr<locale_t> lc, size_t refs=0);
    string_type convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: std::shared_ptr<locale_t> lc_;
};
struct impl_posix::utf8_converter<U8Char> : converter<U8Char> {
    ctor(std::shared_ptr<locale_t> lc, size_t refs=0);
    std::basic_string<U8Char> convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: std::shared_ptr<locale_t> lc_;
};
std::locale impl_posix::create_convert(const std::locale& in, std::shared_ptr<locale_t> lc, char_facet_t type);

// STD impl
struct impl_std::converter<Ch> : converter<Ch> {
    using string_type = std::basic_string<Ch>; using ctype_type = std::ctype<Ch>;
    ctor(std::string& locale_name);
    string_type convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: std::locale base_;
};
struct impl_std::utf8_converter<U8Char> : converter<U8Char> {
    using string_type = std::basic_string<U8Char>; using ctype_type = std::ctype<wchar_t>;
    ctor(std::string& locale_name);
    std::basic_string<U8Char> convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: std::locale base_;
};
std::locale impl_std::create_convert(const std::locale& in, std::string& locale_name, char_facet_t type, utf8_support utf);

// Win32 impl
struct impl_win::wide_converter<Ch> : converter<wchar_t> {
    ctor(const winlocale& lc, size_t refs=0);
    string_type convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: winlocale lc_;
};
struct impl_win::utf8_converter<U8Char> : converter<U8Char> {
    ctor(const winlocale& lc, size_t refs=0);
    std::basic_string<U8Char> convert(conversion_type how, const Ch* b, const Ch e, int flags=0) const override;
private: wide_converter cvt_;
};
std::locale impl_win::create_convert(const std::locale& in, const winlocale& lc, char_facet_t type);
```

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

/// ICU impl
struct impl_icu::collate_impl<Ch> : collator<Ch> {
    int level_to_int(collate_level level) const;
    int do_{utf8,ustring,real}_compare(collate_level level, const char* b1,e1,b2,e2, UErrorCode& status) const;
    int do_compare(collate_level level, const char* b1,e1,b2,e2) const override;
    std::vector<uint8_t> do_basic_transform(collate_level level, const Ch* b,e) const;
    std::basic_string<Ch> do_transform(collate_level level, const Ch* b,e) const override;
    long do_hash(collate_level level, const Ch* b,e) const override;
    ctor(const cdata& d);
    icu::Collator& get_collator(collate_level level) const;
private: static constexpr int level_count = identical+1;
    icu_std_converter<Ch> cvt_; icu::Locale locale_;
    mutable thread_specific_ptr<icu::Collator> collates_[level_count];
    bool is_utf8_;
};
std::locale impl_icu::create_collate(const std::locale& in, const cdata& cd, char_facet_t type);

// POSIX impl
struct impl_posix::coll_traits<Ch>;
struct impl_posix::collator<Ch> : std::collate<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor(std::shared_ptr<locale_t> l, size_t refs=0);
    int do_compare(const Ch* lb,le,rb,re) const override;
    long do_hash(const Ch* b,e) const override;
    string_type do_transform(const Ch* b,e) const override;
private: std::shared_ptr<locale_t> lc_;
};
std::locale impl_posix::create_collate(const std::locale& in, std::shared_ptr<locale_t> lc, char_facet_t type);

// STD impl
struct impl_std::utf8_collator_from_wide : std::collate<char> {
    using wfacet = std::collate<wchar_t>;
    ctor(const std::string& locale_name);
    int do_compare(const Ch* lb,le,rb,re) const override;
    long do_hash(const Ch* b,e) const override;
    string_type do_transform(const Ch* b,e) const override;
private: std::locale base_;
};
bool impl_std::collation_works(const std::locale& l);
std::locale impl_std::create_collate(const std::locale& in, const std::string& locale_name, char_facet_t type, utf8_support utf);

// Win32 impl
struct impl_win::utf_collator<Ch> : collator<Ch> {
    explicit ctor(winlocale lc);
    int do_compare(collate_level level, const Ch* lb,le,rb,re) const override;
    long do_hash(collate_level level, const Ch* b,e) const override;
    string_type do_transform(collate_level level, const Ch* b,e) const override;
private: winlocale base_;
};
std::locale impl_win::create_collate(const std::locale& in, const winlocale& lc, char_facet_t type);
```

#### Formatting

```c++
enum flags::display_flags_type {
    posix=0, number, currency, percent, date, time, datetime, strftime, spellout, ordinal, display_flags_mask=31,
    currency_default=0x00, currency_iso=0x20, currency_national=0x40, currency_flags_mask=0x60,
    time_default=0x000, time_short=0x080, time_medium=0x100, time_long=0x180, time_full=0x200, time_flags_mask=0x380,
    date_default=0x0000, date_short=0x0400, date_medium=0x0800, date_long=0x0c00, date_full=0x1000, date_flags_mask=0x1c00,
};
enum flags::pattern_type { datetime_pattern, time_zone_id };
enum flags::value_type { domain_id };

struct ios_info {
    ctor(); ctor(const self&); self& operator=(const self&); ~dtor();
    static self& get(std::ios_base& ios);
    void {display,currency,date,time}_flags(uint64_t flags); uint64_t {display,currency,date,time}_flags() const;
    void domain_id(int); int domain_id() const;
    void time_zone(const std::string&); const std::string& time_zone() const;
    void date_time_pattern<Ch>(const std::basic_string<Ch>& str); std::basic_string<Ch> date_time_pattern<Ch>() const;
    void on_imbue();
private: uint64_t flags_; int domain_id_; std::string time_zone_; any_string datetime_;
};
std::ios_base& as::{posix,number,currency,percent,date,time,datetime,strftime,spellout,ordinal}(std::iso_base& ios);
std::ios_base& as::currency_{default,iso,national}(std::iso_base& ios);
std::ios_base& as::time_{default,short,medium,long,full}(std::iso_base& ios);
std::ios_base& as::date_{default,short,medium,long,full}(std::iso_base& ios);
struct detail::add_ftime<Ch> { std::basic_string<Ch> ftime; void apply(std::basic_ios<Ch>& ios) const; }; // <</>>
add_ftime<Ch> as::ftime<Ch>(const {std::basic_string<Ch>&,Ch*} format);
struct detail::set_timezone{std::string id;}; // <</>>
std::ios_base& as::{gmt,local_time}(std::ios_base& ios);
set_timezone as::time_zone(const {char*,std::string&} id);

// ICU impl
struct impl_icu::base_formatter { virtual ~dtor()=default; };
struct impl_icu::formatter<Ch> : base_formatter {
    using string_type = std::basic_string<Ch>;
    virtual string_type format({double,int32_t,int64_t,uint64_t} value, size_t& code_points) const =0;
    virtual size_t parse(const string_type& str, {double,int32_t,int64_t,uint64_t}& value) const =0;
    static std::unique_ptr<formatter> create(std::ios_base& ios, const icu::Locale& locale, const std::string& encoding);
};
struct impl_icu::number_format<Ch> : formatter<Ch> {
    ctor(icu::NumberFormat& fmt, const std::string& codepage, bool isNumberOnly=false);
    string_type format({double,int32_t,int64_t,uint64_t} value, size_t& code_points) const override;
    size_t parse(const string_type& str, {double,int32_t,int64_t,uint64_t}& value) const override;
private: icu_std_converter<Ch> cvt_; icu::NumberFormat& icu_fmt_; const bool is_NumberOnly_;
};
struct impl_icu::date_format<Ch> : formatter<Ch> {
    ctor({std::unique_ptr<icu::DateFormat>,icu::DateFormat&} fmt, const std::string& encoding);
    string_type format({double,int32_t,int64_t,uint64_t} value, size_t& code_points) const override;
    size_t parse(const string_type& str, {double,int32_t,int64_t,uint64_t}& value) const override;
private: icu_std_converter<Ch> cvt_; std::unique_ptr<icu::DateFormat> icu_fmt_holder_; icu::DateFormat& icu_fmt_;
};
enum class impl_icu::format_len{Short,Medium,Long,Full};
enum class impl_icu::num_fmt_type{number,sci,curr_nat,curr_iso,percent,spell,ordinal};
struct impl_icu::formatters_cache : std::locale::facet {
    static std::locale::id id;
    ctor(const icu::Locale& locale);
    icu::NumberFormat& number_format(num_fmt_type type) const;
    const icu::UnicodeString& {date,time}_format(format_len f) const;
    const icu::UnicodeString& date_time_format(format_len d, format_len t) const;
    const icu::UnicodeString& default_{date,time,date_time}_format() const;
    icu::SimpleDateFormat* date_formatter() const;
private: static constexpr unsigned num_fmt_type_count=ordinal+1, format_len_count=Full+1;
    mutable thread_specific_ptr<icu::NumberFormat> number_format_[num_fmt_type_count];
    mutable thread_specific_ptr<icu::SimpleDateFormat> date_formatter_;
    icu::UnicodeString date_format_[format_len_count], time_format_[format_len_count] date_time_format_[format_len_count][format_len_count],
        default_date_format_, default_time_format_, default_date_time_format_;
    icu::Locale locale_;
};
struct impl_icu::num_format<Ch> : std::num_put<Ch> {
    using string_type = std::basic_string<Ch>; using formater_type = formatter<Ch>;
    ctor(const cdata& d, size_t refs=0);
protected:
    iter_type do_put(iter_type out, std::ios_base& ios, Ch fill, {<unsigned><long>long,<long>double} val) const override;
private: icu::Locale loc_; std::string enc_;
};

std::locale impl_icu::install_formatting_facets<Ch>(const std::locale& in, const cdata& cd);
std::locale impl_icu::create_formatting(const std::locale& in, const cdata& cd, char_facet_t type);
```

#### Parsing

```c++
// ICU impl
struct impl_icu::num_parse<Ch> : std::num_get<Ch> {
    ctor(const cdata& d, size_t refs=0);
protected:
    using string_type = std::basic_string<Ch>; using formater_type = formatter<Ch>; using stream_type = std::basic_istream<Ch>;
    iter_type do_get(iter_type in, iter_type end, std::ios_base& ios, std::ios_base::iostate& err,
        {long,<unsigned>{short,int,<long>long},<long>double,float}& val) const override;
private: icu::Locale loc_; std::string enc_;
};
std::locale impl_icu::install_parsing_facets<Ch>(const std::locale& in, const cdata& cd);
std::locale impl_icu::create_parsing(const std::locale& in, const cdata& cd, char_facet_t type);
```

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
### Details & Utils

```c++
struct detail::facet_id<Derived> { static std::locale::id id; };
struct detail::is_supported_char<Ch>; // true_type for char/wchar_t/char{8,16,32}_t

class detail::any_string {
    struct base { virtual ~dtor()=default; virtual self* clone() const=0; }; // default all other 5 special members in protected
    struct impl<Ch> : base { explicit ctor(basic_string_view<Ch> value); self* clone() const override; std::basic_string<Ch> s; };
    std::unique_ptr<const base> s_;
public: ctor()=default; ctor(const self& other); ctor(self&&)=default; self& operator=(self other);
    void set<Ch>(const basic_string_view<Ch> s); std::basic_string<Ch> get() const;
};

Ch* util::str_end<Ch>(Ch* str);
bool util::is_{upper,lower,numeric}_ascii(const char c);
char util::to_char(unsigned char c);
```

date_time_<facet>, encoding_<errors,utf>, format, generator,
generic_codecvt, gnu_gettext, hold_ptr, info, localization_backend, message, utf, utf8_codecvt, util

detail/: allocator_traits, encoding
util/: locale_data

encoding/: codepage, {i,u,w}conv_converter
icu/: all_generator, cdata, codecvt, date_time, icu_backend, icu_util, time_zone, uconv
posix/: all_generator, codecvt, numeric, posix_backend
shared/: date_time, format<ting>, generator, iconv_codecvt, ids, ios_prop, localization_backend, message, mo_hash, mo_lambda, std_collate_adapter
std/: all_generator, codecvt, numeric, std_backend
util/: codecvt_converter, default_locale, encoding, foreach_char, gregorian, iconv, info, locale_data, make_std_unique, numeric_<conversion>, timezone, win_codepages
win32/: all_generator, api, lcid, numeric, win_backend

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
