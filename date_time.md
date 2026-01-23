# Boost.DateTime

* lib: `boost/libs/date_time`
* repo: `boostorg/date_time`
* commit: `69c45b7`, 2025-07-07

------
### Common Date Time

#### Common Definitions

```c++
enum weekdays {Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday};
enum months_of_year {Jan=1,Feb,Mar,Apr,May,Jun,Jul,Aug,Sep,Oct,Nov,Dec,NotAMonth,NumMonths};
enum special_values {not_a_date_time, neg_infin, pos_infin, min_date_time,  max_date_time, not_special, NumSpecialValues};
enum time_resolutions {sec,tenth,hundredth,milli,ten_thousandth,micro,nano,NumResolutions};
enum dst_flags {not_dst, is_dst, calculate};

struct period<PointRep,DurationRep> : private less_than_comparable<self, equality_comparable<self>> {
    using point_type = PointRep; using duration_type = DurationRep;
    ctor(PointRep first_point, PointRep end_point); ctor(PointRep first_point, DurationRep len);
    PointRep begin() const, end() const, last() const; DurationRep length() const;
    bool is_null() const;
    bool operator{==,<}(const self& rhs) const;
    void shift(const DurationRep& d); void expand(const DurationRep& d);
    bool contains(const PointRep& point) const; bool contains(const self& other) const;
    bool intersects(const self& other) const;
    bool is_adjacent(const self& other) const; bool is_{before,after}(const PointRep& point) const;
    self intersection(const sefl& other) const;
    self merge(const self& other); self span(const self& other) const;
private: PointRep begin_, last_;
};

struct year_month_day_base<Y,M,D> {
    using year_type = Y; using month_type = M; using day_type = D;
    ctor(Y year, M month, D day);
    Y year; M month; D day;
};
struct date<T,Calendar,Duration> : private less_than_comparable<T,equality_comparable<T>> {
    using date_type = T; using calendar_type = Calendar; using duration_type = Duration;
    using traits_type = Calendar::date_traits_type; // and {year,month,day,ymd,day_of_week}_type; date_{rep,int}_type
    ctor(year_type y, month_type m, day_type d); ctor(const ymd_type& ymd);
    year_type year() const; month_type month() const; day_type day() const; day_of_week_type day_of_week() const; ymd_type year_month_day() const;
    bool operator{<|==}(const self& rhs) const;
    bool is_special() const; bool is_not_a_date() const; bool is_<pos,neg>_infinity() const;
    special_values as_special() const;
    duration_type operator-(const date_type& d) const;
    date_type operator{-,+}(const duration_type& dd) const; date_type operator{-,+}=(const duration_type& dd);
protected: explicit ctor(date_{int,rep}_type days);
    date_int_type days_;
};

struct base_time<T,TimeSystem> : private less_than_comparable<T,equality_comparable<T>> {
    using _is_boost_date_time_point = void;
    using time_type = T;
    using time_rep_type = TimeSystem::time_rep_type; // and date_type, date_duration_type, time_duration_type;
    ctor(const date_type& day, const time_duration_type& td, dst_flags dst=not_dst);
    ctor(special_values sv);
    ctor(const self& rhs);
    date_type date() const; time_duration_type time_of_day() const;
    std::string zone_name(bool=false)const; std::string zone_abbrev(bool=false) const; std::string zone_as_posix_string()const;
    bool is_not_a_date_time() const; bool is_<pos,neg>_infinity() const; bool is_special() const;
    bool operator{==,<}(const time_type& rhs) const;
    time_duration_type operator-(const time_type& rhs) const;
    time_type operator{+,-}(const {date,time}_duration_type& dd) const; time_type operator{+,-}=(const {date,time}_duration_type& dd);
protected: time_rep_type time_;
};

struct date_duration<DurationRepTraits> : private less_than_comparable<self,equality_comparable<self,addable1<self,subtractable1,self,dividable2<self,int>>>> {
    using duratino_rep_type = DurationRepTraits::int_type; using duration_rep = DurationRepTraits::impl_type;
    explicit ctor(duration_rep day_count); ctor(special_values sv);
    duration_rep get_rep() const;
    special_values as_special() const; bool is_special() const;
    duration_rep_type days() const;
    static date_duration unit();
    bool operator{==,<}(const self& rhs) const;
    self& operator{-,+}=(const self& rhs);
    self operator-() const;
    self& operator/=(int d);
    bool is_negative() const;
private: duration_rep days_;
};

struct duration_traits_long{using int_type=long; using impl_type=long; static int_type as_number(impl_type i);};
struct duration_traits_adapted{using int_type=long; using impl_type=int_adapter<long>; static int_type as_number(impl_type i);};

struct time_duration<T,RepType> : private less_than_comparable<T,equality_comparable<T>> {
    using _is_boost_date_time_duration=void;
    using duration_type = T; using traits_type = RepType;
    using day_type = RepType::day_type; // and {hour,min,sec,fractional_seconds,tick,impl}_type
    ctor();
    ctor(hour_type hours_in, min_type minutes_in, sec_type seconds_in=0, fractional_seconds_type frac_sec_in=0);
    ctor(special_values sv);
    static duration_type unit(); static tick_type ticks_per_second(); static time_resolutions resolution();
    hour_type hours() const; min_type minutes() const; sec_type seconds() const;
    sec_type total_seconds() const; tick_type total_milliseconds() const; tick_type total_nanoseconds() const; tick_type total_microseconds() const;
    fractional_seconds_type fractional_seconds() const;
    static unsigned short num_fractional_digits();
    duration_type invert_sign() const; duration_type abs() const;
    bool is_negative() const; bool is_zero() const; bool is_positive() const;
    bool operator{<,==}(const time_duration& rhs) const;
    duration_type operator-() const;
    duration_type operator{-,+}(const duration_type& d) const;
    duration_type operator{*,/}(int v) const;
    duration_type operator{-,+}=(const duration_type& d);
    duration_type operator{*,/}=(int v);
    tick_type ticks() const;
    bool is_special() const; bool is_{pos,neg}_infinity() const; bool is_not_a_date_time() const;
    impl_type get_rep() const;
protected: explicit ctor(impl_type in);
    impl_type ticks_;
};
struct subsecond_duration<BaseDuration,frac_of_seconds> : BaseDuration {
    explicit ctor<T>(T const& ss) requires integer<T>;
};

struct weeks_duration<Config> : date_duration<Config> { ctor(Config::impl_type w); ctor(special_values sv); };
struct months_duration<Config> {
private:
    using int_rep=Config::int_rep; using int_type = int_rep::int_type;
    using date_type=Config::date_type; using duration_type = date_type::duration_type;
    using month_adjustor_type = Config::month_adjustor_type;
    using months_type = self; using years_type = years_duration<Config>;
public: ctor(int_rep num); ctor(special_values sv);
    int_rep number_of_months() const;
    duration_type get_<neg>_offset(const date_type& d) const;
    bool operator{==,!=}(const self& rhs) const;
    self operator{+,-}(const {months,years}_type& rhs) const; self& operator{+,-}=(const {months,years}_type& rhs);
    self operator{*,/}(int rhs) const; self& operator{*,/}=(int rhs);
    friend date_type operator{+,-}(const date_type& d, const self& m); friend date_type operator{+,-}=(date_type& d, const self& m);
private: int_rep _m;
};
struct years_duration<Config> {
private:
    using int_rep=Config::int_rep; using int_type = int_rep::int_type;
    using date_type=Config::date_type; using duration_type = date_type::duration_type;
    using month_adjustor_type = Config::month_adjustor_type;
    using months_type = months_duration<Config>; using years_type = self;
public: ctor(int_rep num); ctor(special_values sv);
    int_rep number_of_years() const;
    duration_type get_<neg>_offset(const date_type& d) const;
    bool operator{==,!=}(const self& rhs) const;
    self operator{+,-}(const years_type& rhs) const; self& operator{+,-}=(const years_type& rhs);
    self operator{*,/}(int rhs) const; self& operator{*,/}=(int rhs);
    month_type operator{+,-}(const month_type& m) const;
    friend date_type operator{+,-}(const date_type& d, const self& m); friend date_type operator{+,-}=(date_type& d, const self& m);
private: int_rep _y;
};

struct c_time {
    static std::tm* localtime(const std::time_t* t, std::tm* result);
    static std::tm* gmtime(const std::time_t* t, std::tm* result);
};

struct day_clock<DateType> {
    using ymd_type = DateType::ymd_type;
    static DateType local_day(), universal_day();
    static ymd_type local_day_ymd(), universal_day_ymd();
};
```

#### Adjust & Functions

```c++
struct day_functor<DateType> {
    using duration_type = DateType::duration_type;
    ctor(int f);
    duration_type get_<neg>_offset(const date_type&) const;
private: int f_;
};

struct month_functor<DateType> {
    using duration_type = DateType::duration_type; using cal_type = DateType::calendar_type;
    using ymd_type = cal_type::ymd_type; using day_type = cal_type::day_type;
    ctor(int f);
    duration_type get_<neg>_offset(const date_type&) const;
private: int f_; mutable short origDayOfMonth_;
};

struct week_functor<DateType> {
    using duration_type = DateType::duration_type; using cal_type = DateType::calendar_type;
    ctor(int f);
    duration_type get_<neg>_offset(const date_type&) const;
private: int f_;
};

struct year_functor<DateType> {
    using duration_type = DateType::duration_type;
    ctor(int f);
    duration_type get_<neg>_offset(const date_type&) const;
private: month_functor<date_type> _mf;
};

struct c_local_adjustor<TimeType> {
    using time_duration_type = TimeType::time_duration_type;
    using date_type = TimeType::date_type; using date_duration_type = date_type::duration_type;
    static TimeType utc_to_local(const time_type& t);
};

struct year_based_generator<DateType> {
    using calendar_type = DateType::calendar_type; using year_type = calendar_type::year_type;
    ctor(); virtual ~dtor();
    virtual DateType get_date(year_type y) const =0;
    virtual std::string to_string() const =0;
};

struct partial_date<DateType> : public year_based_generator<DateType> {
    using day_type = calendar_type::day_type; using month_type = calendar_type::month_type;
    using duration_type = DateType::duration_type; using duration_rep = duration_type::duration_rep;
    ctor(day_type d, month_type m); ctor(duration_rep days);
    date_type get_date(year_type y) const override; date_type operator()(year_type y) const;
    bool operator{==,<}(const self& rhs) const;
    month_type month() const; day_type day() const;
    std::string to_string() const override;
private: day_type day_; month_type month_;
};

const char* nth_as_str(int ele);
struct nth_kday_of_month<DateType> : public year_based_generator<DateType> {
    using day_of_week_type = calendar_type::day_of_week_type; using month_type = calendar_type::month_type;
    using duration_type = DateType::duration_type;
    ctor(week_num week_no, day_of_week_type dow, month_type m);
    date_type get_date(year_type y) const override;
    month_type month() const; week_num nth_week() const; day_of_week_type day_of_week() const;
    const char* nth_week_as_str() const; std::string to_string() const override;
private: month_type month_; week_num wn_; day_of_week_type dow_;
};

struct {first,last}_kday_of_month<DateType> : public year_based_generator<DateType> {
    using day_of_week_type = calendar_type::day_of_week_type; using month_type = calendar_type::month_type;
    using duration_type = DateType::duration_type;
    ctor(day_of_week_type dow, month_type m);
    date_type get_date(year_type y) const override;
    month_type month() const; day_of_week_type day_of_week() const;
    std::string to_string() const override;
private: month_type month_; day_of_week_type dow_;
};

struct first_kday_{after,before}<DateType> {
    using calendar_type = DateType::calendar_type; using day_of_week_type = calendar_type::day_of_week_type;
    using duration_type = DateType::duration_type;
    ctor(day_of_week_type dow);
    date_type get_date(year_type y) const;
    day_of_week_type day_of_week() const;
private: day_of_week_type dow_;
};

DateType::duration_type days_{until,before}_weekday <DateType,WeekdayType> (const DateType& d, const WeekdayType& wd);
DateType {next,previous}_weekday <DateType,WeekdayType> (const DateType& d, const WeekdayType& wd);

struct date_itr_base<DateType> {
    using duration_type = DateType::duration_type; using value_type = DateType;
    using iterator_category = std::input_iterator_tag;
    ctor(DateType d); virtual ~dtor();
    self& operator{++,--}();
    virtual duration_type get_<neg>_offset(const DateType& current) const =0;
    const DateType& operator*() const; const DateType* operator->() const;
    bool operator{<,<=,>,>=,==,!=}(const self& d) const;
private: DateType current_;
};
struct date_itr<OffsetFunctor,DateType> : date_itr_base<DateType> {
    ctor(DateType d, int factor=1);
private: duration_type get_<neg>_offset(const DateType& current) const override;
    OffsetFunctor of_;
};
```

#### Utility

```c++
enum CV::violation_enum {min_violation,max_violation};
struct CV::constrained_value<ValuePolicies> {
    using value_type = ValuePolicies::value_type;
    ctor(value_type value); self& operator=(value_type v);
    static value_type max(), min();
    operator value_type() const;
protected: value_type value_;
private: void assign(value_type value);
};
class CV::simple_exception_policy<RepType min_value,max_value,ExceptionType> {
    struct exception_wrapper: public ExceptionType { operator std::out_of_range () const; };
    using actual_exception_type = conditional<is_base_of_v<std::exception,ExceptionType>, ExceptionType, exception_wrapper>::type;
public: using value_type = RepType;
    static value_type min(), max();
    static void on_error(RepType,RepType,violation_enum); // throw_exception(actual_exception_type{});
};

struct wrapping_int<Int wrap_val> {
    using int_type = Int;
    static int_type wrap_value(){return wrap_val;}
    ctor(int_type v);
    int_type as_int() const; operator int_type() const;
    Int2 add<Int2>(Int2 v); Int2 subtract<Int2>(Int2 v);
private: int_type value_;
};

struct wrapping_int2<Int wrap_min,wrap_max> {
    using int_type = Int;
    static int_type wrap_value(){return wrap_max;}, min_value(){return wrap_min;}
    ctor(int_type v);
    int_type as_int() const; operator int_type() const;
    Int2 add<Int2>(Int2 v); Int2 subtract<Int2>(Int2 v);
private: int_type value_; // wrap_min <= value_ <= wrap_max
};
```

date_facet, date_format_simple, date_formatting_<{limited,locales}>, date_generators_{formatter,parser}, date_names_put,
date_parsing, dst_rules, dst_translation_generators, filetime_functions, find_match, format_date_parser, gregorian_calendar,
int_adapter, iso_format, local_time_adjustor, local_timezone_defs, microsec_time_clock, parse_format_base,
period_{formatter,parser}, special_values_{formatter,parser}, string_convert, string_parse_tree, strings_from_facet
time_clock, time_facet, time_formatting_streams, time_iterator, time_parsing, time_resolution_traits,
time_system_{counted,split}, time_zone_{base,names}, tz_db_base

gregorian/: conversion, formatters_<limited>, greg_{calendar,date,day_of_year,day,duration_<types>,facet,month,serialize,weekday,year,ymd}, gregorian_<{io,types}>, parsers
local_time/: conversion, custom_time_zone, date_duration_operators, dst_transition_day_rules, local_date_time, local_time_<{io,types}>, posix_time_zone, tz_database
posix_time/: conversion, date_duration_operators, posix_time_<{config,duration,io,legacy_io,system,types}>, ptime, time_foramtters_<limited>, time_parsers, time_period, time_serialize

------
### Dependency

#### Boost.Algorithm

* `<boost/algorithm/string.hpp>`
* `<boost/algorithm/string/case_conv.hpp>`
* `<boost/algorithm/string/erase.hpp>`
* `<boost/algorithm/string/replace.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/config/no_tr1/cmath.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/limits.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`
* `<boost/core/nvp.hpp>`

#### Boost.IO

* `<boost/io/ios_state.hpp>`

#### Boost.LexicalCast

* `<boost/lexical_cast.hpp>`

#### Boost.Numeric/Conversion

* `<boost/numeric/conversion/cast.hpp>`

#### Boost.Range

* `<boost/range/as_literal.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Tokenizer

* `<boost/tokenizer.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/conditional.hpp>`
* `<boost/type_traits/integral_constant.hpp>`
* `<boost/type_traits/is_base_of.hpp>`
* `<boost/type_traits/is_integral.hpp>`

#### Boost.Utility

* `<boost/operators.hpp>`

#### Boost.WinAPI

* `<boost/winapi/time.hpp>`

------
### Standard Facilities
