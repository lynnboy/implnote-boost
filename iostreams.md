# Boost.Range

* lib: `boost/libs/iostreams`
* repo: `boostorg/iostreams`
* commit: `8561bdb`, 2024-12-16

------
### Concept

```c++
// Devices
concept Device<D,Ch=char_type_of<D>::type,Cat=category_of<D>::type,Mode=mode_of<D>::type> =
    convertible_to<Cat,device_tag> && convertible_to<Cat,Mode> &&
    requires(D dev, Ch* s1, const Ch* s2, std::streamsize n, stream_offset off, ios_base::seekdir way, ios_base::openmode which) {
    {io::read(dev,s1,n)}->same_as<std::streamsize>; // when Cat=>input && !Cat=>direct_tag
    {dev.input_sequence()}->same_as<std::pair<Ch*,Ch*>>; // when Cat=>input && Cat=>direct_tag
    {io::write(dev,s2,n)}->same_as<std::streamsize>; // when Cat=output && !Cat=>direct_tag
    {dev.output_sequence()}->same_as<std::pair<Ch*,Ch*>>; // when Cat=>output && Cat=>direct_tag
    {io::seek(dev,off,way)}->same_as<std::streampos>; // when Cat=>seekable && !Cat=>direct_tag
    {dev.seek(dev,off,way,which)}->same_as<std::streampos>; // when (Cat=>dual_seekable || Cat=>bidirectional_seekable) && !Cat=>direct_tag
};
concept Source<D> = Device<D> && convertible_to<Mode,input>; // read, input_sequence
concept Sink<D> = Device<D> && convertible_to<Mode,output>; // write, output_sequence
concept BidirectionalDevice<D> = Source<D> && Sink<D> && convertible_to<Mode,bidirectional>; // read/write, input/output_seqeunce
concept SeekableDevice<D> = Source<D> && Sink<D> && convertible_to<Mode,seekable>; // read/write/seek, input/output_seqeunce
concept SeekableSource<D> = Source<D> && convertible_to<Mode,input_seekable>;
concept SeekableSink<D> = Sink<D> && convertible_to<Mode,output_seekable>;
concept BidirectionalSeekableDevice<D> = Source<D> && Sink<D> && convertible_to<Mode,bidirectional_seekable>;
concept DualSeekableDevice<D> = Source<D> && Sink<D> && convertible_to<Mode,dual_seekable>;
concept Peekable<D> = Source<D,Ch,Cat> && requires(D p, Ch c) { {io::putback(p,c)}->same_as<bool>; };
concept Blocking<D> = Device<D,Ch,Cat>;
concept Direct<D> = Device<D,Ch,Cat>;

// Filters
concept Filter<F,Ch=char_type_of<F>::type,Cat=category_of<F>::type,Mode=mode_of<F>::type,Device<Ch,Cat,Mode> D> =
    convertible_to<Cat,filter_rag> && convertible_to<Cat,Mode> &&
    requires(F f, D d, Ch c, Ch* s1, const Ch* s2, std::streamsize n, stream_offset off, ios_base::seekdir way, ios_base::openmode which) {
    using Tr=char_traits<Ch>;
    {char_type_of<D>::type}->same_as<Ch>; {category_of<D>::type}->same_as<Cat>;
    {f.get(d)}->same_as<Tr::int_type>; // when Cat=>input && !Cat=>multichar_tag
    {f.read(d,s1,n)}->same_as<streamsize>; // when Cat=>input && Cat=>multichar_tag
    {f.put(d,c)}->same_as<bool>; // when Cat=>output && !Cat=>multichar_tag
    {f.write(d,s2,n)}->same_as<streamsize>; // when Cat=>output && Cat=>multichar_tag
    {f.seek(d,off,way)}->same_as<streampos>; // when Cat=>seekable && !Cat=>direct_tag
    {f.seek(d,off,way,which)}->same_as<std::streampos>; // when (Cat=>dual_seekable || Cat=>bidirectional_seekable) && !Cat=>direct_tag
};
concept InputFilter<F> = Filter<F> && convertible_to<Mode,input>;
concept OutputFilter<F> = Filter<F> && convertible_to<Mode,output>;
concept BidirectionalFilter<F> = InputFilter<F> && OutputFilter<F> && convertible_to<Mode,bidirectional>;
concept SeekableFilter<F> = InputFilter<F> && OutputFilter<F> && convertible_to<Mode,seekable>;
concept InputSeekableFilter<F> = InputFilter<F> && convertible_to<Mode,input_seekable>;
concept OutputSeekableFilter<F> = OutputFilter<F> && convertible_to<Mode,output_seekable>;
concept BidirectionalSeekableFilter<F> = InputFilter<F> && OutputFilter<F> && convertible_to<Mode,bidirectional_seekable>;
concept DualSeekableFilter<F> = InputFilter<F> && OutputFilter<F> && convertible_to<Mode,dual_seekable>;
concept MultiCharacterFilter<F> = Filter<F> && convertible_to<Mode,multichar_tag>;
concept DualUseFilter<F> = BidirectionalFilter<F> && convertible_to<Mode,dual_use>;

concept Pipable<P, CopyConstructible C> = (Filter<C> || Device<C>) && requires (P p, C c) { f | c; };

// Other
concept Closable<C,Ch=char_type_of<C>::type,Cat=category_of<C>::type,Blocking<Ch> D> =
    convertible_to<Cat,closable_tag> && convertible_to<mode_of<D>::type,mode_of<C>::type> &&
    requires (C c, D d, ios_base::openmode w) {
    {io::close(c,w)}->void; // when Cat=>device_tag
    {io::close(c,d,w)}->void; // when Cat=>filter_tag
};
concept Flushable<F,Cat=category_of<F>::type,Sink D> =
    convertible_to<Cat,flushable_tag> &&
    requires (F f, D d) {
    {io::flush(f)}->convertible_to<bool>; // when Cat=>device_tag
    {io::flush(f,d)}->convertible_to<bool>; // when Cat=>filter_tag
};
concept Localizable<L> = requires(L z, const std::locale& loc) { {io::imbue(z,loc)}->void; };
concept OptimallyBuffered<O> = requires (O o) { {io::optimal_buffer_size(0)}->convertible_to<std::streamsize>; };
concept SymmetricFilter<F,Ch=char_type_of<F>::type> =
    requires (F f, const Ch* i1, const Ch* i2, Ch* o1, Ch* o2, bool flush) {
    {f.filter(i1,i2,o1,o2,flush)}->convertible_to<bool>;
    {f.close()}->void;
};
```

------
### Core Components

```c++
```

------
### Details

```c++
std::string detail::absolute_path(const std::string& path);
std::string detail::current_directory();
std::locale detail::add_facet<Facet>(const std::locale& l, Facet* f);

struct protected_{}; struct public_{};
struct detail::prot_<U> : protected U{ using base::ctor; };
struct detail::pub_<U> : public U{ using base::ctor; };
struct detail::access_control_base<T,Access> { using bad_access_specifier=int;
    using type = select<is_same<Access,protected_>, prot_<T>, is_same<Access,public_>, pub_<T>, else_, bad_access_specifier>::type;
};
struct access_control<T,Access> : access_control_base<T,Access>::type{ using base::ctor; };

struct detail::case_<n>;
using detail::yes_type = case_<true>; using detail::no_type = case_<false>;
using else_ = mpl::true_;
struct select<C1=mpl::true_,T1=mpl::void_,...>; // up to 12

class detail::basic_buffer<Ch,A=std::allocator<Ch>> { // no copy
    Ch* buf_; std::streamsize size_;
public: ctor(); ctor(std::streamsize buffer_size); ~dtor();
    void resize(std::streamsize buffer_size);
    Ch* begin() const; Ch* end() const; Ch* data() const; std::streamsize size() const;
    void swap(self& rhs);
};
void detail::swap<Ch,A>(basic_buffer<Ch,A>& lhs, basic_buffer<Ch,A>& rhs);

class detail::buffer<Ch,A=std::allocator<Ch>> : public basic_buffer<Ch,A> {
    Ch *ptr_, *eptr_;
public: using traits_type = char_traits<Ch>; using const_pointer = const Ch*;
    <const> Ch* & ptr() <const>; <const> Ch* & eptr() <const>;
    void set(std::streamsize ptr, std::streamsize end);
    void swap(self& rhs);
    int_type_of<Source>::type fill<Source>(Source& src);
    bool flush<Sink>(Sink& dest);
};
void detail::swap<Ch,A>(buffer<Ch,A>& lhs, buffer<Ch,A>& rhs);

struct detail::param_type<T>; // is_std_io<T> ? T& : const T&
struct detail::value_type<T>; // is_std_io<T> ? T& : T

#define CHAR_TRAITS(ch) std::char_traits<ch>

struct detail::codecvt_intern<T>{using type=T::intern_type;};
struct detail::codecvt_extern<T>{using type=T::extern_type;};
struct detail::codecvt_state<T>{using type=T::state_type;};
struct detail::codecvt_helper<Intern,Extern,State> : std::codecvt<Intern,Extern,State> {
    using intern_type=Intern; using extern_type = Extern; using state_type = State;
    ctor(size_t refs=0):base{refs}{}
};
struct detail::default_codecvt{using intern_type=wchar_t; using extern_type=char; using state_type=std::mbstate_t;};
struct detail::codecvt_holder<Cvt>{ using codecvt_type=Cvt; const Cvt& get() const; void imbue(const std::locale&){} Cvt codecvt_; };
struct detail::codecvt_holder<default_codecvt> {
    using codecvt_type = std::codecvt<wchar_t,char,std::mbstate_t>;
    ctor(){reset_codecvt();}
    const codecvt_type& get() const { return *codecvt_; }
    void imbue(const std::locale& loc) { loc_=loc; reset_codecvt(); }
    void reset_codecvt() { using namespace std; codecvt_= &use_facet<codecvt_type>(loc_); }
    std::locale loc_; const codecvt_type* codecvt_;
};

struct detail::counted_array_source<Ch> {
    using char_type = Ch; using category = source_tag;
    ctor(const Ch* buf, std::streamsize size) : buf_{buf}, ptr_{nullptr}, end_{size}{}
    std::streamsize read(Ch* s, std::streamsize n);
    std::streamsize count() const;
private: const Ch* buf_; std::streamsize ptr_, end_;
};
struct detail::counted_array_sink<Ch> {
    using char_type = Ch; using category = sink_tag;
    ctor(const Ch* buf, std::streamsize size) : buf_{buf}, ptr_{nullptr}, end_{size}{}
    std::streamsize write(Ch* s, std::streamsize n);
    std::streamsize count() const;
private: const Ch* buf_; std::streamsize ptr_, end_;
};

#define Default_Arg(arg) arg

struct detail::dispatch<T,Tag1,Tag2,Tag3=void_,Tag4=void_,Tag5=void_,Tag6=void_,Category=category_of<T>::type>
    : select<is_convertible<Category,Tag1>,Tag1, ...> {};

struct detail::single_object_holder<T> {
    using traits_type = call_traits<T>;
    using param_type = traits_type::param_type; using <const>_reference = traits_type::<const>_reference;
    ctor(); ctor(param_type t);
    <const>_reference first() <const>; <const>_reference second() <const>;
    void swap(self& o);
private: T& first_;
};
struct detail::double_object_holder<T> {
    using traits_type = call_traits<T>;
    using param_type = traits_type::param_type; using <const>_reference = traits_type::<const>_reference;
    ctor(); ctor(param_type t1, param_type t2);
    <const>_reference first() <const>; <const>_reference second() <const>;
    void swap(self& o);
private: T& first_, second_;
};
struct detail::double_object<T,IsDouble> : if_<IsDouble,double_object_holder<T>,single_object_holder<T>>::type { bool is_double() const; };

std::ios::failure detail::cant_read(); // and cant_write, cant_seek, bad_read, bad_putback, bad_write, write_area_exhausted, bad_seek
std::ios::failure detail::system_failure(const char* msg);
std::ios::failure detail::system_failure(const std::string& msg);
void detail::throw_system_failure(const char* msg);
void detail::throw_system_failure(const std::string& msg);

struct detail::execute_traits<Op> {
    using result_type = if_<is_void<result_of<Op()>::type>,int,result_of<Op()>::type>::type;
    static result_type execute(Op op) { if (is_void<result_of<Op()>::type>) return op(),0; else return op(); }
};
execute_traits<Op>::result_type execute_all<Op,...C>(Op op, C...c); // try op(); force call each cleanups c() in order
Op detail::execute_foreach<InIt,Op>(InIt first, InIt last, Op op); // try each op(*first++)

using file_handle = int; // or HANDLE on Windows

struct detail::device_close_operation<T> { // no copy
    ctor(T& t, ios::openmode which);
    void operator()() const { close(t_, which_); }
private: T& t_; ios::openmode which_;
};
struct detail::filter_close_operation<T,Sink> { // no copy
    ctor(T& t, Sink& snk, ios::openmode which);
    void operator()() const { close(t_, snk_, which_); }
private: T& t_; Sink& snk_; ios::openmode which_;
};
device_close_operation<T> detail::call_close<T>(T& t, ios::openmode which);
filter_close_operation<T,Sink> detail::call_close<T,Sink>(T& t, Sink& snk, ios::openmode which);

struct detail::device_close_all_operation<T> {
    ctor(T& t);
    void operator()() const { close_all(t_); }
private: T& t_;
};
struct detail::filter_close_all_operation<T,Sink> { // no copy
    ctor(T& t, Sink& snk);
    void operator()() const { close_all(t_, snk_); }
private: T& t_; Sink& snk_;
};
device_close_all_operation<T> detail::call_close_all<T>(T& t);
filter_close_all_operation<T,Sink> detail::call_close_all<T,Sink>(T& t, Sink& snk);

struct detail::member_close_operation<T> { // no copy
    ctor(T& t, ios::openmode which);
    void operator()() const { t_.close(which_); }
private: T& t_; ios::openmode which_;
};
member_close_operation<T> detail::call_member_close<T>(T& t, ios::openmode which);

struct detail::reset_operation<T> {
    ctor(T& t);
    void operator()() const { t_.reset(); }
private: T& t_;
};
reset_operation<T> detail::call_reset<T>(T& t);

struct detail::clear_flags_operation<T> {
    ctor(T& t);
    void operator()() const { t_ = 0; }
private: T& t_;
};
clear_flags_operation<T> detail::clear_flags<T>(T& t);

struct detail::flush_buffer_operation<Buffer,Device> {
    ctor(Buffer& buf, Device& dev, bool flush);
    void operator()() const { if (flush_) buf_.flush(dev_); }
private: Buffer& buf_; Device& dev_; bool flush_;
};
flush_buffer_operation<Buffer,Device> detail::flush_buffer<Buffer,Device>(Buffer& buf, Device& dev, bool flush);

struct detail::is_dereferenceable<T>;
struct is_iterator_range<T>;
struct detail::newline<Ch>; // specialized for char and wchar_t

struct detail::aligned_storage<T> {
    void <const>* address() <const>;
    union {char data[sizeof(T)]; type_with_alignment<alignment_of<T>::value>::type aligner_;} dummy_;
};
struct detail::optional<T> { // no copy
    using element_type = T;
    ctor(){} ctor(const T& t){reset(t);} ~dtor() {reset();}
    <const> T& operator*() <const>;
    <const> T* operator->() <const>;
    <const> T* get() <const>;
    void reset() { if (initialized) {get()->T::~T(); initialized_=false;} }
    void reset(const T& t) { reset(); new (address()) T{t}; initialized_=true; }
private: aligned_storage<T> storage_; bool initialized_{false};
    <const> void* address() <const> { return &storage_; }
};

struct detail::path {
    ctor(); ctor(const std::string& p); ctor(const char* p); // narrow
    explicit ctor<Path>(const Path& p, Path::external_string_type*=0) { init(p.external_file_string()); }
    explicit ctor<Path>(const Path& p, Path::codecvt_type*=0) { init(p.native()); }
    ctor(const self& p);
    self& operator=(const self& p); self& operator=(const std::string& p); self& operator=(const char* p);
    self& operator=<Path>(const Path& p) requires Path::external_string_type {init(p.external_file_string()); return *this; }
    self& operator=<Path>(const Path& p) requires Path::codecvt_type {init(p.native()); return *this; }
    bool is_wide() const;
    const char* c_str() const; const wchar_t* c_wstr() const;
private: // no init from wstring
    void init(std::<w>string const& file_path);
    std::string narrow_; std::wstring wide_; bool is_wide_;
};
bool operator==(const path& lhs, const path& rhs);

struct detail::resolve_traits<Mode,Ch,T>;
resolve_traits<Mode,Ch,T>::type             detail::resolve<Mode,Ch,T>(const T& t);
mode_adapter<Mode,std::basic_streambuf<Ch,Tr>> detail::resolve<Mode,Ch,Tr>(std::basic_streambuf<Ch,Tr>& sb);
mode_adapter<Mode,std::basic_istream<Ch,Tr>> detail::resolve<Mode,Ch,Tr>(std::basic_istream<Ch,Tr>& is);
mode_adapter<Mode,std::basic_ostream<Ch,Tr>> detail::resolve<Mode,Ch,Tr>(std::basic_ostream<Ch,Tr>& os);
mode_adapter<Mode,std::basic_iostream<Ch,Tr>> detail::resolve<Mode,Ch,Tr>(std::basic_iostream<Ch,Tr>& io);
array_adapter<Mode,Ch>                      detail::resolve<Mode,Ch,n>(Ch (&array)[n]);
range_adapter<Mode,iterator_range<It>>      detail::resolve<Mode,Ch,It>(const iterator_range<It>& rng);

struct detail::restricted_indirect_device<Device>: public device_adapter<Device> {
    using param_type = param_type<Device>::type;
    using char_type = char_type_of<Device>::type; using mode = mode_of<Device>::type;
    struct category : public mode, device_tag, closable_tag, flushable_tag, localizable_tag, optimally_buffered_tag {};
    ctor(param_type dev, stream_offset off, stream_offset len=-1);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    streampos seek(stream_offset off, ios::seekdir way);
private: stream_offset beg_, pos_, end_;
};
struct detail::restricted_direct_device<Device>: public device_adapter<Device> {
    using char_type = char_type_of<Device>::type; using mode = mode_of<Device>::type;
    using pair_type = std::pair<char_type*, char_type*>;
    struct category : public mode, device_tag, direct_tag, closable_tag, localizable_tag {};
    ctor(const Device& dev, stream_offset off, stream_offset len=-1);
    pair_type input_sequence();
    pair_type output_sequence();
private: char_type *beg_, *end_;
};
struct detail::restricted_filter<Filter> : public filter_adapter<Filter> {
    using char_type = char_type_of<Device>::type; using mode = mode_of<Device>::type;
    struct category : public mode, filter_tag, multichar_tag, closable_tag, localizable_tag, optimally_buffered_tag {};
    ctor(const Filter& flt, stream_offset off, stream_offset len=-1);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
    streampos seek<Device>(Device& dev, stream_offset off, ios::seekdir way);
    void close<Device>(Device& dev, <ios::openmode which>);
private: stream_offset beg_, pos_, end_; bool open_;
    void open<Device>(Device& dev, ios::openmode which);
};
struct detail::restriction_traits<T>
    : select<is_filter<T>, restricted_filter<T>, is_direct<T>, restricted_direct_device<T>, else_, restricted_indirect_device<T>>{};
struct restriction<T> : public restriction_traits<T>::type {};

struct detail::wrapped_type<T> : if_<is_std_io<T>,reference_wrapper<T>,T> {};
struct detail::unwrapped_type<T> : unwrap_reference<T> {};
struct detail::unwrap_ios<T> : eval_if<is_std_io<T>, unwrap_reference<T>, identity<T>> {};
T detail::wrap<T>(const T& t) requires !is_std_io<T>;
wrapped_type<T>::type detail::wrap<T>(const T& t) requires is_std_io<T>;
unwrapped_type<T>::type& detail::unwrap<T>(const reference_wrapper<T>& ref);
<const> unwrapped_type<T>::type& detail::unwrap<T>(<const> T& t);
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/config/auto_link.hpp>`
* `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`

#### Boost.Core

* `<boost/checked_delete.hpp>`
* `<boost/core/type.hpp>`
* `<boost/core/typeinfo.hpp>`
* `<boost/noncopyable.hpp>`
* `<boost/ref.hpp>`
* `<boost/utility/enable_if.hpp>`, `<boost/core/enable_if.hpp>`

#### Boost.Detail

* `<boost/detail/is_incrementable.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.Integer

* `<boost/integer_traits.hpp>`

#### Boost.Iterator

* `<boost/next_prior.hpp>`

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Numeric/Conversion

* `<boost/numeric/conversion/cast.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.Random

* `<boost/random/linear_congruential.hpp.hpp>`
* `<boost/random/uniform_smallint.hpp>`

#### Boost.Range

* `<boost/range/iterator_range.hpp>`
* `<boost/range/value_type.hpp>`

#### Boost.Regex

* `<boost/regex.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`
* `<boost/utility/base_from_member.hpp>`
* `<boost/utility/result_of.hpp>`

------
### Standard Facilities
