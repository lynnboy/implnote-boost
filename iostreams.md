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

#### Categories

```c++
const streamsize default_device_buffer_size = 4096, default_filter_buffer_size = 128, default_pback_buffer_size = 4;

struct any_tag{};
struct detail::two_sequence : virtual any_tag{}; struct detail::random_access : virtual any_tag{};
struct detail::one_head : virtual any_tag{}; struct detail::two_head : virtual any_tag{};
struct input : virtual any_tag{}; struct output : virtual any_tag{};
struct bidirectional : virtual input, virtual output, two_sequence{}; struct dual_use : virtual input, virtual output{};
struct input_seekable : virtual input, virtual random_access{}; struct output_seekable : virtual output, virtual random_access{};
struct seekable : virtual input_seekable, virtual output_seekable, one_head{};
struct dual_seekable : virtual input_seekable, virtual output_seekable, two_head{};
struct bidirectional_seekable : input_seekable, output_seekable, bidirectional, two_head{};

struct device_tag : virtual any_tag{}; struct filter_tag : virtual any_tag{};

struct peekable_tag : virtual any_tag{}; struct closable_tag : virtual any_tag{};
struct flushable_tag : virtual any_tag{}; struct localizable_tag : virtual any_tag{};
struct optimally_buffered_tag : virtual any_tag{};
struct direct_tag : virtual any_tag{}; struct multichar_tag : virtual any_tag{};

struct source_tag : device_tag, input{}; struct sink_tag : device_tag, output{};
struct bidirectional_device_tag : device_tag, bidirectional{};
struct seekable_device_tag : virtual device_tag, seekable {};

struct input_filter_tag : filter_tag, input{}; struct output_filter_tag: filter_tag, output{};
struct bidirectional_filter_tag : filter_tag, bidirectional{};
struct seekable_filter_tag : filter_tag, seekable{};
struct dual_use_filter_tag : filter_tag, dual_use{};

struct multichar_input_filter_tag : multichar_tag, input_filter_tag{};
struct multichar_output_filter_tag : multichar_tag, output_filter_tag{};
struct multichar_bidirectional_filter_tag : multichar_tag, bidirectional_filter_tag{};
struct multichar_seekable_filter_tag : multichar_tag, seekable_filter_tag{};
struct multichar_dual_use_filter_tag : multichar_tag, dual_use_filter_tag{};

struct std_io_tag : virtual localizable_tag{};

struct istream_tag : virtual device_tag, virtual peekable_tag, virtual std_io_tag{};
struct ostream_tag : virtual device_tag, virtual std_io_tag{};
struct iostream_tag : istream_tag, ostream_tag{};
struct streambuf_tag : device_tag, peekable_tag, std_io_tag{};

struct ifstream_tag : input_seekable, closable_tag, istream_tag{};
struct ofstream_tag : output_seekable, closable_tag, ostream_tag{};
struct fstream_tag : seekable, closable_tag, iostream_tag{};
struct filebuf_tag : seekable, closable_tag, streambuf_tag{};

struct istringstream_tag : input_seekable, istream_tag{};
struct ostringstream_tag : output_seekable, ostream_tag{};
struct stringstream_tag : dual_seekable, iostream_tag{};
struct stringbuf_tag : dual_seekable, streambuf_tag{};

struct generic_istream_tag : input_seekable, istream_tag{};
struct generic_ostream_tag : output_seekable, ostream_tag{};
struct generic_iostream_tag : seekable, iostream_tag{};
struct generic_streambuf_tag : seekable, streambuf_tag{};
```

#### Traits

```c++
struct is_istream<T>; struct is_ostream<T>; struct is_iostream<T>; struct is_streambuf<T>;
struct is_istringstream<T>; struct is_ostringstream<T>; struct is_stringstream<T>; struct is_stringbuf<T>;
struct is_ifstream<T>; struct is_ofstream<T>; struct is_fstream<T>; struct is_filebuf<T>;
struct is_std_io<T>; struct is_std_file_device<T>; struct is_std_string_device<T>;

struct char_type_of<T>; struct category_of<T>; struct int_type_of<T>; struct mode_of<T>;

struct is_device<T>; struct is_filter<T>; struct is_direct<T>;

struct detail::is_boost_stream<T>; struct detail::is_boost_stream_buffer<T>;
struct detail::is_filtering_stream<T>; struct detail::is_filtering_streambuf<T>;
struct detail::is_linked<T>;
struct detail::is_boost<T>;
```

#### Stream

```c++
struct detail::stream_buffer_traits<T,Tr,Alloc,Mode>; // category_of<T>::type=>direct_tag ? direct_streambuf : indirect_streambuf
struct stream_buffer<T,Tr=std::char_traits<char_type>, Alloc=std::allocator<char_type>, Mode=mode_of<T>::type> : streambuffer_traits<T,Tr,Alloc,Mode>::type {
    using char_type = char_type_of<T>::type; using traits_type = Tr;
    using int_type = traits_type::int_type; using off_type = traits_type::off_type; using pos_type = traits_type::pos_type;
    struct category : Mode, closable_tag, streambuf_tag{};
    ctor(); ~dtor();
    ctor(<const> T& t, std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    ctor(const reference_wrapper<T>& ref , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void open(<const> T& t , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void open(const reference_wrapper<T>& ref , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1)
    ctor<U,[U1],[U2]>(<const> U &u0, <const U1& u1>, <const U2& u2>) requires !same_as<U,T>;
    void open<U,[U1],[U2]>(<const> U &u, <const U1& u1>, <const U2& u2>) requires !same_as<U,T>;
    T& operator*(); T* operator->();
private: void open_impl(const T& t, std::streamsize buffer_size = -1 , std::streamsize pback_size = -1); // throw if is_open()
};

struct detail::stream_traits<Device,Tr> {
    using char_type = char_type_of<Device>::type; using traits_type = Tr; using mode = category_of<Device>::type;
    using stream_type = select<and_<is_convertible<mode,input>,is_convertible<mode,output>>, std::basic_iostream<char_type,traits_type>,
            is_convertible<mode,input>, std::basic_istream<char_type,traits_type>,
            else_, std::basic_ostream<char_type,traits_type> >::type;
    using stream_tag = select<and_<is_convertible<mode,input>,is_convertible<mode,output>>, iostream_tag,
            is_convertible<mode,input>, istream_tag,
            else_, ostream_tag >::type;
};
struct detail::stream_base<Device,Tr=std::char_traits<char_type>,Alloc=std::allocator<char_type>>
    : protected base_from_member<stream_buffer<Device,Tr,Alloc>>, public stream_traits<Device,Tr>::stream_type {
    ctor(): base_from_member{}, stream_type{&base_from_member::member} {}
};
struct stream<Device,Tr=std::char_traits<char_type>,Alloc=std::allocator<char_type>> : stream_base<Device,Tr,Alloc> {
    using char_type = char_type_of<Device>::type;
    struct category : mode_of<Device>::type, closable_tag, stream_traits<Device,Tr>::stream_tag {};
    using stream_type = stream_traits<Device,Tr>::stream_type;
    ctor();
    ctor(<const> T& t, std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    ctor(const reference_wrapper<T>& ref , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void open(<const> T& t , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void open(const reference_wrapper<T>& ref , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1)
    ctor<U,[U1],[U2]>(<const> U &u0, <const U1& u1>, <const U2& u2>) requires !same_as<U,T>;
    void open<U,[U1],[U2]>(<const> U &u, <const U1& u1>, <const U2& u2>) requires !same_as<U,T>;
    bool is_open() const { return member.is_open(); }
    void close() { member.close(); }
    bool auto_close() const { return member.auto_close(); }
    void set_auto_close(bool close) { member.set_auto_close(close); }
    bool strict_sync() { return member.strict_sync(); }
    Device& operator*() { return *member; } Device* operator->() { return &*member; }
    Device* component() { return member.component(); }
private: void open_impl(const T& t, std::streamsize buffer_size = -1 , std::streamsize pback_size = -1); // clear() and member.open()
};
```

#### Filter Stream

------
### Details

#### Common

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

#### Adapters

```c++
struct detail::device_wrapper_impl<any_tag>{
    static streampos seek<Device,Dummy>(Device& dev, Dummy*, stream_offset off, ios::seekdir way, ios::openmode which) {return seek(dev,off,way,which,category_of<Device>::type{}); }
    static streampos seek<Device>(Device&, stream_offset, ios::seekdir, ios::openmode, any_tag); // throw_exception(cant_seek())
    static streampos seek<Device>(Device& dev, stream_offset off, ios::seekdir way, ios::openmode which, random_access);
    static void close<Device,Dummy>(Device& dev, Dummy*, ios::openmode which);
    static bool flush<Device,Dummy>(Device& dev, Dummy*);
};
struct detail::device_wrapper_impl<input> : device_wrapper_impl<any_tag> {
    static streamsize read<Device,Dummy>(Device& dev, Dummy*, char_type_of<Device>::type* s, streamsize n);
    static streamsize write<Device,Dummy>(Device&, Dummy*, const char_type_of<Device>::type*, streamsize); // throw_exception(cant_write())
};
struct detail::device_wrapper_impl<output> {
    static streamsize read<Device,Dummy>(Device&, Dummy*, char_type_of<Device>::type*, streamsize); // throw_exception(cant_read())
    static streamsize write<Device,Dummy>(Device& dev, Dummy*, const char_type_of<Device>::type* s, streamsize n);
};

struct detail::flt_wrapper_impl<any_tag>{
    static streampos seek<Filter,Device>(Filter& f, Device* dev, stream_offset off, ios::seekdir way, ios::openmode which) {return seek(f,dev,off,way,which,category_of<Filter>::type{}); }
    static streampos seek<Filter,Device>(Filter&,Device*, stream_offset, ios::seekdir, ios::openmode, any_tag); // throw_exception(cant_seek())
    static streampos seek<Filter,Device>(Filter& f, Device* dev, stream_offset off, ios::seekdir way, ios::openmode which, random_access tag) {return seek(f,dev,off,way,which,tag,category_of<Filter>::type{}); }
    static streampos seek<Filter,Device>(Filter& f, Device* dev, stream_offset off, ios::seekdir way, ios::openmode, random_access, any_tag) {return f.seek(*dev,off,way);}
    static streampos seek<Filter,Device>(Filter& f, Device* dev, stream_offset off, ios::seekdir way, ios::openmode which, random_access, two_sequence) {return f.seek(*dev,off,way,which);}
    static void close<Filter,Device>(Filter& f, Device* dev, ios::openmode which);
    static bool flush<Filter,Device>(Filter& f, Device* dev);
};
struct detail::flt_wrapper_impl<input> {
    static streamsize read<Filter,Source>(Filter& f, Source* src, char_type_of<Filter>::type* s, streamsize n);
    static streamsize write<Filter,Sink>(Filter&, Sink*, const char_type_of<Filter>::type*, streamsize); // throw_exception(cant_write())
};
struct detail::flt_wrapper_impl<output> {
    static streamsize read<Filter,Source>(Filter&, Source*, char_type_of<Filter>::type*, streamsize); // throw_exception(cant_read())
    static streamsize write<Filter,Sink>(Filter& f, Sink* snk, const char_type_of<Filter>::type* s, streamsize n);
};

class detail::concept_adapter<T> { // no copy-assign
    using value_type = value_type<T>::type;
    using input_tag = dispatch<T,input,output>::type; using output_tag = dispatch<T,output,input>::type;
    using {input|output|any}_impl = if_<is_device<T>,device_wrapper_impl<{input|output|any}_tag>, flt_wrapper_impl<{input|output|any}_tag>>::type;
public: using char_type = char_type_of<T>::type; using category=category_of<T>::type;
    explicit ctor(const reference_wrapper<T>& ref); explicit ctor(const T& t);
    T& operator*(); T* operator->();
    streamsize read(char_type* s, streamsize n); streamsize read<Source>(char_type* s, streamsize n, Source* src); // input_impl::read
    streamsize write(const char_type* s, streamsize n); streamsize write<Sink>(const char_type* s, streamsize n, Sink* snk); // output_impl::write
    streampos seek(stream_offset off, ios::seekdir way, ios::openmode which);
    streampos seek<Device>(stream_offset off, ios::seekdir way, ios::openmode which, Device* dev); // any_impl::seek
    void close(ios::openmode which); void close<Device>(ios::openmode which, Device* dev); // any_impl::close
    bool flush<Device>(Device* dev); // any_impl::flush
    void imbue<Locale>(const Locale& loc);
    streamsize optimal_buffer_size() const; 
private: value_type t_;
};

class detail::device_adapter<T> {
    using value_type = value_type<T>::type; using param_type = param_type<T>::type;
public: explicit ctor(param_type t);
    T& component();
    void close(); void close(ios::openmode which);
    bool flush();
    void imbue<Locale>(const Locale& loc);
    streamsize optimal_buffer_size() const;
    value_type t_;
};

class detail::device_adapter<T> {
    using value_type = value_type<T>::type; using param_type = param_type<T>::type;
public: explicit ctor(param_type t);
    T& component();
    void close<Device>(Device& dev); void close<Device>(Device& dev, ios::openmode which);
    bool flush<Device>(Device& dev);
    void imbue<Locale>(const Locale& loc);
    streamsize optimal_buffer_size() const;
    value_type t_;
};

struct detail::direct_adapter_base<Direct> {
    using char_type = char_type_of<Direct>::type; using mode_type = mode_of<Direct>::type;
    struct category : public mode_type, device_type, closable_tag, localizable_tag{};
protected: explicit ctor(const Direct& d);
    using is_double = is_convertible<category, two_sequence>;
    struct pointers {char_type *beg, *ptr, *end;};
    double_object<pointers,is_double> ptrs_; Direct d_;
};
struct detail::direct_adapter<Direct> : private direct_adapter_base<Direct> {
    ctor(const Direct& d); ctor(const self& d); ctor<...P>(const P&...p);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    streampos seek(stream_offset, ios::seekdir, ios::openmode = ios::in|ios::out);
    void close(); void close(ios::openmode which);
    void imbue(const std::locale&);
    Direct& operator*(); Direct* operator->();
};

struct detail::wrap_direct_traits<Device> : if_<is_direct<Device>, direct_adapter<Device>, Device>{};
wrap_direct_traits<Device>::type detail::wrap_direct<Device>(Device dev);
Device& detail::unwrap_direct<Device>(Device& d);
Device& detail::unwrap_direct<Device>(direct_adapter<Device>& d);

class detail::mode_adapter<Mode,T> {
    struct empty_base{};
public: using component_type = wrapped_type<T>::type; using char_type = char_type_of<T>::type;
    struct category : public Mode, device_tag, if_<is_filter<T>,filter_tag,device_tag>, if_<is_filter<T>,multichar_tag,empty_base>, closable_tag, localizable_tag{};
    explicit ctor(const component_type& t);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    streampos seek(stream_offset off, ios::seekdir way, ios::openmode which = ios::in|ios::out);
    void close(); void close(ios::openmode which);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize read<Sink>(Sink& snk, const char_type* s, streamsize n);
    streampos seek<Device>(Device& dev, stream_offset off, ios::seekdir way, <ios::openmode which>);
    void close<Device>(Device& dev); void close<Device>(Device& dev, ios::openmode which);
    void imbue<Locale>(const Locale& loc);
private: component_type t_;
};

struct non_blocking_adapter<Device> { // no copy-assign
    using char_type = char_type_of<Device>::type;
    struct category : public mode_of<Device>::type, device_tag {};
    explicit ctor(Device& dev);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    streampos seek(stream_offset off, ios::seekdir way, ios::openmode which = ios::in|ios::out);
private: Device& device_;
};

struct detail::output_iterator_adapter<Mode,Ch,OutIt> {
    using char_type = Ch; using category = sink_tag;
    explicit ctor(OutIt out);
    streamsize write(const char_type* s, streamsize n);
private: OutIt out_;
};

struct detail::range_adapter_impl<std::forward_iterator_tag> {
    static streamsize read<It,Ch>(It& cur, It& last, Ch* s, streamsize n);
    static streamsize write<It,Ch>(It& cur, It& last, const Ch* s, streamsize n);
};
struct detail::range_adapter_impl<std::random_access_iterator_tag> {
    static streamsize read<It,Ch>(It& cur, It& last, Ch* s, streamsize n);
    static streamsize write<It,Ch>(It& cur, It& last, const Ch* s, streamsize n);
    static void seek<It>(It& first, It& cur, It& last, stream_offset off, ios::seekdir way);
};
class detail::range_adapter<Mode,Range> {
    using iterator = Range::iterator; using iter_traits = std::iterator_traits<iterator>; using iter_cat = iter_traits::iterator_category;
public: using char_type = Range::value_type;
    struct category : public Mode, device_tag {};
    using tag = if_<is_convertible<iter_cat,std::random_access_iterator_tag>, random_access_iterator_tag, forward_iterator_tag>::type;
    using impl = range_adapter_impl<tag>;
    explicit ctor(const Range& rng); ctor(iterator first, iterator last);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    streampos seek(stream_offset off, ios::seekdir way);
private: iterator first_, cur_, last_;
};
```

#### Stream Buffers

```c++
class detail::linked_streambuf<Ch,Tr=std::char_traits<Ch>> : public std::basic_streambuf<Ch,Tr> {
    enum flag_type { f_true_eof=1, f_input_closed=2, f_output_closed=4 };
    int flags_;
protected: ctor() :flags_{0}{}
    void set_true_eof(bool eof) { flags_ = (flags_&~f_true_eof) | (eof?f_true_eof:0); }
    void close(ios::openmode which) {
        if (which==ios::in && (flags_&f_input_closed)==0) { flags_ |= f_input_closed; close_impl(which); }
        if (which==ios::out && (flags_&f_output_closed)==0) { flags_ |= f_output_closed; close_impl(which); }
    }
    void set_needs_close() { flags_ &= ~(f_input_closed|f_output_closed); }
    virtual void set_next(self*){}
    virtual void close_impl(ios::openmode) =0;
    virtual bool auto_close() const =0;
    virtual void set_auto_close(bool) =0;
    virtual bool strict_sync() =0;
    virtual const typeinfo& component_type() const =0;
    virtual void* component_impl() =0;
public: bool true_eof() const { return flags_&f_true_eof; }
};

class detail::chainbuf<Chain,Mode,Access> : public std::basic_streambuf<Chain::char_type,Chain::traits_type>,
    public access_control<Chain::client_type,Access>, noncopyable {
    using client_type = access_control<chain_client<Chain>,Access>;
public: using char_type = Chain::char_type; using traits_type = Chain::traits_type;
    using int_type = traits_type::int_type; using off_type = traits_type::off_type; using pos_type = traits_type::pos_type;
protected: using delegate_type = linked_streambuf<char_type,traits_type>;
    ctor() { client_type::set_chain(&chain_); }
    int_type underflow() { sentry t{this}; return translate(delegate().underflow()); }
    int_type pbackfail(int_type c) { sentry t{this}; return translate(delegate().pbackfail(c)); }
    streamsize xsgetn(char_type* s, streamsize n) { sentry t{this}; return delegate().xsgetn(s,n); }
    int_type overflow(int_type c) { sentry t{this}; return translate(delegate().overflow(c)); }
    streamsize xsputn(const char_type* s, streamsize n) { sentry t{this}; return delegate().xsputn(s,n); }
    int sync() { sentry t{this}; return delegate().sync(); }
    pos_type seekoff(off_type off, ios::seekdir way, ios::openmode which=ios::in|ios::out) { sentry t{this}; return delegate().seekoff(off,way,which); }
    pos_type seekpos(pos_type sp, ios::openmode which=ios::in|ios::out) { sentry t{this}; return delegate().seekpos(sp,which); }
private: using std_traits = std::char_traits<char_type>; using chain_traits = Chain::traits_type;
    static chain_traits::int_type translate(std::traits::int_type c) { return translate_int_type<std_traits,chain_traits>(c); }
    delegate_type& delegate() { return (delegate_type&)chain_.front(); }
    void get_pointers() { setg(delegate().epback(), delegate().gptr(), delegate().egptr());
        setp(delegate().pbase(), delegate().epptr()); pbump((int)(delegate().pptr()-delegate().pbase())); }
    void set_pointers() { delegate().setg(eback(),gptr(),egptr()); delegate().setp(pbase(),epptr()); delegate().pbump((int)(pptr()-pbase())); }
    struct sentry {ctor(chainbuf* buf):buf_{buf}{buf_->set_pointers();} ~dtor(){buf_->get_pointers();} chainbuf* buf_; };
    Chain chain_;
};

class detail::direct_streambuf<T,Tr=std::char_traits<char_type_of<T>::type>> : public linked_streambuf<char_type_of<T>::type,Tr> {
    using category = category_of<T>::type; using streambuf_type = std::basic_streambuf<char_type,traits_type>;
    optional<T> storage_;
    char_type *ibeg_, *iend_, *obeg_, *oend_;
    bool auto_close_;
public: using char_type = char_type_of<T>::type; using traits_type = Tr;
    using int_type = traits_type::int_type; using off_type = traits_type::off_type; using pos_type = traits_type::pos_type;
    void open(const T& t, streamsize buffer_size, streamsize pback_size);
    bool is_open() const;
    void close();
    bool auto_close() const override { return auto_close_; }
    void set_auto_cose(bool close) override { auto_close_ = close; }
    bool strict_sync() override { return true; }
    T* component() { return storage_.get(); }
protected: ctor();
    void close_impl(ios::openmode) override;
    const typeinfo& component_type() const override { return typeid(T); }
    void* component_impl() override { return component(); }
};

class detail::indirect_streambuf<T,Tr,Alloc,Mode> : public linked_streambuf<char_type_of<T>::type,Tr> {
    using category = category_of<T>::type; using wrapper = concept_adapter<T>;
    using buffer_type = basic_buffer<char_type,Alloc>; using streambuf_type = std::basic_streambuf<char_type,traits_type>;

    enum flag_type { f_open=1, f_output_buffered=2, f_auto_close=4 };
    optional<T> storage_; streambuf_type* next_;
    double_object<buffer_type,is_convertible<Mode,two_sequence>> buffer_;
    streamsize pback_size_; int flags_;
public: using char_type = char_type_of<T>::type; using traits_type = Tr;
    using int_type = traits_type::int_type; using off_type = traits_type::off_type; using pos_type = traits_type::pos_type;
    ctor();
    void open(const T& t, streamsize buffer_size=-1, streamsize pback_size=-1);
    bool is_open() const;
    void close();
    bool auto_close() const override;
    void set_auto_cose(bool close) override;
    bool strict_sync() override;
    T* component() { return &*obj(); }
protected:
    void imbue(const std::locale& loc);
    void set_next(streambuf_type* next) override;
    void close_impl(ios::openmode) override;
    const typeinfo& component_type() const override { return typeid(T); }
    void* component_impl() override { return component(); }
};
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
