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

struct microsec_clock<TimeType> {
    using time_converter = std::tm* (*)(const std::time_t*, std::tm*);
    using date_type = TimeType::date_type; using time_duration_type = TimeType::time_duration_type;
    using resolution_traits_type = time_duration_type::rep_type;
    static TimeType local_time<TimeZoneType>(shared_ptr<TimeZoneType> tz_ptr);
    static TimeType {local,universal}_time();
};

struct second_clock<TimeType> {
    using date_type = TimeType::date_type;
    using date_type = TimeType::date_type; using time_duration_type = TimeType::time_duration_type;
    static TimeType local_time<TimeZoneType>(shared_ptr<TimeZoneType> tz_ptr);
    static TimeType {local,universal}_time();
};

struct time_resolution_traits<FracSecType,time_resolutions res, FracSecType::int_type resolution_adjust, frac_digits, VarType=int64_t> {
    using fractional_seconds_type = FracSecType::int_type; using int_type = FracSecType::int_type; // and impl_type
    using {day,hour,min,sec}_type = VarType;
    static fractional_seconds_type as_number(impl_type i);
    static bool is_adapted();
    static constexpr fractional_seconds_type ticks_per_seconds = resolution_adjust;
    static time_resolutions resolution(); static unsigned short num_fractional_digits();
    static fractional_seconds_type res_adjust();
    static tick_type to_tick_count(hour_type hour, min_type minutes, sec_type seconds, fractional_seconds_type fs);
};
using milli_res = time_resolution_traits<...,milli,1000,3>;
using micro_res = time_resolution_traits<...,micro,1000000,6>;
using nano_res = time_resolution_traits<...,nano,1000000000,9>;
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

struct time_itr<TimeType> {
    using time_duration_type = TimeType::time_duration_type;
    ctor(TimeType t, time_duration_type d);
    self& operator{++,--}();
    const TimeType& operator*() const; const TimeType* operator->() const;
    bool operator{<,<=,!=,==,>,>=}(const TimeType& t) const;
};
```

##### Timezone

```c++
enum time_is_dst_result {is_not_in_dst, is_in_dst, ambiguous, invalid_time_label};
struct dst_calculator<DateType,TimeDurationType> {
    using date_type = DateType; using time_duration_type = TimeDurationType;
    static time_is_dst_result process_local_dst_{start,end}_day(const time_duration_type& time_of_day,
        unsigned dst_start_offset_minutes, long dst_length_minutes);
    static time_is_dst_result local_is_dst(const date_type& current_day, const time_duration_type& time_of_day,
                const date_type& dst_start_day, const time_duration_type& dst_start_offset,
                const date_type& dst_end_day, const time_duration_type& dst_end_offset, const time_duration_type& dst_length)
    static time_is_dst_result local_is_dst(const date_type& current_day, const time_duration_type& time_of_day,
                const date_type& dst_start_day, unsigned int dst_start_offset_minutes,
                const date_type& dst_end_day, unsigned int dst_end_offset_minutes, long dst_length_minutes)
};

struct dst_calc_engine<DateType,TimeDurationType,DstTraits> {
    using year_type = date_time::year_type; using calendar_type = DateType::calendar_type;
    using dstcalc = dst_calculator<DateType,TimeDurationType>;
    static time_is_dst_result local_is_dst(const DateType& d, const TimeDurationType& td);
    static bool is_dst_boundary_day(DateType d);
    static TimeDurationType dst_offset();
    static DateType local_dst_{start,end}_day(year_type year);
};

struct us_dst_rules<DateType,TimeDurationType,dst_start_offset_minutes=120,dst_length_minutes=60> {
    using time_duration_type = TimeDurationType; using date_type = DateType;
    using year_type = date_time::year_type; using calendar_type = DateType::calendar_type;
    using {l,f,n}kday = date_time::{last,first,nth}_kday_of_month<date_type>;
    using dstcalc = dst_calculator<DateType,TimeDurationType>;
    static time_is_dst_result local_is_dst(const DateType& d, const TimeDurationType& td);
    static bool is_dst_boundary_day(DateType d);
    static TimeDurationType dst_offset();
    static DateType local_dst_{start,end}_day(year_type year);
};
struct null_dst_rules<DateType,TimeDurationType> {
    using time_duration_type = TimeDurationType; using date_type = DateType;
    static time_is_dst_result local_is_dst(const DateType& d, const TimeDurationType& td);
    static bool is_dst_boundary_day(DateType d);
    static TimeDurationType dst_offset();
    static DateType local_dst_{start,end}_day(year_type year);
};

struct dst_day_calc_rule<DateType> {
    using year_type = DateType::year_type;
    virtual ~dtor();
    virtual date_type {start,end}_day(year_type y) const =0;
    virtual std::string {start,end}_rule_as_string() const =0;
};
struct day_calc_dst_rule<Spec> : public dst_day_calc_rule<Spec::date_type> {
    using date_type = Spec::date_type; using year_type = date_type::year_type;
    using {start,end}_rule = Spec::{start,end}_rule;
    ctor(start_rule dst_start, end_rule dst_end);
    virtual date_type {start,end}_day(year_type y) const override;
    virtual std::string {start,end}_rule_as_string() const override;
private: start_rule dst_start_; end_rule dst_end_;
};

struct utc_adjustment<TimeDurationType,hours,minutes=0> {
    static TimeDurationType local_to_utc_base_offset(), utc_to_local_base_offset();
};
struct dynamic_local_time_adjustor<TimeType,DstRules> : public DstRules {
    using time_duration_type = TImeType::time_duration_type; using date_type = TimeType::date_type;
    ctor(time_duration_type utc_offset);
    time_duration_type utc_offset(bool is_dst);
private: time_duration_type utc_offset_;
};
struct static_local_time_adjustor<TImeType,DstRules,UtcOffsetRules> {
    using time_duration_type = TImeType::time_duration_type; using date_type = TimeType::date_type;
    static TimeDurationType local_to_utc_base_offset(), utc_to_local_base_offset();
};
struct local_adjustor<TimeType,utc_offset,DstRule> {
    using time_duration_type = TimeType::time_duration_type; using date_type = TimeType::date_type;
    using dst_adjustor = static_local_time_adjustor<TimeType,DstRule,utc_adjustment<time_duration_type,utc_offset>>;
    static TimeType local_to_utc(), utc_to_local();
};

struct us_dst_trait<DateType> {
    using year_type = date_time::year_type; // and month_type, day_of_week_type
    using start_rule_functor = date_time::nth_kday_of_month<date_type>;
    using end_rule_functor = date_time::first_kday_of_month<date_type>;
    using start_rule_functor_pre2007 = date_time::first_kday_of_month<date_type>;
    using end_rule_functor_pre2007 = date_time::last_kday_of_month<date_type>;
    static day_of_week_type {start,end}_day(year_type) {return Sunday;}
    static month_type {start,end}_month(year_type y){return y<2007 ? {Apr,Oct}:{Mar,Nov};}
    static int dst_{start,end}_offset_minutes() {return 120;}
    static int dst_shift_length_minutes() {return 60;}
    static date_type local_dst_{start,end}_day(year_type year);
};
struct eu_dst_trait<DateType> {
    using year_type = date_time::year_type; // and month_type, day_of_week_type
    using {start,end}_rule_functor = date_time::last_kday_of_month<date_type>;
    static day_of_week_type {start,end}_day(year_type) {return Sunday;}
    static month_type {start,end}_month(year_type){return {Mar,Nov};}
    static int dst_{start,end}_offset_minutes() {return {120,180};}
    static int dst_shift_length_minutes() {return 60;}
    static date_type local_dst_{start,end}_day(year_type year);
};
struct uk_dst_trait<DateType> : public eu_dst_trait<DateType> {
    static int dst_{start,end}_offset_minutes() {return {60,120};}
    static int dst_shift_length_minutes() {return 60;}
};
struct acst_dst_trait<DateType> {
    using year_type = date_time::year_type; // and month_type, day_of_week_type
    using {start,end}_rule_functor = date_time::last_kday_of_month<date_type>;
    static day_of_week_type {start,end}_day(year_type) {return Sunday;}
    static month_type {start,end}_month(year_type){return {Oct,Mar};}
    static int dst_{start,end}_offset_minutes() {return {120,180};}
    static int dst_shift_length_minutes() {return 60;}
    static date_type local_dst_{start,end}_day(year_type year);
};

struct time_zone_base<TimeType,Ch> {
    using char_type = Ch;
    using string_type = std::basic_string<Ch>; using stringstream_type = std::basic_ostringstream<Ch>;
    using year_type = TimeType::date_type::year_type; using time_duration_type = TimeType::time_duration_type;
    ctor(){} virtual ~dtor(){}
    virtual string_type {dst,std}_zone_abbrev() const =0;
    virtual string_type {dst,std}_zone_name() const =0;
    virtual bool has_dst() const =0;
    virtual time_type dst_local_{start,end}_time(year_type y) const =0;
    virtual time_duration_type {base_utc,dst}_offset() const =0;
    virtual string_type to_posix_string() const =0;
};

struct dst_adjustment_offsets<TimeDurationType> {
    ctor(const TimeDurationType& dst_adjust, const TimeDurationType& dst_start_offset, const TimeDurationType& dst_end_offset);
    TimeDurationType dst_adjust_, dst_start_offset, dst_end_offset_;
};

struct default_zone_name<Ch> {
    using char_type = Ch;
    static const char_type standard_name[]="std_name", standard_abbrev[]="std_abbrev", non_dst_identifier[]="no-dst";
};
struct time_zone_names_base<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor();
    ctor(const string_type& std_zone_name_str, std_zone_abbrev_str, dst_zone_name_str, dst_zone_abbrev_str);
    string_type {dst,std}_zone_{abbrev,name}() const;
private: string_type std_zone_name_, std_zone_abbrev_, dst_zone_name_, dst_zone_abbrev_;
};

struct data_not_accessible : public std::logic_error { ctor(); ctor(const std::string& filespec); };
struct bad_field_count : public std::out_of_range {ctor(const std::string& s); };

struct tz_db_base<TimeZoneType,RuleType> {
    using char_type = char;
    using time_zone_base_type = TimeZoneType::base_type; using time_duration_type = TimeZoneType::time_duration_type;
    using time_zone_names = time_zone_names_base<char_type>;
    using dst_adjustment_offsets = dst_adjustment_offsets<time_duration_type>;
    using string_type = std::basic_string<char_type>;
    ctor();
    void load_from_stream(std::istream& in);
    void load_from_file(const std::string& pathspec);
    bool add_record(const string_type& region, shared_ptr<time_zone_base_type> tz);
    shared_ptr<time_zone_base_type> time_zone_from_region(const string_type& region) const;
    std::vector<std::string> region_list() const;
private: using map_type = std::map<string_type,shared_ptr<time_zone_base_type>>;
    map_type m_zone_map;
    using week_num = RuleType::start_rule::week_num;
    RuleType* parse_rules(const string_type& sr, const string_type& er) const;
    week_num get_week_num(int nth) const;
    void split_rule_spec(int& nth, int& d, int& m, string_type rule) const;
    bool parse_string(string_type& s);
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

struct int_adapter<IntType> {
    using int_type = IntType;
    ctor(int_type v);
    static bool is_<neg,pos>_info(int_type v); bool is_not_a_number(int_type v);
    static bool has_infinity(); static self {pos,neg}_infinity(); static self not_a_number(); static self {min,max}();
    static self from_special(special_values sv); static special_values to_special(int_type v);
    bool is_<pos,neg>_infinity() const; bool is_nan() const; bool is_special() const;
    bool operator{==,!=,<,>}(const self& rhs) const; bool operator{==,!=,<}(const int& rhs) const;
    int_type as_number() const; special_values as_special() const;
    self operator{+,-}<RhsType>(const self<RhsType>& rhs) const;
    self operator{*,/,%}(const self& rhs) const;
    self operator{+,-,*,/,%}(const int_type& rhs) const;
private: int_type value_;
};
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,IntType> (std::basic_ostream<Ch,Tr>& os, const int_adapter<IntType>& ia);

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

TimeT time_from_ftime<TimeT,FileTimeT>(const FileTimeT& ft); // Win32 FILETIME

short find_match<Ch>(const Ch* const* short_names, const Ch* const* long_names, short size, const std::basic_string<Ch>& s);

std::basic_string<OutT> convert_string_type<InT,OutT>(const std::basic_string<InT>& in_str);
struct parse_match_result<Ch> {
    using string_type = std::basic_string<Ch>;
    ctor();
    string_type remaining() const;
    Ch last_char() const;
    bool has_remaining() const;
    string_type cache; unsigned short match_depth; short current_match; enum PARSE_STATE{PARSE_ERROR=-1;};
};
std::basic_ostream<Ch>& operator<< <Ch>(std::basic_ostream<Ch>& os, parse_match_result<Ch>& mr);

struct string_parse_tree<Ch> {
    using ptree_coll = std::multimap<Ch,self>;
    using value_type = ptree_coll::value_type; // and <const>_iterator
    using string_type = std::basic_string<Ch>; using collection_type = std::vector<string_type>;
    using parse_match_result_type = parse_match_result<Ch>;
    ctor(collection_type names, unsigned int starting_point=0);
    ctor(short value=PARSE_ERROR);
    ptree_coll m_next_chars; short m_value;
    void insert(const string_type& s, unsigned short value);
    short match(std::istreambuf_iterator<Ch>& siter, std::istreambuf_iterator<Ch>& stream_end, parse_match_result_type& result, unsigned int& level) const;
    parse_match_result_type match(std::istreambuf_iterator<Ch>& siter, std::istreambuf_iterator<Ch>& stream_end) const;
    void printme(std::ostream& os, int& level);
    void print(std::ostream& os);
    void printmatch(std::ostream& os, Ch c);
};

struct counted_time_rep<Config> {
    using int_type = Config::int_type; // and date_type, impl_type, time_duration_type, resolution_traits
    using date_duration_type = date_type::duration_type;
    using ymd_type = date_type::ymd_type; using time_duration_type = date_type::time_duration_type; 
    ctor(const date_type& d, const time_duration_type& time_of_day);
    explicit ctor(int_type count); explicit ctor(impl_type count);
    date_type date() const; unsigned long day_count() const; int_type time_count() const; int_type tod() const;
    static int_type frac_sec_per_day();
    bool is_<pos,neg>_infinity() const; bool is_not_a_date_time() const; bool is_special() const;
    impl_type get_rep() const;
private: impl_type time_count_;
};

struct counted_time_system<TimeRep> {
    using time_rep_type = TimeRep;
    using impl_type = time_rep_type::impl_type; // and {date,time}_duration_type, date_time
    using fractional_seconds_type = time_duration_type::fractional_seconds_type;
    static time_rep_type get_time_rep(const date_type& day, const time_duration_type& tod, date_time::dst_flags dst=not_dst);
    static time_rep_type get_time_rep(special_values sv);
    static date_type get_date(const time_rep_type& val);
    static time_duration_type get_time_of_day(const time_rep_type& val);
    static std::string zone_name(const time_rep_type&);
    static bool is_{equal,less}(const time_rep_type& lhs, const time_rep_type& rhs);
    static time_rep_type {add,subtract}_days(const time_rep_type& base, const date_duration_type& dd);
    static time_rep_type {add,subtract}_time_duration(const time_rep_type& base, const time_duration_type& td);
    static time_duration_type subtract_times(const time_rep_type& lhs, const time_rep_type& rhs);
};

struct split_timedate_system<Config> {
    using time_rep_type = Config::time_rep_type; // and date_type, {date,time}_duration_type, int_type, resolution_traits
    static constexpr int_type ticks_per_day = 86400*config::tick_per_second;
    using wrap_int_type = wrapping_int<int_type,ticks_per_day>;
    static time_rep_type get_time_rep(special_values sv);
    static time_rep_type get_time_rep(const date_type& day, const time_duration_type& tod, date_time::dst_flags dst=not_dst);
    static date_type get_date(const time_rep_type& val);
    static time_duration_type get_time_of_day(const time_rep_type& val);
    static std::string zone_name(const time_rep_type&);
    static bool is_{equal,less}(const time_rep_type& lhs, const time_rep_type& rhs);
    static time_rep_type {add,subtract}_days(const time_rep_type& base, const date_duration_type& dd);
    static time_rep_type {add,subtract}_time_duration(const time_rep_type& base, const time_duration_type& td);
    static time_duration_type subtract_times(const time_rep_type& lhs, const time_rep_type& rhs);
};
```

##### I/O

```c++
enum month_format_spec {month_as_integer, month_as_short_string, month_as_long_string};
enum ymd_order_spec {ymd_order_iso, ymd_order_dmy, ymd_order_us};

struct simple_format<Ch> { // for char and wchar_t
    static const Ch* not_a_date(), {pos,neg}_infinity();
    static month_format_spec month_format();
    static ymd_order_spec date_order();
    static bool has_date_sep_chars(); static Ch {year,month,day,hour,minute,second}_sep_char();
};

struct iso_format_base<Ch> { // char and wchar_t
    static const Ch* not_a_date(), {pos,neg}_infinity();
    static month_format_spec month_format();
    static Ch {year,month,day,hour,minute,second}_sep_char();
    static Ch {period,time,week}_start_char(), {period,time,fractional_time,element}_sep_char();
    static bool is_component_sep(Ch sep), is_fractional_time_sep(Ch sep), is_timezone_sep(Ch sep);
};
struct iso_<extended>_format<Ch> : iso_format_base<Ch> { static bool has_date_sep_chars(); }; // false/true

std::string convert_to_lower(std::string in);
unsigned short month_str_to_ushort<MonthType>(std::string const& s);
DateType parse_date<DateType>(const std::string& s, int order_spec = ymd_order_iso);
DateType parse_undelimited_date<DateType>(const std::string& s);
DateType from_stream_type<DateType,It>(It& beg, It const& end, {std::<w>string const&,char,wchar_t})

Int power<Int>(Int base, Int exponent);
TimeDuration str_from_delimited_time_duration<TimeDuration,Ch>(const std::basic_string<Ch>& s);
TimeDuration parse_delimited_time_duration<TimeDuration>(const std::string& s);
bool split(const std::string& s, char sep, std::string& first, std::string second);
TimeType parse_delimited_time<TimeType>(const std::string& s, char sep);
TimeDuration parse_undelimited_time_duration<TimeDuration>(const std::string& s);
TimeType parse_iso_time<TimeType>(const std::string& s, char sep);

struct month_formatter<MonthType,FormatType,Ch=char>
{ static std::basic_ostream<Ch>& format_month(const MonthType& month, ostream_type& os); };
struct ymd_formatter<YmdType,FormatType,Ch=char>
{ static std::basic_string<Ch> ymd_to_string(YmdType ymd); };
struct date_formatter<DateType,FormatType,Ch=char>
{ static std::basic_string<Ch> date_to_string(DateType ymd); };
struct ostream_month_formatter<FacetType,Ch=char> {
    using month_type=FacetType::month_type; using ostream_type = std::basic_ostream<Ch>;
    static void format_month(const MonthType& month, ostream_type& os, const FacetType& f);
};
struct ostream_weekday_formatter<WeekdayType,FacetType,Ch=char> {
    using month_type=FacetType::month_type; using ostream_type = std::basic_ostream<Ch>;
    static void format_weekday(const WeekdayType& wd, ostream_type& os, const FacetType& f, bool as_long_string);
};
struct ostream_ymd_formatter<YmdType,FacetType,Ch=char> {
    using month_type=FacetType::month_type; using month_formatter_type = ostream_month_formatter<FacetType,Ch>;
    using ostream_type = std::basic_ostream<Ch>;
    static void ymd_put(YmdType ymd, ostream_type& os, const FacetType& f);
};
struct ostream_ymd_formatter<DateType,FacetType,Ch=char> {
    using ymd_type=DateType::ymd_type; using ostream_type = std::basic_ostream<Ch>;
    static void date_put(const DateType& d, ostream_type& os, <const FacetType& f>);
};

struct period_formatter<Ch,OutIt=std::ostreambuf_iterator<Ch,std::char_traits<Ch>>> {
    using char_type = Ch; using string_type = std::basic_string<Ch>; using const_itr_type = string_type::const_iterator;
    using collection_type = std::vector<string_type>;
    static const Ch default_period_separator[2], default_period_start_delimiter[2], default_period_{close,open}_range_end_delimiter[2]; // /, [, ), ]
    enum range_display_options {AS_OPEN_RANGE,AS_CLOSED_RANGE};
    ctor(range_display_options range_option_in=AS_CLOSED_RANGE, const Ch* const period_separator=default_period_separator,...); // al 4 delim
    OutIt put_period_separator(OutIt& o) const;
    OutIt put_period_{start,end}_delimiter(Out& o) const;
    range_display_options range_option() const; void range_option(range_display_options option) const;
    void delimiter_strings(const string_type& sep, start_delim, open_end_delim, closed_end_delim);
    OutIt put_period<PeriodType,FacetType>(OutIt next, std::ios_base& a_ios, Ch a_fill, const PeriodType& p, const FacetType& facet) const;
private: range_display_options m_range_option;
    string_type m_period_separator, m_period_start_delimiter, m_{open,closed}_range_end_delimiter;
};
struct period_parser<DateType,Ch> {
    using string_type = std::basic_string<Ch>; using char_type=Ch; using stream_itr_type=std::istreambuf_iterator<Ch>;
    using parse_tree_type = string_parse_tree<Ch>; using match_results = parse_tree_type::parse_match_result_type;
    using collection_type = std::vector<string_type>;
    static const Ch default_period_separator[2], default_period_start_delimiter[2], default_period_{close,open}_range_end_delimiter[2]; // /, [, ), ]
    enum period_range_option {AS_OPEN_RANGE,AS_CLOSED_RANGE};
    ctor(range_display_options range_option_in=AS_CLOSED_RANGE, const Ch* const period_separator=default_period_separator,...); // al 4 delim
    period_range_option range_option() const; void range_option(period_range_option option) const;
    collection_type delimiter_strings() const; void delimiter_strings(const string_type& sep, start_delim, open_end_delim, closed_end_delim);
    PeriodType get_period<PeriodType,DurationType,FacetType>(stream_itr_type& sitr, stream_itr_type& stream_end, std::ios_base& a_ios,
        const PeriodType&, const DurationType& dur_unit, const FacetType& facet) const;
private: collection_type delimiters; period_range_option m_range_option;
};

struct special_values_formatter<Ch,OutIt=std::ostreambuf_iterator<Ch,std::char_traits<Ch>>> {
    using char_type = Ch; using string_type = std::basic_string<Ch>;
    using collection_type = std::vector<string_type>;
    static const char_type default_special_value_names[3][17]; // not-a-date-time, {-,+}infinity
    ctor(); ctor(const Ch* const* begin, end); ctor(collection_type::iterator beg, end);
    OutIt put_special(OutIt next, const special_values& value) const;
protected: collection_type m_special_value_names;
};
struct special_values_parser<DateType,Ch> {
    using char_type = Ch; using string_type = std::basic_string<Ch>;
    using stringstream_type = std::basic_stringstream<Ch>; using stream_itr_type = std::istreambuf_iterator<Ch>;
    using duration_type = DateType::duration_type;
    using parse_tree_type = string_parse_tree<Ch>; using match_results = parse_tree_type::parse_match_result_type;
    using collection_type = std::vector<string_type>;
    static const char_type nadt_string[], {neg,pos}_inf_string[], {min,max}_date_time_string[]; // not-a-date-time, {-,+}infinity, {min,max}imum-date-time
    ctor(); ctor(collection_type::iterator beg, end);
    ctor(const string_type& nadt_str, neg_inf_str, pos_inf_str, min_dt_str, max_dt_str);
    void sv_strings(const string_type& nadt_str, neg_inf_str, pos_inf_str, min_dt_str, max_dt_str);
    static bool should_call_match(const string_type& str);
    bool match(stream_itr_type& sitr, stream_itr_type& str_end, match_results& mr) const;
private: parse_tree_type m_sv_strings;
};

struct format_date_parser<DateType,Ch> {
    using string_type = std::basic_string<Ch>; using const_itr = string_type::const_iterator;
    using stringstream_type = std::basic_istringstream<Ch>; using stream_itr_type = std::istreambuf_iterator<Ch>;
    using year_type = DateType::year_type; // and {month,day,duration,day_of_week,day_of_year}_type
    using parse_tree_type = string_parse_tree<Ch>; using match_results = parse_tree_type::parse_match_result_type;
    using input_collection_type = std::vector<string_type>;
    ctor(const string_type& format_str, const input_collection_type& month_short_names, month_long_names, weekday_short_names, weekday_long_names);
    ctor(const string_type& format_str, const std::locale& locale);
    ctor( const self& o);
    string_type format() const; void format(string_type format_str);
    void {short,long}_{month,weekday}_names(const input_collection_type& names);
    DateType parse_date(const string_type& value, const string_type& format_str, const special_values_parser<DateType,Ch>& sv_parser) const;
    DateType parse_{date,month}(stream_itr_type& sitr, stream_itr_type& stream_end, <string_type format_str>, const special_values_parser<DateType,Ch>& sv_parser) const;
    DateType parse_<var>_day_of_month(stream_itr_type& sitr, stream_itr_type& stream_end) const;
    {day_of_week,year}_type parse_{weekday,year}(stream_itr_type& sitr, stream_itr_type& stream_end, string_type format_str, <match_results& mr>) const;
private: string_type m_format; parse_tree_type m_month_short_names, m_month_long_names, m_weekday_short_names, m_weekday_long_names;
};

struct date_generator_formatter<DateType,Ch,OutIt=std::ostreambuf_iterator<Ch,std::char_traits<Ch>>> {
    using partial_date_type = partial_date<DateType>;
    using {nth,first,last}_kday_type = {nth,first,last}_kday_of_month<DateType>;
    using kday_{after,before}_type = first_kday_{after,before}<DateType>;
    using char_type = Ch; using string_type=std::basic_string<Ch>; using collection_type = std::vector<string_type>;
    static const char_type first_string[], second_string[], third_string[], // first, second, third, ...
        fourth_string[], fifth_string[], last_string[], before_string[], after_string[], of_string[];
    enum phrase_elements {first=0, second, third, fourth, fifth, last, before, after, of, number_of_phrase_elements};
    ctor(); ctor(const string_type& first_str, second_str, third_str, fourth_str, fifth_str, last_str, before_str, after_str, of_str);
    void elements(const collection_type& new_strings, phrase_elements beg_pos=first);
    OutIt put_partial_date<FacetType>(OutIt next, std::ios_base& a_ios, Ch a_fill, const partial_date_type& pd, const FacetType& facet) const;
    OutIt put_{nth,first,last}_kday<FacetType>(OutIt next, std::ios_base& a_ios, Ch a_fill, const {nth,first,last}_kday_type& kd, const FacetType& facet) const;
    OutIt put_kday_{before,after}<FacetType>(OutIt next, std::ios_base& a_ios, Ch a_fill, const kday_{before,after}_type& kd, const FacetType& facet) const;
private: collection_type phrase_strings;
};

struct date_generator_parser<DateType,Ch> {
    using char_type = Ch; using string_type=std::basic_string<Ch>; using stream_itr_type=std::istreambuf_iterator<Ch>;
    using month_type = DateType::month_type; // and day_of_week_type, day_type
    using parse_tree_type = string_parse_tree<Ch>; using match_results = parse_tree_type::parse_match_result_type;
    using collection_type = std::vector<std::basic_string<Ch>>;
    using partial_date_type = partial_date<DateType>;
    using {nth,first,last}_kday_type = {nth,first,last}_kday_of_month<DateType>;
    using kday_{after,before}_type = first_kday_{after,before}<DateType>;
    static const char_type first_string[], second_string[], third_string[], // first, second, third, ...
        fourth_string[], fifth_string[], last_string[], before_string[], after_string[], of_string[];
    enum phrase_elements {first=0, second, third, fourth, fifth, last, before, after, of, number_of_phrase_elements};
    ctor(); ctor(const string_type& first_str, second_str, third_str, fourth_str, fifth_str, last_str, before_str, after_str, of_str);
    void element_strings(const string_type& first_str, second_str, third_str, fourth_str, fifth_str, last_str, before_str, after_str, of_str);
    void element_strings(const collection_type& col);
    partial_date_type get_partial_date_type<FacetType>(stream_itr_type& sitr, stream_itr_type& stream_end, std:ios_base& a_ios, const FacetType& facet) const;
    {nth,first,last}_kday_type get_{nth,first,last}_kday_type<FacetType>(stream_itr_type& sitr, stream_itr_type& stream_end, std:ios_base& a_ios, const FacetType& facet) const;
    kday_{before,after}_type get_kday_{before,after}_type<FacetType>(stream_itr_type& sitr, stream_itr_type& stream_end, std:ios_base& a_ios, const FacetType& facet) const;
private: parse_tree_type m_element_strings;
};

struct date_facet<DateType,Ch,OutIt=std::ostreambuf_iterator<Ch,std::char_traits<Ch>>> : public std::locale::facet {
    using duration_type = DateType::duration_type; // day_of_week_type, day_type, month_type
    using period_type = period<DateType,duration_type>;
    using string_type = std::basic_string<Ch>; using char_type = Ch;
    using period_foramtter_type = period_formatter<Ch>; using special_values_formatter_type = special_values_formatter<Ch>;
    using input_collection_type = std::vector<std::basic_string<Ch>>;
    using date_gen_formatter_type = date_generator_formatter<DateType,Ch>;
    using partial_date_type = partial_date<DateType>;
    using {nth,first,last}_kday_type = {nth,first,last}_kday_of_month<DateType>;
    using kday_{after,before}_type = first_kday_{after,before}<DateType>;
    static const char_type long_weekday_format[]="%A", short_weekday_format[]="%a",
        long_month_format[]="%B", short_month_format[]="%b",
        default_period_separator[]=" / ", standard_format_specifier[]="%x",
        iso_format_specifier[]="%Y%m%d", iso_format_extended_specifier[]="%Y-%m-%d", default_data_format[]="%Y-%b-%d";
    static std::locale::id id;
    explicit ctor(size_t a_ref=0);
    explicit ctor(const char_type* format_str, const input_collection_type& short_names, size_t ref_count=0);
    explicit ctor(const char_type* format_str, period_formatter_type per_formatter={}, special_values_formatter_type sv_formatter={}, date_gen_formatter_type dg_formatter={}, size_t ref_count=0);
    void format(const char_type* const format_str);
    virtual void set_iso_<extended>_format();
    void {month,weekday}_format(const char_type* const format_str);
    void period_formatter(period_formatter_type per_formatter);
    void special_values_formatter(const special_values_formatter_type& svf);
    void {short_long}_{weekday,month}_names(const input_collection_type& names);
    void date_gen_phrase_strings(const input_collection_type& new_strings, date_gen_formatter_type::phrase_elements beg_pos=first);
    OutIt put(OutIt next, std::ios_base& a_ios, char_type fill_char,
        const {date,duration,month,day,day_of_week,period,partial_date,{nth,first,last}_kday_type,kday_{before,after}}_type& d) const;
protected:
    virtual OutIt do_put_special(OutIt next, std::ios_base&, char_type, const special_values sv) const;
    virtual OutIt do_put_tm(OutIt next, std::ios_base& a_ios, char_type fill_char, const tm& tm_value, string_type a_format) const;
    string_type m_format=default_date_format, m_month_format=short_month_format, m_weekday_format=short_weekday_format;
    period_formatter_type m_period_formatter; date_gen_formatter_type m_date_gen_formatter; special_values_formatter_type m_special_values_formatter;
    input_collection_type m_month_short_names, m_month_long_names, m_weekday_short_names, m_weekday_long_names;
};

struct date_input_facet<DateType,Ch,InIt=std::istreambuf_iterator<Ch,std::char_traits<Ch>>> : public std::locale::facet {
    using duration_type = DateType::duration_type; // day_of_week_type, day_type, month_type, year_type
    using period_type = period<DateType,duration_type>;
    using string_type = std::basic_string<Ch>; using char_type = Ch;
    using period_parser_type = period_parser<DateType,Ch>; using special_values_parser_type = special_values_parser<DateType,Ch>;
    using input_collection_type = std::vector<std::basic_string<Ch>>;
    using format_date_parser_type = format_date_parser<DateType,Ch>;
    using date_gen_parser_type = date_generator_formatter<DateType,Ch>;
    using partial_date_type = partial_date<DateType>;
    using {nth,first,last}_kday_type = {nth,first,last}_kday_of_month<DateType>;
    using kday_{after,before}_type = first_kday_{after,before}<DateType>;
    static const char_type long_weekday_format[]="%A", short_weekday_format[]="%a",
        long_month_format[]="%B", short_month_format[]="%b",
        four_digit_year_format[]="", two_digit_year_format[]="",
        default_period_separator[]=" / ", standard_format_specifier[]="%x",
        iso_format_specifier[]="%Y%m%d", iso_format_extended_specifier[]="%Y-%m-%d", default_data_format[]="%Y-%b-%d";
    static std::locale::id id;
    explicit ctor(size_t a_ref=0);
    explicit ctor(const string_type& format_str, size_t a_ref=0);
    explicit ctor(const string_type& format_str, const format_date_parser_type& date_parser,
        const special_values_parser_type sv_parser, const period_parser_type& per_parser,
        const date_gen_parser_type& date_gen_parser, size_t ref_count=0);
    void format(const char_type* const format_str);
    virtual void set_iso_<extended>_format();
    void {month,weekday,year}_format(const char_type* const format_str);
    void period_parser(period_parser_type per_parser);
    void {short_long}_{weekday,month}_names(const input_collection_type& names);
    void date_gen_element_strings(const input_collection_type& col);
    void date_gen_element_strings(const string_type& first, second, third, fourth, fifth, last, before, after, of);
    void special_values_parser(special_values_formatter_type sv_parser);
    InIt get(InIt& from, InIt& to, std::ios_base& a_ios,
        {date,duration,year,month,day,day_of_week,period,partial_date,{nth,first,last}_kday_type,kday_{before,after}}_type& d) const;
protected:
    virtual OutIt do_put_special(OutIt next, std::ios_base&, char_type, const special_values sv) const;
    virtual OutIt do_put_tm(OutIt next, std::ios_base& a_ios, char_type fill_char, const tm& tm_value, string_type a_format) const;
    string_type m_format=default_date_format, m_month_format=short_month_format, m_weekday_format=short_weekday_format, m_year_format;
    date_gen_parser_type m_date_gen_parser;
    period_parser_type m_period_parser;
    special_values_formatter_type m_special_values_formatter;
    input_collection_type m_month_short_names, m_month_long_names, m_weekday_short_names, m_weekday_long_names;
};

struct date_names_put<Config,Ch=char,OutIt=std::ostreambuf_iterator<Ch>> : std::locale::facet {
    ctor();
    using iter_type = OutIt;
    using month_type = Config::month_type; // and month_enum, weekday_enum, special_value_enum;
    using char_type = Ch; using string_type = std::basic_string<Ch>;
    static const char_type default_special_value_names[3][17], separator[2]; // not-a-date-time, +,-infinity, -
    static std::locale::id id;
    void put_special_value(iter_type& o, special_value_enum sv) const;
    void put_{month,weekday}_{short,long}(iter_type& o, {month,weekday}_enum d) const;
    bool has_date_sep_chars() const;
    void {year,month,day}_sep_char(iter_type& o) const;
    ymd_order_spec date_order() const; month_format_spec month_format() const;
protected:
    virtual void do_put_special_value(iter_type& o, special_value_enum sv) const;
    virtual void do_put_{month,weekday}_{short,long}(iter_type& o, {month,weekday}_enum d) const;
    virtual bool do_has_date_sep_chars() const;
    virtual void do_{year,month,day}_sep_char(iter_type& o) const;
    virtual ymd_order_spec do_date_order() const; virtual month_format_spec do_month_format() const;
    void put_string(iter_type& o, const {Ch*,string_type&} const s) const;
};
struct all_date_names_put<Config,Ch=char,OutIt=std::ostreambuf_iterator<Ch>> : date_names_put<Config,Ch,OutIt> {
    ctor(const Ch *month_short_names[], *month_long_names[], *special_value_names[], *weekday_short_names[], *weekday_long_names[],
        Ch separator_char='-', ymd_order_spec order_spec=ymd_order_iso, month_format_spec month_format=month_as_short_string);
    const Ch* const* get_{short,long}_{month,weekday}_names() const;
    const Ch* const* get_special_value_names() const;
protected:
    virtual void do_put_special_value(iter_type& o, special_value_enum sv) const;
    virtual void do_put_{month,weekday}_{short,long}(iter_type& o, {month,weekday}_enum d) const;
    virtual void do_{month,day}_sep_char(iter_type& o) const;
    virtual ymd_order_spec do_date_order() const; virtual month_format_spec do_month_format() const;
private:
    const Ch* const* month_short_names_, month_long_names_, special_value_names_, weekday_short_names_, weekday_long_names_;
    Ch separator_char_[2]; ymd_order_spec order_spec_; month_format_spec month_format_spec_;
};

std::vector<std::basic_string<Ch>> gather_{month,weekday}_strings<Ch>(const std::locale& locale, bool short_string=true);

struct time_formats<Ch> {
    using char_type = Ch;
    static const char_type fractional_seconds_<or_none>_format[3], // %f, %F
        seconds_<with_fractional_seconds>_format[3], // %S, %s
        <unrestricted>_hours_format[3], // %H, %O
        {full,short}_24_hour_time_<expanded>_format[3,6,9], // %T, %H:%M:%S, %R, %H:%M
        standard_format[9], // "%x %X %z"
        zone_{abbrev,name}_format[3], zone_iso_<extended>_format[3], posix_zone_string_format[4] // %z, %Z, %q, %Q, %ZP
        duration_sign_{negative_only,always}[3], duration_separator[2], // %-, %+, :
        {negative,positive}_sign[2], // -, +
        iso_time_format_<extended>_specifier[18,22], // "%Y%m%dT%H%M%S%F%q", "%Y-%m-%d %H:%M:%S%F%Q"
        default_time_<input,duration>_format[23,24,11]; // "%Y-%b-%d %H:%M:%S%F %z", "%Y-%b-%d %H:%M:%S%F %ZP", "%O:%M:%S%F"
};

struct time_facet<TimeType,Ch,OutIt=std::ostreambuf_iterator<Ch,std::char_traits<Ch>>> : date_facet<TimeType::date_type,Ch,OutIt> {
    using formats_type = time_formats<Ch>;
    using date_type = TimeType::date_type; // and time_duration_type
    using period_type = period<TimeType,time_duration_type>;
    static const char_type *fractional_seconds_<or_none>_format, *seconds_<with_fractional_seconds>_format,
        *<unrestricted>_hours_format, *standard_format,
        *zone_{abbrev,name}_format, *zone_iso_<extended>_format, *posix_zone_string_format,
        *duration_sign_{negative_only,always}, *duration_separator, *{negative,positive}_sign,
        *iso_time_format_<extended>_specifier, *default_time_<duration>_format;
    static std::locale::id id;
    explicit ctor(size_t ref_arg=0);
    explicit ctor(const char_type* format_arg, period_formatter_type period_formatter_arg={},
        const special_values_formatter_type& special_value_formatter={}, date_gen_formatter_type dg_formatter={}, size_t ref_arg=0);
    void time_duration_format(const char_type* format);
    void set_iso_<extended>_format() override;
    OutIt put(OutIt next, std::ios_base& ios_arg, char_type fill, const TimeType& time) const;
    OutIt put(OutIt next, std::ios_base& ios_arg, char_type fill, const {time_duration,period}_type& d) const;
protected:
    static string_type fractional_seconds_as_string(const time_duration_type& time_arg, bool null_when_zero);
    static string_type hours_as_string(const time_duration_type& time_arg, int width=2);
    static string_type integral_as_string<Int>(Int val, int width=2);
private: string_type m_time_duration_format;
};

struct time_input_facet<TimeType,Ch,OutIt=std::ostreambuf_iterator<Ch,std::char_traits<Ch>>> : date_input_facet<TimeType::date_type,Ch,OutIt> {
    using date_type = TimeType::date_type; // and time_duration_type
    using fractional_seconds_type = time_duration_type::fractional_seconds_type;
    using period_type = period<TimeType,time_duration_type>;
    using const_itr = string_type::const_iterator;
    static const char_type *fractional_seconds_<or_none>_format, *seconds_<with_fractional_seconds>_format,
        *standard_format, *zone_{abbrev,name}_format, *zone_iso_<extended>_format,
        *duration_separator, *iso_time_format_<extended>_specifier, *default_time_{input,duration}_format;
    static std::locale::id id;
    explicit ctor(size_t ref_arg=0);
    explicit ctor(const string_type& format, size_t ref_arg=0);
    explicit ctor(const string_type& format, const format_date_parser_type& date_parser,
        const special_values_parser_type sv_parser, const period_parser_type& per_parser,
        const date_gen_parser_type& date_gen_parser, size_t ref_arg=0);
    void time_duration_format(const char_type* format);
    void set_iso_<extended>_format() override;
    InIt get(InIt& from, InIt& to, std::ios_base& a_ios, {period,time_duration,time}_type& d) const;
    InIt get_local_time(InIt& from, InIt& to, std::ios_base& a_ios, time_type& t, string_type& tz_str) const;
protected:
    InIt get(InIt& from, InIt& to, std::ios_base& a_ios, time_type& t, string_type& tz_str, bool time_is_local) const;
    InIt check_special_value<TemporalType>(InIt& from, InIt& to, TemporalType& tt, char_type c='\0') const;
    void parse_frac_type(InIt& from, InIt& to, fractional_seconds_type& frac) const;
private: string_type m_time_duration_format;
};

struct ostream_time_duration_formatter<TimeDurationType,Ch=char> {
    using ostream_type = std::basic_ostream<Ch>;
    using fractional_seconds_type = TimeDurationType::fractional_seconds_type;
    static void duration_put(const TimeDurationType& td, ostream_type& os);
};
struct ostream_time_formatter<TimeType,Ch=char> {
    using ostream_type = std::basic_ostream<Ch>;
    using date_type = TimeType::date_type; // and time_duration_type
    using duration_formatter = ostream_time_duration_formatter<time_duration_type,Ch>;
    static void time_put(const TimeType& td, ostream_type& os);
};
struct ostream_time_period_formatter<TimePeriodType,Ch=char> {
    using ostream_type = std::basic_ostream<Ch>;
    using time_type = TimePeriodType::point_type;
    using time_formatter = ostream_time_formatter<time_type,Ch>;
    static void period_put(const TimePeriodType& tp, ostream_type& os);
};
```

------
### Gregorian

```c++
class gregorian_calendar_base<YmdType,DateIntType> {
    using ymd_type = YmdType; using {month,day,year}_type = ymd_type::{month,day,year}_type;
    using date_int_type = DateIntType;
    unsigned short day_of_week(const ymd_type& ymd);
    int week_number(const ymd_type& ymd);
    date_int_type <julian,modjulian>_day_number(const ymd_type& ymd);
    ymd_type from_<julian,modjulian>_day_number(date_int_type);
    bool is_leap_year(year_type);
    unsigned short end_of_month_day(year_type y, month_type m);
    ymd_type epoch();
    unsigned short days_in_week();
};

namespace gregorian {

using date_period = period<date,date_duration>;
using year_based_generator = year_based_generator<date>;
using partial_date = partial_date<date>;
using {nth,first,last}_day_of_the_week_in_month = {nth,first,last}_kday_of_month = {nth,first,last}_kday_of_month<date>;
using first_day_of_the_week_{after,before} = first_kday_{after,before} = first_kday_{after,before}<date>;
using day_clock = day_clock<date>;
using date_iterator = date_itr_base<date>;
using {day,week,month,year}_iterator = date_itr<{day,week,month,year}_functor<date>,date>;

struct date : date<self, gregorian_calendar, date_duration> {
    using year_type = gregorian_calendar::year_type; // and {month,day,day_of_year,ymd,date_rep,date_int}_type;
    using duration_type = date_duration;
    ctor(); ctor(year_type y, month_type m, day_type d); explicit ctor(const ymd_type& ymd);
    explicit ctor(const date_int_type& rhs); explicit ctor(date_rep_type rhs); explicit ctor(special_values sv);
    date_int_type julian_day() const; day_of_year_type day_of_year() const; date_int_type modjulian_day() const;
    int week_number() const; date_int_type day_number() const; date end_of_month() const;
    friend bool operator==(const self& lhs, const self& rhs);
};

struct bad_day_of_month : std::out_of_range{ctor();ctor(const std::string& s);};
using greg_day_policies = simple_exception_policy<unsigned short, 1, 31, bad_day_of_month>;
using greg_day_rep = constrained_value<greg_day_policies>;
struct greg_day : greg_day_rep {
    ctor(value_type day_of_month);
    value_type as_number() const; operator value_type() const;
};

struct bad_day_of_year : std::out_of_range{ctor();};
using greg_day_of_year_policies = simple_exception_policy<unsigned short, 1, 366,bad_day_of_year>;
using greg_day_of_year_rep = constrained_value<greg_day_of_year_policies>;

struct bad_month : std::out_of_range{ctor();};
using greg_month_policies = simple_exception_policy<unsigned short,1,12,bad_month>;
using greg_month_rep = constrained_value<greg_month_policies>;
struct greg_month : greg_month_rep {
    using month_enum = months_of_year;
    ctor(month_enum m); ctor(value_type m);
    operator value_type() const;
    value_type as_number() const; month_enum as_enum() const;
    const {char,wchar_t}* as_{short,long}_<w>string(<{char,wchar_t}>) const;
};

struct bad_weekday : std::out_of_range{ctor();};
using greg_weekday_policies = simple_exception_policy<unsigned short,0,6,bad_weekday>;
using greg_weekday_rep = constrained_value<greg_weekday_policies>;
struct greg_weekday : greg_weekday_rep {
    using weekday_enum = weekdays;
    ctor(value_type dow);
    value_type as_number() const; weekday_enum as_enum() const;
    const {char,wchar_t}* as_{short,long}_<w>string() const;
};

struct bad_year : std::out_of_range{ctor();};
using greg_year_policies = simple_exception_policy<unsigned short,1400,9999,bad_year>;
using greg_year_rep = constrained_value<greg_year_policies>;
struct greg_year : greg_year_rep {
    ctor(value_type y);
    operator value_type() const;
};

using greg_year_month_day = year_month_day_base<greg_year,greg_month,greg_day>;

using date_duration_rep = duration_traits_adapted;
struct date_duration : date_duration<date_duration_rep> {
    explicit ctor(duration_rep day_count=0); ctor(special_values sv); ctor(const base& oth);
    bool operator{==,!=,<,>,<=,>=}(const self& rhs) const;
    self& operator{-=,+=}(const self& rhs); friend self operator{-,+}(self rhs, self const& lhs);
    self operator-() const;
    self& operator/=(int d); friend self operator/(self rhs, int lhs);
    self unit();
};

struct greg_durations_config { using date_type=date; using int_rep = int_adapter<int>; using month_adjustor_type=month_functor<date_type>; };
using months = months_duration<greg_durations_config>;
using years = years_duration<greg_durations_config>;
struct weeks_duration : date_duration { ctor(duration_rep w); ctor(special_values sv); };
using weeks = weeks_duration;

using fancy_date_rep = int_adapter<uint32_t>;
struct gregorian_calendar : gregorian_calendar_base<greg_year_month_day,fancy_date_rep::int_type> {
    using day_of_week_type = greg_weekday; using day_of_year_type = greg_day_of_year_rep;
    using date_rep_type = fancy_date_rep; using date_traits_type = fancy_date_rep;
};

std::tm to_tm(const date& d);
date date_from_tm(const std::tm& datetm);

std::basic_string<Ch> to_{simple,iso}_string_type<Ch>(const date_<period>& d);
std::<w>string to_{simple,iso}_<w>string(const date_<period>& d);
std::basic_string<Ch> to_iso_extended_string_type<Ch>(const date& d);
std::<w>string to_iso_extended_<w>string(const date& d);
std::basic_string<Ch> to_sql_string_type(const date& d);
std::<w>string to_sql_<w>string(const date& d);

special_values special_value_from_string(const std::string& s);
date from_<simple,us,uk,undelimited>_string(const std::string& s);
date date_from_iso_string(const std::string& s);
date from_stream<It>(It beg, It end);
date_period date_period_from_<w>string(const std::<w>string& s);

using <w>period_formatter = period_formatte<{wchar_t,char}>;
using <w>date_facet = date_facet<date,{wchar_t,char}>;
using <w>period_parser = period_parser<date,{wchar_t,char}>;
using <w>special_values_formatter = special_values_formatter<{wchar_t,char}>;
using <w>special_values_parser = special_values_parser<date,{wchar_t,char}>;
using <w>date_input_facet = date_input_facet<date,{wchar_t,char}>;
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os,
    const {date_<duration,period>,greg_{month,weekday},partial_date,{nth,first,last}_day_of_the_week_in_month,first_day_of_the_week_{after,before}}& d);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>& is,
    {date_<duration,period>,greg_{month,weekday,day,year},partial_date,{nth,first,last}_day_of_the_week_in_month,first_day_of_the_week_{after,before}}& d);
}

// T: for date_<duration,period> date_duration::duration_rep, greg_{year,month,day,weekday}, partial_date, {nth,first,last}_kday_of_month, first_kday_{before,after}
void serialize<Ar>(Ar& ar, T& t, unsigned ver);
void save<Ar>(Ar& ar, const T& t, unsigned);
void load<Ar>(Ar& ar, T& t, unsigned);
void load_construct_data<Ar>(Ar&, T* p, unsigned);
```

------
### Posix Time

```c++
namespace posix_time;

using time_res_traits = time_resolution_traits<...,micro,1000000,6>;
struct time_duration : time_duration<self, time_res_traits> {
    using rep_type = time_res_traits;
    using day_type = time_res_traits::day_type; // and {hour,min,sec,fractional_seconds,tick,impl}_type
    ctor(hour_type hour, min_type min, sec_type sec, fractional_seconds_type fs=0);
    ctor(); ctor(special_values sv);
protected: explicit ctor(impl_type tick_count);
};
struct millisec_posix_time_system_config {
    using time_rep_type = int64_t;
    using date_type = gregorian::date; using date_duration_type = gregorian::date_duration;
    using time_duration_type = time_duration; using resolution_traits = time_res_traits;
    using int_type = time_res_traits::tick_type; using impl_type = time_res_traits::impl_type;
    static constexpr int64_t tick_per_second = 1000000;
};

struct ptime : public base_time<self,posix_time_system> {
    using time_system_type = posix_time_system; using time_type = self;
    using time_rep_type = time_system_type::time_rep_type; // and time_duration_type
    ctor(gregorian::date d, time_duration_type td); explicit ctor(gregorian::date d);
    ctor(const time_rep_type& rhs); ctor(const special_values sv); ctor();
    friend operator==(const self& lhs, const self& rhs);
};

ptime operator{+,-}(const ptime& t, const gregorian::{months,years}& m);
ptime operator{+=,-=}(ptime& t, const gregorian::{months,years}& m);

struct hours : time_duration { explicit ctor<integral T>(T const& h); };
struct minutes : time_duration { explicit ctor<integral T>(T const& h); };
struct seconds : time_duration { explicit ctor<integral T>(T const& h); };
using millisec<onds> = subsecond_duration<time_duration,1000>;
using microsec<onds> = subsecond_duration<time_duration,1000000>;

using time_period = period<ptime,time_duration>;

using int64_time_rep = counted_time_rep<millisec_posix_time_system_config>;
using posix_time_system = counted_time_system<int64_time_rep>;

using time_iterator = time_itr<ptime>;
using second_clock = second_clock<ptime>;
using microsec_clock = microsec_clock<ptime>;
using no_dst = null_dst_rules<ptime::date_type,time_duration>;
using us_dst = us_dst_rules<ptime::date_type,time_duration>;

ptime from_time_t(time_t t);
time_t to_time_t(ptime pt);
std::tm to_tm(const {ptime,time_duration}& t);
ptime ptime_from_tm(const std::tm& t);
TimeT from_ftime<TimeT,FileTimeT>(const FileTimeT& ft);

std::basic_string<Ch> to_simple_string_type<Ch>({time_{duration,period},ptime} t);
std::string to_simple_<w>string({time_{duration,period},ptime} t);
std::basic_string<Ch> to_iso_string_type<Ch>({time_duration,ptime} t);
std::string to_iso_<w>string({time_duration,ptime} t);
std::basic_string<Ch> to_iso_extended_string_type<Ch>(ptime t);
std::string to_iso_extended_<w>string(ptime t);

time_duration duration_from_string(const std::string&s );
ptime time_from_string(const std::string& s);
ptime from_iso_<extended>_string(const std::string& s);

using <w>time_facet = time_facet<ptime,{wchar_t,char}>;
using <w>time_input_facet = time_input_facet<ptime,{wchar_t,char}>;
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os,
    const {ptime,time_{duration,period}}& p);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>& os,
    {ptime,time_{duration,period}}& p);

// for T in ptime, time_duration, time_period
void serialize<Ar>(Ar& ar, T& t, unsigned ver);
void save<Ar>(Ar& ar, const T& td, unsigned ver);
void load<Ar>(Ar& ar, T& td, unsigned ver);
void load_construct_data<Ar>(Ar&, {ptime,time_period}* pt, unsigned);
```

------
### Local Time

```c++
struct ambiguous_result : std::logic_error { ctor(std::string const& msg={}); };
struct time_label_invalid : std::logic_error { ctor(std::string const& msg={}); };
struct dst_not_valid : std::logic_error { ctor(std::string const& msg={}); };

struct local_date_time_base<UtcTime=posix_time::ptime, TzType=time_zone_base<UtcTime,char>> : base_time<UtcTime,PosixTimeSystem> {
    using utc_time_type = UtcTime;
    using date_type = utc_time_type::date_type; // time_duration_type, time_system_type
    using date_duration_type = date_type::duration_type;
    ctor(utc_time_type t, shared_ptr<TzType> tz);
    ctor(date_type d, time_duration_type td, shared_ptr<TzType> tz, bool dst_flag);
    enum DST_CALC_OPTIONS { EXCEPTION_ON_ERROR, NOT_DATE_TIME_ON_ERROR };
    ctor(date_type d, time_duration_type td, shared_ptr<TzType> tz, DST_CALC_OPTIONS calc_option);
    static time_is_dst_result check_dst(date_type d, time_duration_type td, shared_ptr<TzType> tz);
    ~dtor(); ctor(const self& rhs);
    explicit ctor(const special_values sv, shared_ptr<TzType> tz={});
    shared_ptr<TzType> zone() const; bool is_dst() const;
    utc_time_type utc_time() const, local_time() const;
    std::string to_string() const;
    self local_time_in(shared_ptr<TzType> new_tz, time_duration_type td={0,0,0}) const;
    std::string zone_{name,abbrev}(bool as_offset=false) const;
    std::string zone_as_posix_string() const;
    bool operator{==,!=,<,<=,>,>=}(const self& rhs) const;
    self operator{+,-}(const {date,time}_duration_type& d) const;
    self operator{+=,-=}(const {date,time}_duration_type& d);
    self operator-(const self& rhs) const;
private: shared_ptr<TzType> zone_;
};

local_date_time operator{+,-}(const local_date_time& t, const gregorian::{months,years}& t);
local_date_time operator{+=,-=}(local_date_time& t, const gregorian::{months,years}& t);

using dst_adjustment_offsets = dst_adjustment_offsets<posix_time::time_duration>;
using dst_calc_rule_ptr = shared_ptr<dst_calc_rule>;

struct custom_time_zone_base<Ch> : time_zone_base<posix_time::ptime, Ch> {
    using time_duration_type = posix_time::time_duration;
    using time_zone_names = time_zone_names_base<Ch>; using char_type = Ch;
    ctor(const time_zone_names& zone_names, const time_duration_type& utc_offset,
        const dst_adjustment_offsets& dst_shift, shared_ptr<dst_calc_rule> calc_rule);
    virtual ~dtor();
    virtual string_type {dst,std}_zone_<abbrev,name>() const;
    virtual bool has_dst() const;
    virtual posix_time:ptime dst_local_{start,end}_time(gregorian::greg_year y) const;
    virtual time_duration_type {base_utc,dst}_offset() const;
    virtual string_type to_posix_string() const;
private: time_zone_names zone_names_; time_duration_type base_utc_offset_;
    dst_adjustment_offsets dst_offsets_; shared_ptr<dst_calc_rule> dst_calc_rules_;
};

using tz_database = tz_db_base<custom_time_zone, nth_kday_dst_rule>;

using local_time_period = period<local_date_time, posix_time::time_duration>;
using local_time_iterator = time_itr<local_date_time>;
using local_<micro>sec_clock = <micro>sec_clock<local_date_time>;
using <w>time_zone = time_zone_base<posix_time::ptime, {char,wchar_t}>;
using <w>time_zone_ptr = shared_ptr<<w>time_zone>;
using <w>time_zone_names = time_zone_names_base<{char,wchar_t}>;

using dst_calc_rule = dst_day_calc_rule<gregorian::date>;
struct partial_date_rule_spec {
    using date_type = gregorian::date;
    using {start,end}_rule = gregorian::partial_date;
};
using partial_date_dst_rule = day_calc_dst_rule<partial_date_rule_spec>;
struct {first,last,nth}_last_rule_spec {
    using date_type = gregorian::date;
    using start_rule = gregorian::{first,last,nth}_kday_of_month;
    using end_rule = gregorian::last_kday_of_month;
};
using {first,last,nth}_last_dst_rule = day_calc_dst_rule<{first,last,nth}_last_rule_spec>;
struct nth_kday_rule_spec {
    using date_type = gregorian::date;
    using {start,end}_rule = gregorian::nth_kday_of_month;
};
using nth_day_of_the_week_in_month_dst_rule = nth_kday_dst_rule = day_calc_dst_rule<nth_kday_rule_spec>;

struct bad_offset : std::out_of_range {ctor(std::string const& msg={});};
struct bad_adjustment : std::out_of_range {ctor(std::string const& msg={});};
using dst_adjustment_offsets = dst_adjustment_offsets<posix_time::time_duration>;

struct posix_time_zone_base<Ch> : time_zone_base<posix_time::ptime,Ch> {
    using time_duration_type = posix_time::time_duration;
    using time_zone_names = time_zone_names_base<Ch>;
    using char_type = Ch; using char_separator_type = char_separator<char_type,std::char_traits<char_type>>;
    using tokenizer_type = boost::tokenizer<char_separator_type,string_type::const_iterator,string_type>;
    using tokenizer_iterator_type = tokenizer_type::iterator;
    ctor(const string_type& s); virtual ~dtor();
    virtual string_type {std,dst}_zone_{abbrev,name}() const;
    virtual bool has_dst() const;
    virtual posix_time::ptime dst_local_{start,end}_time(gregorian::greg_year y) const;
    virtual time_duration_type {base_utc,dst}_offset() const;
    virtual string_type to_posix_string() const;
private: time_zone_names zone_names_; bool has_dst_; time_duration_type base_utc_offset_;
    dst_adjustment_offsets dst_offsets_; shared_ptr<dst_calc_rule> dst_calc_rules_;
};
using posix_time_zone = posix_time_zone_base<char>;

std::tm to_tm(const local_date_time& lt);

using <w>local_time_facet = time_facet<local_date_time,{char,wchar_t}>;
using <w>local_time_input_facet = time_input_facet<local_date_time::utc_time_type,{char,wchar_t}>;
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, const {local_date_time,local_time_period}& ldt);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>& os, {local_date_time,local_time_period}& ldt);
```

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
