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

#### Formatting & Parsing

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
struct impl_icu::num_parse<Ch> : std::num_get<Ch> {
    ctor(const cdata& d, size_t refs=0);
protected:
    using string_type = std::basic_string<Ch>; using formater_type = formatter<Ch>; using stream_type = std::basic_istream<Ch>;
    iter_type do_get(iter_type in, iter_type end, std::ios_base& ios, std::ios_base::iostate& err,
        {long,<unsigned>{short,int,<long>long},<long>double,float}& val) const override;
private: icu::Locale loc_; std::string enc_;
};

std::locale impl_icu::install_formatting_facets<Ch>(const std::locale& in, const cdata& cd);
std::locale impl_icu::install_parsing_facets<Ch>(const std::locale& in, const cdata& cd);
std::locale impl_icu::create_formatting(const std::locale& in, const cdata& cd, char_facet_t type);
std::locale impl_icu::create_parsing(const std::locale& in, const cdata& cd, char_facet_t type);

// POSIX impl
struct impl_posix::num_format<Ch> : util::base_num_format<Ch> {
    ctor(std::shared_ptr<locale_t> lc, size_t refs=0);
protected:
    iter_type do_format_currency(bool intl, iter_type out, std::ios_base&, Ch, long double val) const override;
    std::ostreambuf_iterator<{char|wchar_t}> write_it(std::ostreambuf_iterator<{char|wchar_t}> out, const char* ptr, size_t n) const;
private: std::shared_ptr<locale_t> lc_;
};
struct impl_posix::time_put_posix<Ch> : public std::time_put<Ch> {
    ctor(std::shared_ptr<loclae_t> lc, size_t refs=0);
    using string_type = std::basic_string<Ch>;
    iter_type do_put(iter_type out, std::ios_base&, Ch, const std::tm* tm, char format, char modifier) const override;
private: std::shared_ptr<locale_t> lc_;
};

struct impl_posix::ctype_posix<{char|wchar_t}> : public std::ctype<{char|wchar_t}> {
    ctor(std::shared_ptr<locale_t> lc);
    bool do_is(mask m, {char|wchar_t} c) const;
    const {char|wchar_t}* do_is(const {char|wchar_t}* b, e, mask* m) const;
    const {char|wchar_t}* do_scan_{is,not}(mask m, const {char|wchar_t}* b, e) const;
    {char|wchar_t} to{upper,lower}({char|wchar_t} c) const; const {char|wchar_t}* to{upper,lower}(char* b, e) const;
private: std::shared_ptr<locale_t> lc_;
};

struct impl_posix::basic_numpunct { std::string grouping, thousands_sep, decimal_point; ctor(<locale_t> lc); };

struct impl_posix::num_punct_posix<Ch> : std::numpunct<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor(locale_t lc, size_t refs=0);
    void to_str(std::string& s1, std::<w>string& s2, locale_t lc);
    Ch do_decimal_point() const override; Ch do_thou() const override;
    std::string do_grouping() const override;
    string_type do_{true,false}name() const override;
private: string_type decimal_point_, thousands_sep_; std::string grouping_;
};

std::locale impl_posix::create_formatting(const std::locale& in, std::shared_ptr<locale_t> lc, char_facet_t type)
std::locale impl_posix::create_parsing(const std::locale& in, std::shared_ptr<locale_t> lc, char_facet_t type)

// STD impl
struct impl_std::time_put_from_base<Ch> : std::time_put<Ch> {
    ctor(const std::locale& base);
    iter_type do_put(iter_type out, std::iso_base&, Ch fill, const std::tm* tm, char format, char modifier) const override;
private: const std::time_put<Ch>& base_facet_; mutable std::basic_ios<Ch> base_ios_;
};
struct impl_std::utf8_time_put_from_wide : std::time_put<char> {
    ctor(const std::locale& base, size_t refs=0);
    iter_type do_put(iter_type out, std::iso_base&, Ch fill, const std::tm* tm, char format, char modifier) const override;
private: std::locale base_;
};
struct impl_std::utf8_numpunct_from_wide : std::numpunct<char> {
    ctor(const std::locale& base, size_t refs=0);
    char do_{decimal_point,thousands_sep}() const override;
    std::string do_{grouping,{true,false}name}() const override;
private: std::string truename_, false_name; char thousands_sep_, decimal_point_; std::string grouping_;
};
struct impl_std::utf8_moneypunct_from_wide<intl> : std::moneypunct<char,intl> {
    ctor(const std::locale& base, size_t refs=0);
    char do_{decimal_point,thousands_sep}() const override;
    std::string do_{grouping,curr_symbol,{positive,negative}_sign}() const override;
    int do_frac_digits() const override;
    std::money_base::pattern do_{pos,neg}_format() const override;
private: char thousands_sep_, decimal_point_;
    std::string grouping_, curr_symbol_, positive_sign_, negative_sign_;
    int frac_digits; std::money_base::pattern pos_format_, neg_format_;
};
struct impl_std::utf8_numpunct : std::numpunct_byname<char> {
    ctor(const std::string& name, size_t refs=0);
    char do_thousands_sep() const override;
    std::string do_grouping() const override;
};
struct impl_std::utf8_moneypunct<intl> : std::moneypunct_byname<char,intl> {
    ctor(const std::string& name, size_t refs=0);
    char do_thousands_sep() const override;
    std::string do_grouping() const override;
};
std::locale impl_std::create_basic_parsing<Ch>(const std::locale& in, const std::string& locale_name);
std::locale impl_std::create_basic_formatting<Ch>(const std::locale& in, const std::string& locale_name);
std::locale impl_std::create_formatting(const std::locale& in, const std::string& locale_name, char_facet_t type, utf8_support utf);
std::locale impl_std::create_parsing(const std::locale& in, const std::string& locale_name, char_facet_t type, utf8_support utf);

// Win32 impl
struct impl_win::num_format<Ch> : util::base_num_format<Ch> {
    ctor(const winlocale& lc, size_t refs=0);
private: iter_type do_format_currency(bool, iter_type out, std::ios_base& ios, Ch fill, long double val) const override;
    winlocale lc_;
};
struct impl_win::time_put_win<Ch> : std::time_put<Ch> {
    ctor(const winlocale& lc, size_t refs=0);
    using string_type = std::basic_string<Ch>;
    iter_type do_put(iter_type out, std::ios_base&, Ch, const std::tm* tm, char format, char) const override;
private: winlocale lc_;
};
struct impl_win::num_punct_win<Ch> : std::numpunct<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor(const winlocale& lc, size_t refs=0);
    void to_str(std::wstring& s1, std::<w>string& s2);
    Ch do_{decimal_point,thousands_sep}() const override;
    std::string do_grouping() const override;
    string_type do{true,false}name() const override;
private: string_type decimal_point_, thousands_sep_; std::string grouping_;
};
std::locale impl_win::create_formatting(const std::locale& in, const winlocale& lc, char_facet_t type);
std::locale impl_win::create_parsing(const std::locale& in, const winlocale& lc, char_facet_t type);
```

#### Message

```c++
using count_type = long long;
struct message_format<Ch> : std::locale_facet, facet_id<self> {
    using char_type = Ch; using string_type = std::basic_string<Ch>;
    ctor(size_t refs=0);
    virtual const char_type* get(int domain_id, const char_type* context, const char_type* id, <count_type n>) const =0;
    virtual int domain(const std::string& domain) const =0;
    virtual const char_type* convert(const char_type* msg, string_type& buffer) const =0;
};
struct basic_message<Ch> {
    using char_type = Ch; using string_type = std::basic_string<Ch>; using facet_type = message_format<Ch>;
    ctor(); ctor(self {const&|&&})<noexcept> =default; self& operator=(self {const&|&&}) <noexcept(...)> =default;
    explicit ctor(<const char_type* context>, const {char_type*|string_type&} id);
    explicit ctor(<const char_type* context>, const {char_type*|string_type&} single, const {char_type*|string_type&} plural, count_type n);
    void swap(self& other) noexcept(...); friend void swap(self& x, self& y) noexcept(...);
    operator string_type() const; string_type str() const;
    string_type str(<const std::locale& loc>, const std::string& domain_id) const;
    string_type str(const std::locale& loc, <int id>) const;
    void write(std::basic_ostream<char_type>& out) const;
private: count_type n_; const char_type *c_id_, *c_context_, *c_plural_; string_type id_, context_, plural_;
};
using message = basic_message<char>; using wmessage = basic_message<wchar_t>; using u{8,16,32}message = basic_message<char{8,16,32}_t>;
std::basic_ostream<Ch>& operator<< <Ch> (std::basic_ostream<Ch>& out, const basic_message<Ch>& msg);
basic_message<Ch> translate<Ch> (<const Ch* context>, const Ch* msg);
basic_message<Ch> translate<Ch> (<const Ch* context>, const Ch* single, const Ch* plural, count_type n);
basic_message<Ch> translate<Ch> (<const std::basic_string<Ch>& context>, const std::basic_string<Ch>& msg);
basic_message<Ch> translate<Ch> (<const std::basic_string<Ch>& context>, const std::basic_string<Ch>& single, const std::basic_string<Ch>& plural, count_type n);
std::basic_string<Ch> gettext<Ch> (const Ch* id, const std::locale& loc={});
std::basic_string<Ch> ngettext<Ch> (const Ch* s, const Ch* p, count_type n, const std::locale& loc={});
std::basic_string<Ch> dgettext<Ch> (const char* domain, const Ch* id, const std::locale& loc={});
std::basic_string<Ch> dngettext<Ch> (const char* domain, const Ch* s, const Ch* p, count_type n, const std::locale& loc={});
std::basic_string<Ch> pgettext<Ch> (const Ch* context, const Ch* id, const std::locale& loc={});
std::basic_string<Ch> npgettext<Ch> (const Ch* context, const Ch* s, const Ch* p, count_type n, const std::locale& loc={});
std::basic_string<Ch> dpgettext<Ch> (const char* domain, const Ch* context, const Ch* id, const std::locale& loc={});
std::basic_string<Ch> dnpgettext<Ch> (const char* domain, const Ch* context, const Ch* s, const Ch* p, count_type n, const std::locale& loc={});

struct as::detail::set_domain{ std::string domain_id; };
std::basic_ostream<Ch>& as::detail::operator<< <Ch> (std::basic_ostream<Ch>& out, const set_domain& dom);
set_domain as::domain(const std::string& id);

// gettext impl
namespace gnu_gettext {
struct messages_info {
    ctor();
    std::string language, country, variant, encoding, locale_category;
    struct domain { std::string name, encoding; ctor(); ctor(const std::string& n); bool operator{==,!=}(const self& other) const; };
    using domains_type = std::vector<domain>;
    domains_type domains; std::vector<std::string> paths;
    using callback_type = std::function<std::vector<char>(const std::string& file_name, const std::string& encoding)>;
    callback_type callback;
    std::vector<std::string> get_catalog_paths() const;
private: std::vector<std::string> get_lang_folders() const;
};
message_format<Ch>* create_messages_facet<Ch>(const messages_info& info) requires is_supported_char<Ch>;

struct lambda::expr {
    using value_type = long long;
    virtual ~dtor()=default;
    virtual value_type operator()(value_type n) const=0;
};
using lambda::expr_ptr = std::unique_ptr<expr>;
struct lambda::plural_expr {
    ctor()=default; explicit ctor(expr_ptr p);
    value_type operator()(expr::value_type n) const;
    explicit operator bool() const;
private: expr_ptr p_;
};
plural_expr lambda::compile(const char* c_expression);

struct pj_winberger_hash {
    using state_type = uint32_t;
    static constexpr state_type initial_state=0;
    static state_type update_state(state_type value, {char c,const char* p,const char* b,e});
};
pj_winberger_hash::state_type pj_winberger_hash_function(const char* {ptr|b,e});

struct c_file {FILE* handle; ~dtor(){if (handle) fclose(handle);} ctor(const std::string& file_name, const std::string& encoding); };
std::vector<char> read_file(FILE* file);
struct mo_file {
    ctor(std::vector<char> data);
    string_view find(const char* context_in, const char* key_in) const;
    static bool key_equals(const char* real_key, const char* cntx, const char* key);
    const char* key(unsigned id) const;
    string_view value(unsigned id) const;
    bool has_hash() const;
    size_t size() const; bool empty() const;
private: uint32_t keys_offset_, translations_offset_, hash_size_, hash_offset_;
    const std::vector<char> data_; bool native_byteorder_; size_t size_;
};
struct mo_file_use_traits<Ch> {
    static constexpr bool in_use = false; // true for char and char8_t
    using string_view_type = basic_string_view<Ch>;
    static stringview_type use(const mo_file&, const Ch*, const Ch*); // throws by default; work for char and char8_t
};

struct converter<Ch> : conv::utf_encoder<Ch> {ctor(std::string&,std::string in_enc); using base::operator();};
struct converter<char> : conv::narrow_converter> {ctor(std::string& out_enc,std::string in_enc); using base::operator();};

struct message_key<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor(const string_type& c={}); ctor(const Ch* c, const Ch* k);
    bool operator{<,==,!=}(const self& other) const;
    const Ch* context() const; const Ch* key() const;
private: string_type context_, key_; const Ch *c_context_, *c_key_;
};
struct hash_function<Ch> {size_t operator()(const message_key<Ch>& msg) const;};
const Ch* runtime_conversion<Ch>(const Ch* msg, std::basic_string<Ch>& buf, bool do_conv, const std::string& locale_encoding, const std::string& key_encoding);

class mo_message<Ch> : message_format<Ch> {
    using key_type = message_key<Ch>; using catalog_type = std::unordered_map<key_type,string_type,hash_function<Ch>>;
    struct domain_data_type {std::unique_ptr<mo_file> mo_catalog; catalog_type catalog; lambda::plural_expr plural_form;};
public: using string_view_type = mo_file_use_traits<Ch>::string_view_type;
    ctor(const messages_info& info);
    const Ch* get(int domain_id, const Ch* context, const Ch* in_id, <count_type n>) const override;
    int domain(const std::string& domain) const override;
    const Ch* convert(const Ch* msg, string_type& buf) const override;
private: std::map<std::string,unsigned> domains_; std::vector<domain_data_type> domain_data_;
    std::string locale_encodings_, key_encodings_; bool key_conversion_required_;
};
}

std::locale install_message_facet(const std::locale& in, const char_facet_t type, const util::locale_data& data, const std::vector<std::string>& domains,paths);
```

#### Codepage, Encoding

```c++
struct detail::charset_converter<ChIn,ChOut> {
    using char_in_type=ChIn; using char_out_type=ChOut; using string_type=std::basic_string<ChOut>;
    virtual ~dtor()=default;
    virtual string_type convert(const ChIn* b,e) =0;
    string_type convert(const basic_string_view<ChIn> text);
};
using detail::narrow_converter = charset_converter<char,char>;
using detail::utf_encoder<Ch> = charset_converter<char,Ch>;
using detail::utf_decoder<Ch> = charset_converter<Ch,char>;
enum class detail::conv_backend{Default,IConv,ICU,WinAPI};
std::unique_ptr<utf_encoder<Ch>> detail::make_utf_encoder<Ch>(const std::strnig& charset, method_type how, conv_backend impl=Default);
std::unique_ptr<utf_decoder<Ch>> detail::make_utf_decoder<Ch>(const std::strnig& charset, method_type how, conv_backend impl=Default);
std::unique_ptr<narrow_converter> detail::make_narrow_converter(const std::string& src_encoding,target_encoding, method_type how, conv_backend impl=Default);

namespace conv {
struct conversion_error: std::runtime_error {ctor();};
struct invalid_charset: std::runtime_error {ctor();};
enum method_type {skip,stop,default_method=skip};

std::basic_string<Ch1,std::char_traits<Ch1>,A> utf_to_utf<Ch1,Ch2,A=std::allocator<Ch1>,[A2]>
    ({const Ch2* b,e|const Ch2* str|const std::basic_string<Ch2,std::char_traits<Ch2>,{A|A2}>& str}, <method_type how=default_method>, const A& alloc={});

std::basic_string<Ch> :to_utf({const char* b,e|const char* text|const std::string& text}, const std::string& charset, method_type how=default_method);
std::basic_string<Ch> :to_utf({const char* b,e|const char* text|const std::string& text}, const std::locale& loc, method_type how=default_method);
std::string from_utf<Ch>({const Ch* b,e|const std::basic_string<Ch>& text|const Ch* text}, const std::string& charset, method_type how=default_method);
std::string from_utf<Ch>({const Ch* b,e|const std::basic_string<Ch>& text|const Ch* text}, const std::locale& loc, method_type how=default_method);
std::string between({const char* b,e|const char* text|const std::strnig& text}, const std::string& to_encoding,from_encoding, method_type how=default_method);
struct utf_encoder<Ch> {
    using char_type = Ch; using string_type = std::basic_string<Ch>;
    ctor(const std::string& charset, method_type how=default_method);
    string_type convert({const char* b,e|const string_view text}) const; string_type operator()(const string_view text) const;
private: std::unique_ptr<detail::utf_encoder<Ch>> impl_;
};
struct utf_decoder<Ch> {
    using char_type = Ch; using stringview_type = std::basic_string_view<Ch>;
    ctor(const std::string& charset, method_type how=default_method);
    std::string convert({const char* b,e|const stringview_type text}) const; std::string operator()(const stringview_type text) const;
private: std::unique_ptr<detail::utf_decoder<Ch>> impl_;
};
struct narrow_converter {
    ctor(const std::string& src_encoding, target_encoding, method_type how=default_method);
    std::string convert({const char* b,e|const string_view text}) const; std::string operator()(const string_view text) const;
private: std::unique_ptr<detail::narrow_converter> impl_;
};
}

struct generic_codecvt_base { enum initial_conversion_state{to_unicode_state,from_unicode_state}; };
struct generic_codecvt<Ch,CodecvtImpl,charSize=sizeof(Ch)>;
struct generic_codecvt<Ch,CodecvtImpl,{2,4}>; : std::codecvt<Ch,char,std::mbstate_t>, generic_codecvt_base {
    using uchar=Ch;
    ctor(size_t refs=0);
    const CodecvtImpl& implementation() const;
protected:
    std::codecvt_base::result do_unshift(std::mbstate_t& s, char* from,from_end, char*& next) const override;
    int do_{encoding,max_length,always_noconv}() const noexcept override;
    int do_length(std::mbstate_t& std_state, const char* from,from_end, size_t max) const override;
    std::codecvt_base::result do_in(std::mbstate_t& s, const char* from,from_end, const char*& from_next, uchar* to,to_end, uchar*& to_next) const override;
    std::codecvt_base::result do_in(std::mbstate_t& s, const uchar* from,from_end, const uchar*& from_next, char* to,to_end, char*& to_next) const override;
};
struct generic_codecvt<char,CodecvtImpl,1>; : std::codecvt<char,char,std::mbstate_t>, generic_codecvt_base {
    using uchar=char;
    ctor(size_t refs=0);
    const CodecvtImpl& implementation() const;
};

namespace utf {
using code_point = uint32_t;
constexpr code_point illegal=0xFFFFFFFFu, incomplete=0xFFFFFFFEu;
using len_or_error = code_point;
bool is_valid_codepoint(code_point v);

struct utf_traits<Ch,size=sizeof(Ch)>;
struct utf_traits<Ch,1> {
    using char_type=Ch;
    static constexpr int max_width=4;
    static int {trait_length,width}(char_type ci);
    static bool is_{trail,lead}(char_type ci);
    static code_point decode<It>(It& p, It e);
    static code_point decode_valid<It>(It& p);
    static It encode<It>(code_point v, It out);
};
struct utf_traits<Ch,2> {
    using char_type=Ch;
    static bool is_{first,second}_surrogate(uint16_t x);
    static code_point combine_surrogate(uint16_t w1, uint16_t w2);
    static int {trait_length,width}(char_type c);
    static bool is_{trail,lead}(char_type c);
    static constexpr int max_width=2;
    static code_point decode<It>(It& p, It e);
    static code_point decode_valid<It>(It& p);
    static It encode<It>(code_point v, It out);
};
struct utf_traits<Ch,4> {
    using char_type=Ch;
    static int {trail_length,width}(char_type c);
    static bool is_{trail,lead}(char_type c);
    static constexpr int max_width=1;
    static code_point decode<It>(It& p, It e);
    static code_point decode_valid<It>(It& p);
    static It encode<It>(code_point v, It out);
};

struct utf8_codecvt<Ch> : generic_codecvt<Ch,self> {
    struct state_type{};
    ctor(size+t refs=0);
    static int max_encoding_length();
    static state_type initial_state(initial_conversion_state);
    static utf::code_point to_unicode(state_type&, const char*& b, const char* e);
    static utf::len_or_error from_unicode(state_type&, utf::code_point u, char* b, const char* e);
};
}

// ICU impl
struct impl_icu::uconv_converter : util::base_converter {
    ctor(const std::string& encoding);
    bool is_thread_safe() const override;
    self* clone() const override;
    utf::code_point to_unicode(const char*& b, const char* e) override;
    utf::len_or_error from_unicode(utf::code_point u, char* b, const char* e) override;
    int max_len() const override;
private: std::string encoding_; uconv cvt_;
};
std::unique_ptr<util::base_converter> impl_icu::create_uconv_converter(const std::string& encoding);
std::locale impl_icu::create_codecvt(const std::locale& in, const std::string& encoding, char_facet_t type);

struct impl_icu::icu_handle {
    explicit ctor(UConverter* h=nullptr); self& operator=(UConverter* h); ~dtor(); // no copy, allow move
    operator UConverter*() const; explicit operator bool() const;
private: UConverter* h_;
};
struct impl_icu::uconv {
    ctor(const std::string& charset, cpcvt_type cvt_type=skip); // no copy
    int max_char_size() const;
    std::basic_string<U8Char> go<U8Char=char>(const UChar* buf, int length, int max_size) const;
    size_t cut(size_t n, const char* b,e) const;
private: icu_handle cvt_;
};
struct impl_icu::icu_std::converter<Ch,char_size=sizeof(Ch)>;
struct impl_icu::icu_std::converter<Ch,{1,2,4}>{
    using string_type = std::basic_string<Ch>;
    ctor(const std::string& charset, cpcvt_type cvt_type=skip);
    icu::UnicodeString icu_<checked>(const Ch* b,e) const;
    string_type std(const icu::UnicodeString& str) const;
    size_t cut(const icu::UnicodeString& str, const Ch* b,e, size_t n, size_t from_u=0, size_t from_char=0) const;
private:
    uconv cvt_; const int max_len_; // for 1
    cpcvt_type mode_; // for 2 and 4
};

// POSIX impl
std::locale impl_posix::create_codecvt(const std::locale& in, const std::string encoding, char_facet_t type);

// STD impl
std::locale impl_std::codecvt_bychar(const std::locale& in, const std::string& locale_name);
std::locale impl_std::create_codecvt(const std::locale& in, const std::string& locale_name, char_facet_t type, utf8_support utf);
```

#### Calendar

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

struct util::formatting_size_traits<Ch> { static size_t size(const std::basic_string<Ch>& s, const std::locale&); }; // spec for char
struct util::base_num_format<Ch> : std::num_put<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor(size_t refs=0);
protected:
    iter_type do_put(iter_type out, std::ios_base& ios, Ch fill, {<unsigned><long>long,<long>double} val) const override;
};
struct util::base_num_parse<Ch> : std::num_get<Ch> {
    ctor(size_t refs=0);
protected:
    using string_type = std::basic_string<Ch>;
    iter_type do_get(iter_type in, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, {<unsigned><long>long,<long>double} val) const override;
};

std::string util::get_system_locale(bool use_utf8_on_windows=false);
std::locale create_info(const std::locale& in, const std::string& name);
struct util::base_converter {
    static constexpr utf::code_point illegal=illegal, incomplete=incomplete;
    virtual ~dtor();
    virtual int max_len() const{return 1;}
    virtual bool is_thread_safe() const {return false;}
    virtual self* clone() const { return new self{}; }
    virtual utf::code_point to_unicode(const char*& b, const char* e);
    virtual utf::len_or_error from_unicode(utf::code_point u, char* b, const char* e);
};
std::unique_ptr<base_converter> util::create_{utf8,simple}_converter();
std::locale util::create_codecvt(const std::locale& in, std::unique_ptr<base_converter> cvt, char_facet_t type);
std::locale util::create_utf8_codecvt(const std::locale& in, char_facet_t type);
std::locale util::create_simpl_codecvt(const std::locale& in, const std::string& encoding, char_facet_t type);

struct util::utf8_converter : base_converter {
    int max_len() const override;
    self* clone() const override;
    utf::code_point to_unicode(const char*& b, const char* e) override;
    utf::len_or_error from_unicode(utf::code_point u, char* b, const char* e) override;
};
struct util::simple_converter_impl {
    static constexpr int hash_table_size = 1024;
    ctor(const std::string& encoding);
    utf::code_point to_unicode(const char*& b, const char* e) const;
    utf::len_or_error from_unicode(utf::code_point u, char* b, const char* e) const;
private: utf::code_point to_unicode_tbl_[256]; unsigned char from_unicode_tbl_[hash_table_size];
};
struct util::simple_converter : base_converter {
    ctor(const std::string& encoding);
    int max_len() const override {return 1;}
    bool is_thread_safe() const override {return true;}
    base* clone() const override {return new self(*this);}
    utf::code_point to_unicode(const char*& b, const char* e) override;
    utf::len_or_error from_unicode(utf::code_point u, char* b, const char* e) override;
private: simple_converter_impl cvt_;
};

struct util::simple_codecvt<Ch> : public generic_codecvt<Ch,self> {
    ctor(const std::string& encoding, size_t ref=0);
    struct state_type{};
    static state_type initial_state(initial_conversion_state);
    static int max_encoding_length() {return 1;}
    utf::code_point to_unicode(const char*& b, const char* e) const;
    utf::len_or_error from_unicode(utf::code_point u, char* b, const char* e) const;
private: simple_converter_impl cvt_;
};

const char* util::simple_encoding_table[] ={...}; // cp1250..cp1257, iso88591..9,13,15, koi8r, koi8u, usascii, windows1250...1257
std::vector<std::string> util::get_simple_encodings();
bool util::is_simple_encoding(const std::string& encoding);
std::unique_ptr<base_converter> util::create_simple_converter(const std::string& encoding);
std::unique_ptr<base_converter> util::create_utf8_converter();

struct util::code_converter<Ch,threadSafe> : generic_codecvt<Ch,self> {
    using base_converter_ptr = std::unique_ptr<base_converter>; using state_type = base_converter_ptr;
    ctor(base_converter_ptr cvt, size_t refs=0);
    int max_encoding_length() const;
    base_converter_ptr initial_state(initial_conversion_state) const;
    utf::code_point to_unicode(const char*& b, const char* e) const;
    utf::len_or_error from_unicode(utf::code_point u, char* b, const char* e) const;
private: base_converter_ptr cvt_;
};

static std::locale util::do_create_codecvt<Ch>(const std::locale& in, std::unique_ptr<base_converter> cvt);
std::locale util::create_codecvt(const std::locale& in, std::unique_ptr<base_converter> cvt, char_facet_t type);
std::locale util::create_utf8_codecvt(const std::locale& in, char_facet_t type);
std::locale util::create_simple_codecvt(const std::locale& in, const std::string& encoding, char_facet_t type);

struct util::locale_data {
    ctor(); explicit ctor(const std::string& locale_name);
    const std::string& {language,script,country,encoding,variant}() const;
    locale_data& encoding(std::string new_encoding, bool uppercase=true);
    bool is_utf8() const;
    bool parse(const std::string& locale_name);
    std::string to_string() const;
private: std::string language_, script_, country_, encoding_, variant_; bool utf8_;
};

const char* util::utf_name<Ch>();
struct util::is_char8_t<T>;
std::string util::normalize_encoding(string_view encoding);
bool util::are_encodings_equal(const std::string& l,r);
std::vector<std::string> util::get_simple_encodings();
int util::encoding_to_windows_codepage(string_view encoding);

```

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
