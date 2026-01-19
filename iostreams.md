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

// archetypes
struct device<Mode,Ch=char> {
    using char_type = Ch;
    struct category : Mode, device_tag, closable_tag, localizable_tag {};
    void close(); void close(ios::openmode);
    void imbue<Locale>(const Locale&);
};
struct wdevice<Mode,Ch=wchar_t> : device<Mode,Ch>{};
using source=device<input>; using wsource=wdevice<input>;
using sink=device<output>; using wsink=wdevice<output>;

struct filter<Mode,Ch=char> {
    using char_type = Ch;
    struct category : Mode, filter_tag, closable_tag, localizable_tag {};
    void close<Device>(Device&); void close<Device>(Device&, ios::openmode);
    void imbue<Locale>(const Locale&);
};
struct wfilter<Mode,Ch=wchar_t> : filter<Mode,Ch>{};
using input_filter=filter<input>; using input_wfilter=wfilter<input>;
using output_filter=filter<output>; using output_wfilter=wfilter<output>;
using seekable_filter=filter<seekable>; using seekable_wfilter=wfilter<seekable>;
using dual_use_filter=filter<dual_use>; using dual_use_wfilter=wfilter<dual_use>;
struct multichar_filter<Mode,Ch=char> : fitler<Mode,Ch> { struct category : base::category, multichar_tag{}; };
struct multichar_wfilter<Mode,Ch=wchar_t> : multichar_filter<Mode,Ch>{};
using multichar_input_filter=multichar_filter<input>; using multichar_input_wfilter=multichar_wfilter<input>;
using multichar_output_filter=multichar_filter<output>; using multichar_output_wfilter=multichar_wfilter<output>;
using multichar_dual_use_filter=multichar_filter<dual_use>; using multichar_dual_use_wfilter=multichar_wfilter<dual_use>;
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

```c++
struct filtering_<w>streambuf<Mode,Ch={char|wchar_t},Tr=std::char_traits<Ch>,Alloc=std::allocator<Ch>,Access=public_>
    : public chainbuf<chain<Mode,Ch,Tr,Alloc>,Mode,Access> {
    using char_type=Ch; using mode=Mode; using chain_type = chain<Mode,Ch,Tr,Alloc>;
    struct category : Mode, closable_tag, streambuf_tag {};
    ctor(); ~dtor();
    explicit ctor<C,T>(std::basic_streambuf<C,T>& sb, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<C,T>(std::basic_istream<C,T>& is, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<C,T>(std::basic_ostream<C,T>& os, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<C,T>(std::basic_iostream<C,T>& io, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<It>(const iterator_range<It>& rng, std::streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<Pipeline,Concept>(const pipeline<Pipeline,Concept>& p);
};
using filtering_istreambuf = filtering_streambuf<input>;
using filtering_ostreambuf = filtering_streambuf<output>;
using filtering_wistreambuf = filtering_wstreambuf<input>;
using filtering_wostreambuf = filtering_wstreambuf<output>;

struct detail::filtering_stream_traits<Mode,Ch,Tr> {
    using stream_type = select<and_<is_convertible<Mode,input>,is_convertible<Mode,output>>, std::basic_iostream<Ch,Tr>,
            is_convertible<Mode,input>, std::basic_istream<Ch,Tr>,
            else_, std::basic_ostream<Ch,Tr> >::type;
    using stream_tag = select<and_<is_convertible<Mode,input>,is_convertible<Mode,output>>, iostream_tag,
            is_convertible<Mode,input>, istream_tag,
            else_, ostream_tag >::type;
};
struct detail::filtering_stream_base<Chain,Access> : public access_control<chain_client<Chain>,Access>,
    public filtering_stream_traits<Chain::mode,Chain::char_type,Chain::traits_type>::stream_type {
    using chain_type = Chain; using client_type = access_control<...>;
protected: using stream_type = filtering_stream_traits<...>::stream_type;
    ctor() : base{0} { set_chain(&chain_); }
private: void notify() { rdbuf(chain_.empty()?0:&chain_.front()); }
    Chain chain_;
};

struct <w>filtering_stream<Mode,Ch={char|wchar_t},Tr=std::char_traits<Ch>,Alloc=std::allocator<Ch>,Access=public_>
    : public filtering_stream_base<chain<Mode,Ch,Tr,Alloc>,Access> {
    using char_type=Ch; using mode=Mode; using chain_type = chain<Mode,Ch,Tr,Alloc>; using traits_type=Tr;
    using int_type = traits_type::int_type; using off_type = traits_type::off_type; using pos_type = traits_type::pos_type;
    struct category : Mode, closable_tag, filtering_stream_traits<...>::stream_tag {};
    ctor(); ~dtor();
    explicit ctor<C,T>(std::basic_streambuf<C,T>& sb, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<C,T>(std::basic_istream<C,T>& is, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<C,T>(std::basic_ostream<C,T>& os, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<C,T>(std::basic_iostream<C,T>& io, streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<It>(const iterator_range<It>& rng, std::streamsize buffer_size = -1, std::streamsize pback_size = -1);
    explicit ctor<Pipeline,Concept>(const pipeline<Pipeline,Concept>& p);
private: using client_type = access_control<...>;
    void push_impl<T>(const T& t , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1); // client_type::push
};
using filtering_istream = filtering_stream<input>;
using filtering_ostream = filtering_stream<output>;
using filtering_wistream = wfiltering_stream<input>;
using filtering_wostream = wfiltering_stream<output>;
```

#### Chain

```c++
using stream_offset = intmax_t;
std::streamoff stream_offset_to_streamoff(stream_offset off);
std::streampos offset_to_position(stream_offset off);
stream_offset position_to_offset<PosType>(PosType pos);
stream_offset position_to_offset(std::streampos pos);

struct detail::chain_base<Self,Ch,Tr,Alloc,Mode> {
    using char_type = Ch; using traits_type = Tr; using allocator_type = Alloc; using mode = Mode;
    using int_type = traits_type::int_type; using off_type = traits_type::off_type; using pos_type = traits_type::pos_type;
    struct category : Mode, device_tag{};
    using client_type = chain_client<Self>;
private: using streambuf_type = linked_streambuf<Ch>; using list_type = std::list<streambuf_type*>;
protected: ctor(); ctor(const self& rhs);
public: void set_device_buffer_size(streamsize n);
    void set_filter_buffer_size(streamsize n);
    void set_pback_size(streamsize n);
    streamsize read(char_type* streamsize n);
    streamsize write(const char_type* streamsize n);
    streamsize seek(stream_offset off, ios::seekdir way);
    const typeinfo& component_type(int n) const; const typeinfo& component_type<n>() const;
    T* component<T>(int n) const; T* component<n,T>() const;
private: T* component<T>(int n, boost::type<T>) const;
public: using size_type = list_type::size_type;
    streambuf_type& front();
    void push<C,T>(std::basic_streambuf<C,T>& sb, std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<C,T>(std::basic_istream<C,T>& is , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<C,T>(std::basic_ostream<C,T>& os , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<C,T>(std::basic_iostream<C,T>& io , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<It>(const iterator_range<It>& rng , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<Pipeline,Concept>(const pipeline<Pipeline,Concept>& p);
    void push<T>(const T& t , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1) requires !is_std_io<T>;
    void pop();
    bool empty() const;
    size_type size() const;
    void reset();

    bool is_complete() const;
    bool auto_close() const;
    void set_auto_close(bool close);
    bool sync();
    bool strict_sync();
private: void push_impl<T>(const T& t, std::streamsize buffer_size = -1, std:streamsize pback_size = -1);
    <const> list_type& list() <const>;
    void register_client(client_type* client);
    void notify();
    static void close(streambuf_type* b, ios::openmode m);
    static void set_next(streambuf_type* b, streambuf_type* next);
    static void set_auto_close(streambuf_type* b, bool close);
    struct closer { void operator()(streambuf_type* b); ios::openmode mode_; };
    enum flags { f_complete=1, f_open=2, f_auto_close=4 };
    struct chain_impl {
        ctor() ~dtor(); void close(); void reset();
        list_type links_; client_type* client_;
        std::streamsize device_buffer_size_, filter_buffer_size_, pback_size_;
        int flags_;
    };
    shared_ptr<chain_impl> pimpl_;
};

struct <w>chain<Mode,Ch={char|wchar_t},Tr=std::char_traits<Ch>,Alloc=std::allocator<Ch>>
    : public chain_base<self,Ch,Tr,Alloc,Mode> {
    struct category : device_tag, Mode{};
    using mode = Mode; using char_type = Ch; using traits_type = Tr;
    using int_type = traits_type::int_type; using off_type = traits_type::off_type;
    ctor(); ctor(const self& rhs); self& operator=(const self& rhs);
};

class detail::chain_client<Chain> {
    using chain_type = Chain; using char_type = Chain::char_type;
    using traits_type = Chain::traits_type; using size_type = Chain::size_type; using mode = Chain::mode;
    ctor(chain_type* chn=0); ctor(self* client);
    virtual ~dtor(){}
    const typeinfo& component_type(int n) const; const typeinfo& component_type<n>() const;
    T* component<T>(int n) const; T* component<n,T>() const;
    bool is_complete() const;
    bool auto_close() const; void set_auto_close(bool close);
    bool strict_sync();
    void set_device_buffer_size(streamsize n);
    void set_filter_buffer_size(streamsize n);
    void set_pback_size(streamsize n);
    void push<C,T>(std::basic_streambuf<C,T>& sb, std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<C,T>(std::basic_istream<C,T>& is , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<C,T>(std::basic_ostream<C,T>& os , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<C,T>(std::basic_iostream<C,T>& io , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<It>(const iterator_range<It>& rng , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    void push<Pipeline,Concept>(const pipeline<Pipeline,Concept>& p);
    void push<T>(const T& t , std::streamsize buffer_size = -1 , std::streamsize pback_size = -1) requires !is_std_io<T>;
    void pop();
    void empty() const;
    size_type size() const;
    void reset();
    chain_type filters() <const>;
protected: void push_impl<T>(const T& t, std::streamsize buffer_size = -1 , std::streamsize pback_size = -1);
    chain_type& ref();
    void set_chain(chain_type* c);
    virtual void notify(){}
private: chain_type* chain_;
};
```

#### Char Traits, Code Converter

```c++
const int WOULD_BLOCK=EOF-1; const std::wint_t WWOULD_BLOCK=WEOF-1;
struct char_traits<{char|wchar_t}> : std::char_traits<{char|wchar_t}> {
    static {char|wchar_t} newline();
    static {int|wint_t} good(), would_block();
    static bool is_good({int|wint_t} c), is_eof({int|wint_t} c), would_block({int|wint_t} c);
};

struct code_conversion_error: std::ios::failure { ctor(); };
struct detail::conversion_buffer<Codecvt,Alloc> : public buffer<codecvt_extern<Codecvt>::type,Alloc> {
    using state_type = Codecvt::state_type;
    ctor() : base{0}{reset();}
    state_type& state();
    void reset() {if(size()) set(0,0); state_={}; }
private: state_type state_;
};
struct detail::code_converter_impl<Device,Codecvt,Alloc> {
    using extern_type=codecvt_extern<Codecvt>::type; using device_category=category_of<Device>::type;
    using mode = select< is_convertible<device_category,bidirectional>, bidirectional,
                    is_convertible<device_category,input>, input,
                    is_convertible<device_category,output>, output>::type;
    using device_type = if_<is_direct<Device>,direct_adapter<Device>,Device>::type;
    using storage_type = optional<concept_adapter<device_type>>;
    using is_double = is_convertible<device_category,two_sequence>;
    using buffer_type = conversion_buffer<Codecvt,Alloc>;
    ctor(); ~dtor(); // close
    void open<T>(const T& dev, std::streamsize buffer_size);
    void close(); void close(ios::openmode which);
    bool is_open() const;
    device_type& dev();
    enum flag_type { f_open=1, f_input_closed=2, f_output_closed=4 };
    codecvt_holder<Codecvt> cvt_; storage_type dev_; double_object<buffer_type,is_double> buf_; int flags_;
};
struct code_converter_base<Device,Codecvt,Alloc> {
    using impl_type = code_converter_impl<Device,Codecvt,Alloc>;
    ctor();
    shared_ptr<impl_type> pimpl_;
};
class code_converter<Device,Codecvt=default_codecvt,Alloc=std::allocator<char>> : protected code_converter_base<Device,Codecvt,Alloc> {
    using device_type = impl_type::device_type; using buffer_type = impl_type::buffer_type;
    using codecvt_type = codecvt_holder<Codecvt>::codecvt_type;
    using {in|ex}tern_type = codecvt_{in|ex}tern<Codecvt>::type;
    using state_type = codecvt_state<Codecvt>::type;
public: using char_type = intern_type;
    struct category : impl_type::mode, device_tag, closable_tag, localizable_tag {};
    ctor();
    ctor(<const> Device& t, std::streamsize buffer_size = -1); ctor(const reference_wrapper<Device>& ref , std::streamsize buffer_size = -1);
    void open(<const> Device& t , std::streamsize buffer_size = -1); void open(const reference_wrapper<Device>& ref , std::streamsize buffer_size = -1)
    ctor<U,[U1],[U2]>(<const> U &u0, <const U1& u1>, <const U2& u2>) requires !same_as<U,Device>;
    void open<U,[U1],[U2]>(<const> U &u, <const U1& u1>, <const U2& u2>) requires !same_as<U,Device>;
    bool is_open() const;
    void close(ios::openmode which=ios::in|ios::out);
    std::streamsize read(char_type*, std::streamsize); std::streamsize write(const char_type*, std::streamsize);
    void imbue(const std::locale& loc);
    Device& operator*(); Device* operator->();
private: void open_impl<T>(const T& t, std::streamsize buffer_size = -1);
    const codecvt_type& cvt(); device_type& dev(); buffer_type & in(), out();
    impl_type& impl();
};
```

##### Operations

```c++
struct detail::custom_tag{};
struct detail::is_custom<T>; // !is_base_and_derived<custom_tag,operations<T>>
struct operations<T> : custom_tag{};

// operations
void close<T>(T& t);
void close<T>(T& t, ios::openmode which);
void close<T,Sink>(T& t, Sink& snk, ios::openmode which);
void detail::close_all<T>(T& t);
void detail::close_all<T,Sink>(T& t, Sink& snk);
struct detial::close_boost_stream{}; struct detail::close_filtering_stream{};
struct detail::close_tag<T> {
    using category = category_of<T>::type; using unwrapped = unwrapped_type<T>::type;
    using type = select< not_<is_convertible<category,closable_tag>>, any_tag,
                        or_<is_boost_stream<unwrapped>,is_boost_stream_buffer<unwrapped>>, close_boost_stream,
                        or_<is_filtering_stream<unwrapped>,is_filtering_streambuf<unwrapped>>, close_filtering_stream,
                        or_<is_convertible<category,two_sequence>,is_convertible<category,dual_use>>, two_sequence,
                        else_, closable_tag>::type;
};
struct detail::close_impl<T> : if_<is_custom<T>, operations<T>, close_impl<close_tag<T>::type>>::type{};
struct detail::close_impl<any_tag> { static void close<T,[Sink]>(T& t, <Sink& snk>, ios::openmode which); }; // flush, `out` only
struct detail::close_impl<close_boost_stream> { static void close<T,[Sink]>(T& t, <Sink& snk>, ios::openmode which); }; // t.close, `out` only
struct detail::close_impl<close_filtering_stream> { static void close<T,>(T& t, ios::openmode which); }; // t.pop
struct detail::close_impl<closable_tag> { static void close<T,[Sink]>(T& t, <Sink& snk>, ios::openmode which); }; // t.close, `in` only
struct detail::close_impl<two_sequence> { static void close<T,[Sink]>(T& t, <Sink& snk>, ios::openmode which); }; // t.close

bool flush<T>(T& t);
bool flush<T,Sink>(T& t, Sink& snk);
struct detail::flush_device_impl<T> : if_<is_custom<T>, operations<T>, flush_device_impl<dispatch<T,ostream_tag,streambuf_tag,flushable_tag,any_tag>::type>>::type{};
struct detail::flush_device_impl<ostream_tag> { static bool flush<T>(T& t); }; // t.rdbuf()->pubsync()==0;
struct detail::flush_device_impl<streambuf_tag> { static bool flush<T>(T& t); }; // t.pubsync()==0;
struct detail::flush_device_impl<flushable_tag> { static bool flush<T>(T& t); }; // t.flush
struct detail::flush_device_impl<any_tag> { static bool flush<T>(T&); };
struct detail::flush_filter_impl<T> : if_<is_custom<T>, operations<T>, flush_filter_impl<dispatch<T,flushable_tag,any_tag>::type>>::type{};
struct detail::flush_device_impl<flushable_tag> { static bool flush<T,Sink>(T& t, Sink& snk); }; // t.flush
struct detail::flush_device_impl<any_tag> { static bool flush<T,Sink>(T&,Sink&); }; // false

void imbue<T,Locale>(T& t, const Locale& loc);
struct detail::imbue_impl<T> : if_<is_custom<T>, operations<T>, imbue_impl<dispatch<T,streambuf_tag,localizable_tag,any_tag>::type>>::type{};
struct detail::imbue_impl<streambuf_tag> { static void imbue<T,L>(T& t, const L& loc); }; // t.pubimbue
struct detail::imbue_impl<localizable_tag> { static void imbue<T,L>(T& t, const L& loc); }; // t.imbue
struct detail::imbue_impl<any_tag> { static void imbue<T,L>(T&, const L&); };

std::streamsize optimal_buffer_size<T>(const T& t);
struct optimal_buffer_size_impl<T> : if_<is_custom<T>, operations<T>, optimal_buffer_size_impl<dispatch<T,optimally_buffered_tag,device_tag,filter_tag>::type>>::type{};
struct optimal_buffer_size_impl<optimally_buffered_tag> { static streamsize optimal_buffer_size<T>(const T& t); }; // t.optimal_buffer_size;
struct optimal_buffer_size_impl<device_tag> { static streamsize optimal_buffer_size<T>(const T&); }; // default_device_buffer_size
struct optimal_buffer_size_impl<filter_tag> { static streamsize optimal_buffer_size<T>(const T&); }; // default_filter_buffer_size

std::pair<char_type_of<T>::type*, char_type_of<T>::type*> input_sequence<T>(T& t);
struct input_sequence_impl<T> : if_<is_custom<T>, operations<T>, input_sequence_impl<direct_tag>>::type{};
struct input_sequence_impl<direct_tag> { static std::pair<char_type_of<T>::type*,char_type_of<T>::type*> input_sequence<T>(T& t); }; // t.input_sequence

std::pair<char_type_of<T>::type*, char_type_of<T>::type*> output_sequence<T>(T& t);
struct output_sequence_impl<T> : if_<is_custom<T>, operations<T>, output_sequence_impl<direct_tag>>::type{};
struct output_sequence_impl<direct_tag> { static std::pair<char_type_of<T>::type*,char_type_of<T>::type*> output_sequence<T>(T& t); }; // t.output_sequence

int_type_of<T>::type get<T>(T& t);
std::streamsize read<T>(T& t, char_type_of<T>::type* s, std::streamsize n);
std::streamsize read<T,Source>(T& t, Source& src, char_type_of<T>::type* s, std::streamsize n);
bool putback<T>(T& t, char_type_of<T>::type c);
bool detail::true_eof<T>(T& t); // is_linked<T> ? t.true_eof() : true
struct detail::read_device_impl<T> : if_<is_custom<T>, operations<T>, read_device_impl<dispatch<T,istream_tag,streambuf_tag,input>::type>>::type{};
struct detail::read_device_impl<istream_tag> {
    static int_type_of<T>::type get<T>(T& t); // t.get
    static streamsize read<T>(T& t, char_type_of<T>::type* s, streamsize n); // t.rdbuf()->sgetn
    static bool putback<T>(T& t, char_type_of<T>::type c); // t.rdbuf()->sputbackc(c) != eof
};
struct detail::read_device_impl<streambuf_tag> {
    static int_type_of<T>::type get<T>(T& t); // t.sbumpc, would_block if eof
    static streamsize read<T>(T& t, char_type_of<T>::type* s, streamsize n); // t.sgetn
    static bool putback<T>(T& t, char_type_of<T>::type c); // t.sputbackc
};
struct detail::read_device_impl<input> {
    static int_type_of<T>::type get<T>(T& t); // t.read
    static streamsize read<T>(T& t, char_type_of<T>::type* s, streamsize n); // t.read
    static bool putback<T>(T& t, char_type_of<T>::type c); // t.putback
};
struct detail::read_filter_impl<T> : if_<is_custom<T>, operations<T>, read_filter_impl<dispatch<T,multichar_tag,any_tag>::type>>::type{};
struct detail::read_filter_impl<multichar_tag> { static streamsize read<T,Source>(T& t, Source& src, char_type_of<T>::type* s, streamsize n); }; // t.read
struct detail::read_filter_impl<any_tag> { static streamsize read<T,Source>(T& t, Source& src, char_type_of<T>::type* s, streamsize n); }; // loop on t.get

bool put<T>(T& t, char_type_of<T>::type c);
std::streamsize write<T>(T& t, const char_type_of<T>::type* s, std::streamsize n);
std::streamsize write<T,Sink>(T& t, Sink& snk, const char_type_of<T>::type* s, std::streamsize n);
struct detail::write_device_impl<T> : if_<is_custom<T>, operations<T>, write_device_impl<dispatch<T,ostream_tag,streambuf_tag,output>::type>>::type{};
struct detail::write_device_impl<ostream_tag> {
    static bool put<T>(T& t, char_type_of<T>::type c); // t.rdbuf()->sputc
    static streamsize write<T>(T& t, const char_type_of<T>::type* s, streamsize n); // t.rdbuf()->sputn
};
struct detail::write_device_impl<streambuf_tag> {
    static bool put<T>(T& t, char_type_of<T>::type c); // t.sputc
    static streamsize write<T>(T& t, const char_type_of<T>::type* s, streamsize n); // t.sputn
};
struct detail::write_device_impl<output> {
    static bool put<T>(T& t, char_type_of<T>::type c); // t.write
    static streamsize write<T>(T& t, const char_type_of<T>::type* s, streamsize n); // t.write
};
struct detail::write_filter_impl<T> : if_<is_custom<T>, operations<T>, write_filter_impl<dispatch<T,multichar_tag,any_tag>::type>>::type{};
struct detail::write_filter_impl<multichar_tag> { static streamsize write<T,Sink>(T& t, Sink& snk, const char_type_of<T>::type* s, streamsize n); }; // t.write
struct detail::write_filter_impl<any_tag> { static streamsize write<T,Sink>(T& t, Sink& snk, const char_type_of<T>::type* s, streamsize n); }; // loop on t.put

std::streampos seek<T>(T& t, stream_offset off, ios::seekdir way, ios::openmode which=ios::in|ios::out);
std::streampos seek<T,Device>(T& t, Device& dev, stream_offset off, ios::seekdir way, ios::openmode which=ios::in|ios::out);
struct detail::seek_device_impl<T> : if_<is_custom<T>, operations<T>, seek_device_impl<dispatch<T,iostream_tag,istream_tag,ostream_tag,streambuf_tag,two_head,any_tag>::type>>::type{};
struct detail::seek_impl_basic_ios { static streampos seek<T>(T& t, stream_offset off, ios::seekdir way, ios::openmode which); }; // t.rdbuf()->pubseekpos/pubseekoff
struct detail::seek_device_impl<iostream_tag> : seek_impl_basic_ios{};
struct detail::seek_device_impl<istream_tag> : seek_impl_basic_ios{};
struct detail::seek_device_impl<ostream_tag> : seek_impl_basic_ios{};
struct detail::seek_device_impl<streambuf_tag> { static streampos seek<T>(T& t, stream_offset off, ios::seekdir way, ios::openmode which); }; // t.pubseekpos/pubseekoff
struct detail::seek_device_impl<two_head> { static streampos seek<T>(T& t, stream_offset off, ios::seekdir way, ios::openmode which); }; // t.seek(off,way,which)
struct detail::seek_device_impl<any_tag> { static streampos seek<T>(T& t, stream_offset off, ios::seekdir way, ios::openmode); }; // t.seek(off,way)
struct detail::seek_filter_impl<T> : if_<is_custom<T>, operations<T>, seek_filter_impl<dispatch<T,two_head,any_tag>::type>>::type{};
struct detail::seek_filter_impl<two_head> { static streampos seek<T,Device>(T& t, Device& d, stream_offset off, ios::seekdir way, ios::openmode which); }; // t.seek(d, off,way,which)
struct detail::seek_filter_impl<any_tag> { static streampos seek<T,Device>(T& t, Device& d, stream_offset off, ios::seekdir way, ios::openmode); }; // t.seek(d, off,way)

// checked operations
int_type_of<T>::type get_if<T>(T& t);
std::streamsize read_if<T>(T& t, char_type_of<T>::type* s, std::streamsize n);
bool put_if<T>(T& t, char_type_of<T>::type c);
std::streamsize write_if<T>(T& t, const char_type_of<T>::type* s, std::streamsize n);
std::streampos seek_if<T>(T& t, stream_offset off, ios::seekdir way, ios::openmode which=ios::in|ios::out);
struct detail::read_write_if_impl<input> {
    static int_type_of<T>::type get<T>(T& t); // get(t)
    static streamsize read<T>(T& t, char_type_of<T>::type* s, streamsize n); // read(t,s,n)
    static bool put<T>(T&, char_type_of<T>::type); // throw_exception(cant_write())
    static streamsize write<T>(T&, const char_type_of<T>::type*, streamsize n); // throw_exception(cant_write())
};
struct detail::read_write_if_impl<output> {
    static int_type_of<T>::type get<T>(T&); // throw_exception(cant_read())
    static streamsize read<T>(T&, char_type_of<T>::type*, streamsize); // throw_exception(cant_read())
    static bool put<T>(T& t, char_type_of<T>::type c); // put(t,c)
    static streamsize write<T>(T& t, const char_type_of<T>::type* s, streamsize n); // write(t,s,n)
};
struct detail::seek_if_impl<random_access> { static streampos seek<T>(T& t, stream_offset off, ios::seekdir way, ios::openmode which); }; // seek(t,off,way,which)
struct detail::seek_if_impl<any_tag> { static streampos seek<T>(T&, stream_offset, ios::seekdir, ios::openmode); }; // throw_exception(cant_seek())
```

------
### Devices

```c++
// array
struct detail::array_adapter<Mode,Ch> {
    using char_type = Ch; using pair_type = std::pair<char_type*,char_type*>;
    struct category : Mode, device_tag, direct_tag{};
    ctor(<const> char_type* begin, <const> char_type* end);
    ctor(<const> char_type* begin, size_t length);
    ctor<n>(char_type (&ar)[n]);
    pair_type input_sequence(), output_sequence();
private: char_type *begin_, *end_;
};
struct basic_array_source<Ch> : array_adapter<input_seekable,Ch> {};
struct basic_array_sink<Ch> : array_adapter<output_seekable,Ch> {};
struct basic_array<Ch> : array_adapter<seekable,Ch> {};
using array_source = basic_array_source<char>; using warray_source = basic_array_source<wchar_t>;
using array_sink = basic_array_sink<char>; using warray_sink = basic_array_sink<wchar_t>;
using array = basic_array<char>; using warray = basic_array<wchar_t>;

// back_inserter
struct back_insert_device<Container> {
    using char_type = Container::value_type; using category = sink_tag;
    ctor(Container& cnt);
    std::streamsize write(const char_type* s, std::streamsize n); // container->insert at end
protected: Container* container;
};
back_insert_device<Container> back_inserter<Container>(Container& cnt);

// file
struct basic_file<Ch> {
    using char_type = Ch;
    struct category : seekable_device_tag, closable_tag, localizable_tag, flushable_tag{};
    ctor(const std::string& path, ios::openmode mode=ios::in|ios::out, ios::openmode base_mode=ios::in|ios::out);
    streamsize read(char_type* s, streamsize n);
    bool putback(char_type c);
    streamsize write(stream_offset off, ios::seekdir way, ios::openmode which=ios::in|ios::out);
    void open(const std::stream& path, ios::openmode mode=ios::in|ios::out, ios::openmode base_mode=ios::in|ios::out);
    bool is_open() const;
    void close();
    bool flush();
    void imbue(const std::locale& loc);
private: struct impl {
        ctor(const std::string& path, ios::openmode mode); ~dtor(); // open && close
        std::basic_filebuf<Ch> file_;
    };
    shared_ptr<impl> pimpl_;
};
using file = basic_file<char>; using wfile = basic_file<wchar_t>;

struct basic_file_source<Ch> : private basic_file<Ch> {
    using char_type = Ch;
    struct category : input_seekable, device_tag, closable_tag{};
    using base::read; // putback, seek, is_open, close
    ctor(const std::string& path, ios::openmode mode=ios::in);
    void open(const std::stream& path, ios::openmode mode=ios::in);
};
using file_source = basic_file_source<char>; using wfile_source = basic_file_source<wchar_t>;

struct basic_file_sink<Ch> : private basic_file<Ch> {
    using char_type = Ch;
    struct category : output_seekable, device_tag, closable_tag, flushable_tag{};
    using base::write; // seek, is_open, close, flush
    ctor(const std::string& path, ios::openmode mode=ios::out);
    void open(const std::stream& path, ios::openmode mode=ios::out);
};
using file_sink = basic_file_sink<char>; using wfile_sink = basic_file_sink<wchar_t>;

// null
struct basic_null_device<Ch,Mode> {
    using char_type = Ch;
    struct category : Mode, device_tag, closable_tag{};
    streamsize read(Ch*, streamsize) { return -1; }
    streamsize write(const Ch*, streamsize n) { return n; }
    streampos seek(stream_offset, ios::seekdir, ios::openmode=ios::in|ios::out) { return -1; }
    void close(){} void close(ios::openmode){}
};
struct basic_null_source<Ch> : private basic_null_device<Ch,input>
{ using char_type = Ch; using category = source_tag; using base::read; using base::close; };
using null_source = basic_null_source<char>; using wnull_source = basic_null_device<wchar_t>;
struct basic_null_sink<Ch> : private basic_null_device<Ch,output>
{ using char_type = Ch; using category = sink_tag; using base::write; using base::close; };
using null_sink = basic_null_sink<char>; using wnull_sink = basic_null_sink<wchar_t>;

// file descriptor
enum file_descriptor_flags { never_close_handle = 0, close_handle = 3 };
struct file_descriptor {
    using handle_type = file_handle; using char_type = char;
    struct category : seekable_device_tag, closable_tag{};
    ctor(); ctor(const self& other);
    ctor(handle_type fd, file_descriptor_flags);
    explicit ctor(const {std::string&|char*} path, ios::openmode mode=ios::in|ios::out);
    explicit ctor<Path>(const Path& path, ios::openmode mode=ios::in|ios::out);
    void open(handle_type fd, file_descriptor_flags);
    void open(const {std::string&|char*} path, ios::openmode mode=ios::in|ios::out);
    void open<Path>(const Path& path, ios::openmode mode=ios::in|ios::out);
    bool is_open() const;
    void close();
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    streampos seek(stream_offset off, ios::seekdir way);
    handle_type handle() const;
private: void init(); void open(const path& path, ios::openmode, ios::openmode={0});
    shared_ptr<file_descriptor_impl> pimpl_;
};
struct file_descriptor_source : private file_descriptor {
    using handle_type = file_handle; using char_type = char;
    struct category : input_seekable, device_tag, closable_tag{};
    using base::ctor; // is_open, close, read, seek, handle, open, openmode=ios::in
};
struct file_descriptor_sink : private file_descriptor {
    using handle_type = file_handle; using char_type = char;
    struct category : output_seekable, device_tag, closable_tag{};
    using base::ctor; // is_open, close, write, seek, handle, open, openmode=ios::out
};

// mapped file
struct mapped_file_base { enum mapmode { readonly=1, readwrite=2, priv=4 }; };
mapped_file_base::mapmode operator {|,&,^} (mapped_file_base::mapmode a, mapped_file_base::mapmode b)
mapped_file_base::mapmode operator ~ (mapped_file_base::mapmode a)
mapped_file_base::mapmode operator {|=,&=,^=} (mapped_file_base::mapmode& a, mapped_file_base::mapmode b)

struct detail::mapped_file_params_base {
    void normalize();
    mapped_file_base::mapmode flags{0}; ios::openmode mode{};
    stream_offset offset{0}; size_t length{-1}; stream_offset new_file_size{0}; const char* hint{0};
};
struct basic_mapped_file_params<Path> : mapped_file_params_base {
    ctor(); explicit ctor(const Path& p); explicit ctor<Path2>(const Path2& p);
    ctor(const self& other); ctor<Path2>(const self<Path2>& other);
    using path_type = Path;
    Path path;
};
using mapped_file_params = basic_mapped_file_params<std::string>;

class mapped_file_source : public mapped_file_base {
    using impl_type = mapped_file_impl; using param_type = basic_mapped_file_params<path>;
public: using char_type = char; using size_type = size_t; using iterator = const char*;
    struct category : source_tag, direct_tag, closable_tag{};
    constexpr static size_type max_length = size_type{-1};
    ctor(); ctor(const self& other);
    explicit ctor<Path>(const basic_mapped_file_params<Path>& p);
    explicit ctor<Path>(const Path& path, size_type length=max_length, intmax_t offset=0);
    void open<Path>(const basic_mapped_file_params<Path>& p);
    void open<Path>(const Path& path, size_type length=max_length, intmax_t offset=0);
    bool is_open() const;
    void close();
    explicit operator bool() const; bool operator!() const;
    mapmode flags() const;
    size_type size() const;
    const char* data() const;
    iterator begin() const; iterator end() const;
    static int alignment();
private: void init(); void open_impl(const param_type& p);
    shared_ptr<impl_type> pimpl_;
};

class mapped_file : public mapped_file_base {
    using delegate_type = mapped_file_source; using param_type = basic_mapped_file_params<path>;
public: using char_type = char; using size_type = size_t; using <const>_iterator = <const> char*;
    struct category : seekable_device_tag, direct_tag, closable_tag{};
    constexpr static size_type max_length = delegate_type::max_length;
    ctor(); ctor(const self& other);
    explicit ctor<Path>(const basic_mapped_file_params<Path>& p);
    ctor<Path>(const Path& path, mapmode flags, size_type length=max_length, stream_offset offset=0);
    explicit ctor<Path>(const Path& path, ios::openmode mode=ios::in|ios::out, size_type length=max_length, stream_offset offset=0);
    operator <const> mapped_file_source& () <const>;
    void open<Path>(const basic_mapped_file_params<Path>& p);
    void open<Path>(const Path& path, mapmode mode, size_type length=max_length, stream_offset offset=0);
    void open<Path>(const Path& path, ios::openmode mode=ios::in|ios::out, size_type length=max_length, stream_offset offset=0);
    bool is_open() const;
    void close();
    explicit operator bool() const; bool operator!() const;
    mapmode flags() const;
    size_type size() const;
    <const> char* <const>_data() const;
    <const>_iterator <const>_begin() const; <const>_iterator <const>_end() const;
    static int alignment();
    void resize();
private: void init(); void open_impl(const param_type& p);
    delegate_type delegate_;
};

class mapped_file_sink : private mapped_file {
    using base::mapmode, readonly, readwrite, priv, char_type, size_type, iterator, max_length;
    struct category : sink_tag, direct_tag, closable_tag{};
    using base::is_open, close, operator bool, operator!, flags, size, data, begin, end, alignment, resize;
    ctor(); ctor(const self& other);
    explicit ctor<Path>(const basic_mapped_file_params<Path>& p);
    ctor<Path>(const Path& path, size_type length=max_length, stream_offset offset=0, mapmode flags=readwrite);
    void open<Path>(const basic_mapped_file_params<Path>& p);
    void open<Path>(const Path& path, size_type length=max_length, stream_offset offset=0, mapmode flags=readwrite);
};

struct operations<mapped_file_source> : close_impl<closable_tag> { static std::pair<char*,char*> input_sequence(mapped_file_source& src); };
struct operations<mapped_file> : close_impl<closable_tag> { static std::pair<char*,char*> {input|output}_sequence(mapped_file& file); };
struct operations<mapped_file_sink> : close_impl<closable_tag> { static std::pair<char*,char*> output_sequence(mapped_file& file); };
```

------
### Filters

#### Filter Pipeline

```c++
struct detail::pipeline_segment<Component> { // no copy assign
    ctor(const Component& component);
    void for_each<Fn>(Fn fn) const { fn(component_); }
    void push<Chain>(Chain chn) const { chn.push(component_); }
private: const Component& component_;
};
struct pipeline<Pipeline,Component> : Pipeline { // no copy assign
    using pipeline_type = Pipeline; using component_type = Component;
    ctor(const Pipeline& p, const Component& component);
    void for_each<Fn>(Fn fn) const { base::for_each(fn); fn(component_); }
    void push<Chain>(Chain& chn) const { base::oush(chn); chn.push(component_); }
    const Pipeline& tail() const { return *this; }
    const Component& head() const { return component_; }
private: const Component& component_;
};
pipeline<pipeline<P,F>,C> operator| <P,F,C> (const pipeline<P,F>& p, const C& c);
```

#### Helper Filter Base Classes

```c++
// line
struct basic_line_filter<Ch,Alloc=std::allocator<Ch>> {
    using char_type = Ch; using traits_type = char_traits<char_type>;
    using string_type = std::basic_string<Ch,std::char_traits<Ch>,Alloc>;
    struct category : dual_use, filter_tag, multichar_tag, closable_tag {};
protected: ctor(bool suppress_newlines=false);
public: virtual ~dtor(){}
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& src, const char_type* s, streamsize n);
    void close<Sink>(Sink& snk, ios::openmode which);
private: virtual string_type do_filter(const string_type& line) =0;
    streamsize read_line(char_type* s, streamsize n);
    traits_type::int_type next_line<Source>(Source& src);
    bool write_line<Sink>(Sink& snk);
    void close_impl(); void clear();
    enum flag_type { f_read=1, f_write=2, f_suppress=4 };
    string_type cur_line_; string_type::size_type pos_; int flags_;
};
pipeline<pipeline_segment<basic_line_filter<T0,T1>>, C> operator|( const basic_line_filter<T0,T1>& f, const C& c);
using line_filter = basic_line_filter<char>; using wline_filter = basic_line_filter<wchar_t>;

// aggregate
struct aggregate_filter<Ch,Alloc=std::allocator<Ch>> {
    using char_type = Ch;
    struct category : dual_use, filter_tag, multichar_tag, closable_tag {};
    ctor(); virtual ~dtor(){}
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& src, const char_type* s, streamsize n);
    void close<Sink>(Sink& snk, ios::openmode which);
protected: using vector_type = std::vector<Ch,Alloc>; using size_type = vector_type::size_type;
private: virtual void do_filter(const vector_type& src, vector_type& dest) =0;
    virtual void do_close(){}
    void do_read<Source>(Source& src);
    void do_write<Sink>(Sink& snk, const char_type* s, streamsize n);
    void close_impl();
    enum flag_type { f_read=1, f_write=2, f_eof=4 };
    vector_type data_; size_type ptr_; int state_;
};
pipeline<pipeline_segment<aggregate_filter<T0>>, C> operator|( const aggregate_filter<T0>& f, const C& c);

// symmetric_filter
struct symmetric_filter<SymF,Alloc=std::allocator<char_type_of<SymF>::type>> {
    using char_type = char_type_of<SymF>::type; using traits_type = std::char_traits<char_type>;
    using string_type = std::basic_string<char_type, traits_type, Alloc>;
    struct category : dual_use, filter_tag, multichar_tag, closable_tag {};
    explicit ctor<...T>(std::streamsize buffer_size, const T&...t);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& src, const char_type* s, streamsize n);
    void close<Sink>(Sink& snk, ios::openmode which);
private: using buffer_type = buffer<char_type,Alloc>;
    <const> buffer_type& buf() <const>;
    int& state();
    void begin_read(); void begin_write();
    int fill<Source>(Source& src);
    bool flush<Sink>(Sink& snk);
    void close_impl();
    enum flag_type { f_read=1, f_write=2, f_eof=4, f_good, f_would_block };
    struct impl : SymF { ctor<...T>(streamsize buffer_size, const T&...t); buffer_type buf_; int state_; };
    shared_ptr<impl> pimpl_;
};
pipeline<pipeline_segment<symmetric_filter<T0,T1>>, C> operator|( const symmetric_filter<T0,T1>& f, const C& c);

// stdio
class basic_stdio_filter<Ch,Alloc=std::allocator<Ch>> : public aggregate_filter<Ch,Alloc> {
    static std::<w>istream& standard_input({char|wchar_t}*); // cin/wcin
    static std::<w>ostream& standard_output({char|wchar_t}*); // cout/wcout
    struct scoped_redirector{ using traits_type = std::char_traits<Ch>;
        using ios_type = std::basic_ios<Ch,traits_type>; using streambuf_type = std::basic_streambuf<Ch,traits_type>;
        ctor(ios_type& ios, streambuf_type* newbuf) : ios_{ios}, old_{ios.rdbuf(newbuf)}{} // change to newbuf
        ~dtor() { ios_.rdbuf(old_); } // set back
        ios_type& ios_; streambuf_type* old_;
    };
    virtual void do_filter() =0;
    virtual void do_filter(const vector_type& src, vector_type& dest);
};
pipeline<pipeline_segment<basic_stdio_filter<T0,T1>>, C> operator|( const basic_stdio_filter<T0,T1>& f, const C& c);
using stdio_filter = basic_stdio_filter<char>; using wstdio_filter = basic_stdio_filter<wchar_t>;
```

#### Text Filters

```c++
// counter
struct basic_counter<Ch> {
    using char_type = Ch;
    struct category : dual_use, filter_tag, multichar_tag, optimally_buffered_tag{};
    explicit ctor(int first_line=0, int first_char=0);
    int lines() const; int characters() const;
    streamsize optimal_buffer_size() const { return 0; }
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
private: int lines_, chars_;
};
pipeline<pipeline_segment<basic_counter<T0>>, C> operator|( const basic_counter<T0>& f, const C& c);
using counter = basic_counter<char>; using wcounter = basic_counter<wchar_t>;

// regex
struct basic_regex_filter<Ch,Tr=regex_traits<Ch>,Alloc=std::allocator<Ch>> : public aggregate_filter<Ch,Alloc> {
    using string_type = std::basic_string<Ch>; using regex_type = basic_regex<Ch,Tr>;
    using flag_type = regex_constants::match_flag_type; using match_type = match_results<const Ch*>;
    using formatter = function<string_type<const match_type&>>;
    ctor(const regex_type& re, const formatter& replace, flat_type flags=match_default);
    ctor(const regex_type& re, const {string_type&|char_type*} fmt, flat_type flags=match_default, flag_type fmt_flags=format_default);
private: void do_filter(const vector_type& src, vector_type& dest);
    struct simple_formatter {
        string_type operator()(const match_type& match) const;
        string_type fmt_; flag_type fmt_flags_;
    };
    regex_type re_; formatter replace_; flag_type flags_;
};
pipeline<pipeline_segment<basic_regex_filter<T0,T1,T2>>, C> operator|( const basic_regex_filter<T0,T1,T2>& f, const C& c);
using regex_filter = basic_regex_filter<char>; using wregex_filter = basic_regex_filter<wchar_t>;

// grep
const int grep::invert=1, whole_line=2;
struct basic_grep_filter<Ch,Tr=regex_traits<Ch>,Alloc=std::allocator<Ch>> : public basic_line_filter<Ch,Alloc> {
    using traits_type = char_traits<char_type>; using string_type = std::basic_string<Ch>;
    using regex_type = basic_regex<Ch,Tr>; using match_flag_type = regex_constants::match_flag_type;
    ctor(const regex_type& re, match_flat_type matchflags=match_default, int options=0);
    int count() const;
    void close<Sink>(Sink& snk, ios::openmode which);
private: void do_filter(const string_type& line);
    enum flags_ { f_initialized = 65536 };
    regex_type re_; match_flag_type match_flags_; int options_, count_;
};
pipeline<pipeline_segment<basic_grep_filter<T0,T1,T2>>, C> operator|( const basic_grep_filter<T0,T1,T2>& f, const C& c);
using grep_filter = basic_grep_filter<char>; using wgrep_filter = basic_grep_filter<wchar_t>;

// newline
const char newline::CR=0x0D, LF=0x0A;
const int newline::posix=1, mac=2, dos=4, mixed=8, final_newline=16, platform_mask=posix|dos|mac;
struct detail::newline_base {
    bool is_posix() const; bool is_dos() const; bool is_mac() const;
    bool is_mixed_posix() const; bool is_mixed_dos() const; bool is_mixed_mac() const;
    bool is_mixed() const;
    bool has_final_newline() const;
protected: ctor(int flags);
    int flags_;
};
class newline_error : public std::ios::failure, public newline_base { ctor(int flags); };
struct newline_filter {
    using char_type = char;
    struct category : dual_use, filter_tag, closable_tag {};
    explicit ctor(int target);
    int get<Source>(Source& src);
    bool put<Sink>(Sink& dest, char c);
    void close<Sink>(Sink& dest, ios::openmode);
private: int newline();
    bool newline<Sink>(Sink& dest);
    void newline_if_sink<Device>(Device& dest);
    enum flags { f_has_LF=0x8000, f_has_CR=0x10000, f_has_newline=0x20000, f_has_EOF=0x40000, f_read=0x80000, f_write=0x100000 };
    int flags_;
};
pipeline<pipeline_segment<basic_grep_filter>, C> operator|( const basic_grep_filter& f, const C& c);

struct newline_checker : public newline_base {
    using char_type = char;
    struct category : dual_use_filter_tag, closable_tag {};
    explicit ctor(int target=newline::mixed);
    int get<Source>(Source& src);
    bool put<Sink>(Sink& dest, int c);
    void close<Sink>(Sink&, ios::openmode);
private: void fail(){ throw_exception(newline_error(source())); }
    int& source(); int source() const;
    enum flags { f_has_CR=0x8000, f_line_complete=0x10000 };
    int target_; bool open_;
};
pipeline<pipeline_segment<newline_checker>, C> operator|( const newline_checker& f, const C& c);
```

#### Compression Filters

```c++
// bzip2
using bzip2::alloc_func = void* (*) (void*,int,int); using bzip2::free_func = void (*)(void*, void*);
extern const int bzip2::ok, run_ok, flush_ok, finish_ok, stream_end, sequence_error, param_error, mem_error,
    data_error, data_error_magic, io_error, unexpected_eof, outbuf_full, config_error;
extern const int bzip2::finish, run;
const int bzip2::default_block_size=9, default_work_factor=30;
const bool bzip2::default_small=false;

struct bzip2_params {
    ctor(int block_size_=bzip2::default_block_size, int work_factor_=bzip2::default_work_factor);
    ctor(bool small);
    union { int block_size; bool small; }; int work_factor;
};
struct bzip2_error : public std::ios::failure {
    explicit ctor(int error);
    int error() const;
    static void check(int error);
private: int error_;
};
struct detail::bzip2_allocator_traits<Alloc> { using type = std::allocator_traits<Alloc>::rebind_alloc<char>; };
struct detail::bzip2_allocator<Alloc> : bzip2_allocator_traits<Alloc>::type {
    static constexpr bool custom=!is_same_v<std::allocator<char>,base>;
    using allocator_type = base;
    static void* allocate(void* self, int items, int size);
    static void deallocate(void* self, void* address);
};
struct detail::bzip2_base {
    using char_type = char;
protected: ctor(const bzip2_params& params); ~dtor();
    bzip2_params& params(); bool& ready();
    void init<Alloc>(bool compress, bzip2_allocator<Alloc>& alloc);
    void before(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end);
    void after(const char*& src_begin, char*& dest_begin);
    int check_end(const char* src_begin, const char* dest_begin);
    int compress(int action); int decompress();
    int end(bool compress, std::nothrow_t); void end(bool compress);
private: void do_init(bool compress, bzip2::alloc_func, bzip2::free_func, void* derived);
    bzip2_params params_; void* stream_; bool ready_;
};
struct detail::bzip2_compressor_impl<Alloc=std::allocator<char>> : public bzip2_base, bzip2_allocator<Alloc> {
    ctor(const bzip2_params&); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
private: void init();
    bool eof_;
};
struct detail::bzip2_decompressor_impl<Alloc=std::allocator<char>> : public bzip2_base, bzip2_allocator<Alloc> {
    ctor(bool small=bzip2::default_small); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
private: void init();
    bool eof_;
};

struct basic_bzip2_compressor<Alloc=std::allocator<char>> : symmetric_filter<bzip2_compressor_impl<Alloc>,Alloc> {
    ctor(const bzip2_params&=bzip2::default_block_size, streamsize buffer_size=default_device_buffer_size);
};
pipeline<pipeline_segment<basic_bzip2_compressor<T0>>, C> operator|( const basic_bzip2_compressor<T0>& f, const C& c);
using bzip2_compressor = basic_bzip2_compressor<>;

struct basic_bzip2_decompressor<Alloc=std::allocator<char>> : symmetric_filter<bzip2_decompressor_impl<Alloc>,Alloc> {
    ctor(bool small=bzip2::default_small, streamsize buffer_size=default_device_buffer_size);
};
pipeline<pipeline_segment<basic_bzip2_decompressor<T0>>, C> operator|( const basic_bzip2_decompressor<T0>& f, const C& c);
using bzip2_decompressor = basic_bzip2_decompressor<>;

// zlib
using zlib::uint = uint32_t; using zlib::byte = uint8_t; using zlib::ulong = uint32_t;
using zlib::xalloc_func = void* (*)(void*, zlib::uint, zlib::uint); using zlib::xfree_func = void (*)(void*, void*);
extern const int zlib::no_compression, best_speed, best_compression, default_compression;
extern const int zlib::deflated;
extern const int zlib::default_strategy, filtered, huffman_only;
extern const int zlib::okay, stream_end, stream_error, version_error, data_error, mem_error, buf_error;
extern const int zlib::finish, no_flush, sync_flush;
const int zlib::null=0, default_window_bits=15, default_mem_level=8;
const bool zlib::default_crc=false, default_noheader=false;

struct zlib_params {
    int level=default_compression, method=deflated, window_bits=default_window_bits, mem_level=default_mem_level, strategy=default_strategy;
    bool noheader=default_noheader, calculate_crc=default_crc;
};
struct zlib_error : public std::ios::failure {
    explicit ctor(int error);
    int error() const;
    static void check(int error);
private: int error_;
};
struct detail::zlib_allocator_traits<Alloc> { using type = std::allocator_traits<Alloc>::rebind_alloc<char>; };
struct detail::zlib_allocator<Alloc> : private zlib_allocator_traits<Alloc>::type {
    constexpr static bool custom = !is_same_v<std::allocator<char>,base>;
    using allocator_type = base;
    static void* allocate(void* self, zlib::uint items, zlib::uint size);
    static void deallocate(void* self, void* address);
};
struct detail::zlib_base {
    using char_type = char;
protected: ctor() ~dtor();
    void* stream();
    void init<Alloc>(const zlib_params& p, bool compress, zlib_allocator<Alloc>& zalloc);
    void before(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end);
    void after(const char*& src_begin, char*& dest_begin, bool compress);
    int xdeflate(int flush); int xinflate(int flush);
    void reset(bool compress, bool realloc);
public: zlib::ulong crc() const; int total_in() const; int total_out() const;
private: void do_init(const zlib_params& p, bool compress, zlib::xalloc_func, zlib::xfree_func, void* derived);
    void* stream_; bool calculate_crc_; zlib::ulong crc_, crc_imp_; int total_in_, total_out_;
};
struct detail::zlib_compressor_impl<Alloc=std::allocator<char>> : public zlib_base, public zlib_allocator<Alloc> {
    ctor(const zlib_params&=zlib::default_compression); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
};
struct detail::zlib_decompressor_impl<Alloc=std::allocator<char>> : public zlib_base, public zlib_allocator<Alloc> {
    ctor(const zlib_params&); ctor(int window_bits=zlib::default_window_bits); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
    bool eof() const;
private: bool eof_;
};
struct basic_zlib_compressor<Alloc=std::allocator<char>> : symmetric_filter<zlib_compressor_impl<Alloc>,Alloc> {
    ctor(const zlib_params&=zlib::default_compression, streamsize buffer_size=default_device_buffer_size);
    zlib::ulong crc(); int total_in();
};
pipeline<pipeline_segment<basic_zlib_compressor<T0>>, C> operator|( const basic_zlib_compressor<T0>& f, const C& c);
using zlib_compressor = basic_zlib_compressor<>;

struct basic_zlib_decompressor<Alloc=std::allocator<char>> : symmetric_filter<basic_zlib_decompressor<Alloc>,Alloc> {
    ctor(int window_bits=zlib::default_window_bits, streamsize buffer_size=default_device_buffer_size);
    ctor(const zlib_params& p, streamsize buffer_size=default_device_buffer_size);
    zlib::ulong crc(); int total_out(); bool eof();
};
pipeline<pipeline_segment<basic_zlib_decompressor<T0>>, C> operator|( const basic_zlib_decompressor<T0>& f, const C& c);
using zlib_decompressor = basic_zlib_decompressor<>;

// gzip
namespace gzip { using namespace zlib; }
const int gzip::zlib_error=1, bad_crc=2, bad_length=3, bad_header=4, bad_footer=5, bad_method=6;
const int gzip::magic::id1=0x1f, id2=0x8b;
const int gzip::method::deflate=8;
const int gzip::flags::text=1, header_crc=2, extra=4, name=8, comment=16;
const int gzip::extra_flags::best_compression=2, best_speed=4;
const int gzip::os_fat=0, os_amiga=1, os_vms=2, os_unix=3, os_vm_cms=4, os_atari=5, os_hpfs=6, os_macintosh=7,
    os_z_system=8, os_cp_m=9, os_tops_20=10, os_ntfs=11, os_qdos=12, os_acorn=13, os_unknown=255;
struct gzip_params : zlib_params {
    std::string file_name="", comment=""; std::time_t mtime=0;
};
struct gzip_error : public std::ios::failure {
    explicit ctor(int error); explicit ctor(const zlib_error& e)
    int error() const; int zlib_error_code() const;
private: int error_, zlib_error_code_;
};
struct basic_gzip_compressor<Alloc=std::allocator<char>> : basic_zlib_compressor<Alloc> {
    using char_type = char;
    struct category : dual_use, filter_tag, multichar_tag, closable_tag {};
    ctor(const gzip_params&=gzip::default_compression, streamsize buffer_size=default_device_buffer_size);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
    void close<Sink>(Sink& snk, ios::openmode m);
private: static gzip_params normalize_params(gzip_params p);
    void prepare_footer();
    streamsize read_string(char* s, streamsize n, std::string& str);
    static void write_long<Sink>(long n, Sink& next);
    void close_impl();
    enum state_type { f_header_done=1, f_body_done=2, f_footer_done=4 };
    std::string header_, footer_; size_t offset_; int flags_;
};
pipeline<pipeline_segment<basic_gzip_compressor<T0>>, C> operator|( const basic_gzip_compressor<T0>& f, const C& c);
using gzip_compressor = basic_gzip_compressor<>;

struct detail::gzip_header {
    ctor() {reset();}
    void process(char c);
    bool done() const;
    void reset();
    std::string file_name() const; std::string comment() const;
    bool text() const; int os() const; time_t mtime() const;
private: enum state_type { s_id1=1, s_id2, s_cm, s_flg, s_mtime, s_xfl, s_os, s_xlen, s_extra, s_name, s_comment, s_hcrc, s_done };
    std::string file_name_, comment_; int os_; time_t mtime_;
    int flags_, state_, offset_, xlen_;
};
struct detail::gzip_footer {
    ctor() {reset();}
    void process(char c);
    bool done() const;
    void reset();
    zlib::ulong crc() const; zlib::ulong uncompressed_size() const;
private: enum state_type { s_crc=1, s_isize, s_done };
    zlib::ulong crc_, isize_; int state_, offset_;
};
struct basic_gzip_decompressor<Alloc=std::allocator<char>> : basic_zlib_decompressor<Alloc> {
    using char_type = char;
    struct category : dual_use, filter_tag, multichar_tag, closable_tag {};
    ctor(int window_bits=gzip::default_window_bits, streamsize buffer_size=default_device_buffer_size);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
    void close<Sink>(Sink& snk, ios::openmode m);
    std::string file_name() const; std::string comment() const;
    bool text() const; int os() const; time_t mtime() const;
private: static gzip_params make_params(int window_bits);
    struct peekable_source<Source> {
        using char_type = char;
        struct category : source_tag, peekable_tag{};
        explicit ctor(Source& src, const string_type& putback="");
        streamsize read(char* s, streamsize n);
        bool putback(char c); void putback(const string_type& s);
        bool has_unconsumed_input() const; string_type unconsumed_input() const;
        Source& src_; string_type putback_; streamsize offset_;
    };
    enum state_type { s_start=1, s_header, s_body, s_footer, s_done };
    gzip_header header_; gzip_footer footer_; string_type putback_; int state_;
};
pipeline<pipeline_segment<basic_gzip_decompressor<T0>>, C> operator|( const basic_gzip_decompressor<T0>& f, const C& c);
using gzip_decompressor = basic_gzip_decompressor<>;

// lzma
using lzma::alloc_func = void* (*)(void*, size_t, size_t); using lzma::free_func = void (*)(void*, void*);
extern const int lzma::no_compression, best_speed, best_compression, default_compression;
extern const int lzma::okay, stream_end, unsupported_check, mem_error, options_error, data_error, buf_error, prog_error;
extern const int lzma::finish, full_flush, sync_flush, run;
const int lzma::null=0;

struct lzma_params { uint32_t level=default_compression, threads=1; };
struct lzma_error : public std::ios::failure {
    explicit ctor(int error);
    int error() const;
    static void check(int error);
private: int error_;
};
struct detail::lzma_allocator_traits<Alloc> { using type = std::allocator_traits<Alloc>::rebind_alloc<char>; };
struct detail::lzma_allocator<Alloc> : private lzma_allocator_traits<Alloc>::type {
    constexpr static bool custom = !is_same_v<std::allocator<char>,base>;
    using allocator_type = base;
    static void* allocate(void* self, size_t items, size_t size);
    static void deallocate(void* self, void* address);
};
struct detail::lzma_base {
    using char_type = char;
protected: ctor() ~dtor();
    void* stream();
    void init<Alloc>(const lzma_params& p, bool compress, lzma_allocator<Alloc>& zalloc);
    void before(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end);
    void after(const char*& src_begin, char*& dest_begin, bool compress);
    int deflate(int action); int inflate(int action);
    void reset(bool compress, bool realloc);
public: zlib::ulong crc() const; int total_in() const; int total_out() const;
private: void do_init(const lzma_params& p, bool compress, lzma::alloc_func, lzma::free_func, void* derived);
    void init_stream(bool compress);
    void* stream_; uint32_t level_, threads_;
};
struct detail::lzma_compressor_impl<Alloc=std::allocator<char>> : public lzma_base, public lzma_allocator<Alloc> {
    ctor(const lzma_params&={}); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
};
struct detail::lzma_decompressor_impl<Alloc=std::allocator<char>> : public lzma_base, public lzma_allocator<Alloc> {
    ctor(const lzma_params&); ctor(); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
};
struct basic_lzma_compressor<Alloc=std::allocator<char>> : symmetric_filter<lzma_compressor_impl<Alloc>,Alloc> {
    ctor(const lzma_params&={}, streamsize buffer_size=default_device_buffer_size);
    zlib::ulong crc(); int total_in();
};
pipeline<pipeline_segment<basic_lzma_compressor<T0>>, C> operator|( const basic_lzma_compressor<T0>& f, const C& c);
using lzma_compressor = basic_lzma_compressor<>;

struct basic_lzma_decompressor<Alloc=std::allocator<char>> : symmetric_filter<basic_lzma_decompressor<Alloc>,Alloc> {
    ctor(buffer_size=default_device_buffer_size);
    ctor(const lzma_params& p, streamsize buffer_size=default_device_buffer_size);
};
pipeline<pipeline_segment<basic_lzma_decompressor<T0>>, C> operator|( const basic_lzma_decompressor<T0>& f, const C& c);
using lzma_decompressor = basic_lzma_decompressor<>;

// zstd
using zstd::alloc_func = void* (*)(void*, size_t, size_t); using zstd::free_func = void (*)(void*, void*);
extern const int zstd::best_speed, best_compression, default_compression;
extern const int zstd::okay, stream_end;
extern const int zstd::finish, flush, run;
const int zstd::null=0;

struct zstd_params { uint32_t level=default_compression; };
struct zstd_error : public std::ios::failure {
    explicit ctor(int error);
    int error() const;
    static void check(int error);
private: int error_;
};
struct detail::zstd_allocator_traits<Alloc> { using type = std::allocator_traits<Alloc>::rebind_alloc<char>; };
struct detail::zstd_allocator<Alloc> : private zstd_allocator_traits<Alloc>::type {
    constexpr static bool custom = !is_same_v<std::allocator<char>,base>;
    using allocator_type = base;
    static void* allocate(void* self, size_t items, size_t size);
    static void deallocate(void* self, void* address);
};
struct detail::zstd_base {
    using char_type = char;
protected: ctor() ~dtor();
    void init<Alloc>(const zstd_params& p, bool compress, zstd_allocator<Alloc>& zalloc);
    void before(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end);
    void after(const char*& src_begin, char*& dest_begin, bool compress);
    int deflate(int action); int inflate(int action);
    void reset(bool compress, bool realloc);
private: void do_init(const zstd_params& p, bool compress, zstd::alloc_func, zstd::free_func, void* derived);
    void *cstream_, *dstream_, *in_, *out_; int eof_; uint32_t level;
};
struct detail::zstd_compressor_impl<Alloc=std::allocator<char>> : public zstd_base, public zstd_allocator<Alloc> {
    ctor(const zstd_params&=zstd::default_compression); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
};
struct detail::zstd_decompressor_impl<Alloc=std::allocator<char>> : public zstd_base, public zstd_allocator<Alloc> {
    ctor(const zstd_params&); ctor(); ~dtor();
    bool filter(const char*& src_begin, const char* src_end, char*& dest_begin, char* dest_end, bool flush);
    void close();
};
struct basic_zstd_compressor<Alloc=std::allocator<char>> : symmetric_filter<zstd_compressor_impl<Alloc>,Alloc> {
    ctor(const zstd_params&=zstd::default_compression, streamsize buffer_size=default_device_buffer_size);
    zlib::ulong crc(); int total_in();
};
pipeline<pipeline_segment<basic_zstd_compressor<T0>>, C> operator|( const basic_zstd_compressor<T0>& f, const C& c);
using zstd_compressor = basic_zstd_compressor<>;

struct basic_zstd_decompressor<Alloc=std::allocator<char>> : symmetric_filter<basic_zstd_decompressor<Alloc>,Alloc> {
    ctor(buffer_size=default_device_buffer_size);
    ctor(const zstd_params& p, streamsize buffer_size=default_device_buffer_size);
};
pipeline<pipeline_segment<basic_zstd_decompressor<T0>>, C> operator|( const basic_zstd_decompressor<T0>& f, const C& c);
using zstd_decompressor = basic_zstd_decompressor<>;
```

------
### Algorithms

```c++
streamsize copy<Source,Sink>(const Source& src, const Sink& snk, streamsize buffer_size=default_device_buffer_size);
struct detail::copy_operation<Source,Sink> {
    streamsize operator()();
private: Source& src_; Sink& snk_; streamsize buffer_size_;
};
streamsize detail::copy_impl<Source,Sink>(Source src, Sink snk, streamsize buffer_size);

void skip<Device>(Device& dev, stream_offset off);
void skip<Filter,Device>(Filter& flt, Device& dev, stream_offset off, ios::openmode which=ios::in|ios::out);
```

------
### Views

```c++
// combine
struct detail::combined_device<Source,Sink> {
    using in_category=category_of<Source>::type; using out_category=category_of<Sink>::type;
    using char_type=char_type_of<Source>::type; using sink_char_type=char_type_of<Sink>::type;
    struct category : bidirectional, device_tag, closable_tag, localizable_tag {};
    ctor(const Source& src, const Sink& snk);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    void close(ios::openmode);
    void imbue(const std::locale& loc);
private: Source src_; Sink sink_;
};
struct detail::combined_filter<InputFilter,OutputFilter> {
    using in_category=category_of<InputFilter>::type; using out_category=category_of<OutputFilter>::type;
    using char_type=char_type_of<InputFilter>::type; using output_char_type=char_type_of<OutputFilter>::type;
    struct category : multichar_bidirectional_filter_tag, closable_tag, localizable_tag {};
    ctor(const InputFilter& in, const OutputFilter& out);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
    void close<Sink>(Sink& snk, ios::openmode);
    void imbue(const std::locale& loc);
private: InputFilter in_; OutputFilter out_;
};
struct detail::combination_traits<In,Out>
    : if_<is_device<In>, combined_device<wrapped_type<In>::type,wrapped_type<Out>::type>, combined_filter<wrapped_type<In>::type,wrapped_type<Out>::type>>{};
struct combination<In,Out> : combination_traits<In,Out>::type {
    using in_type = wrapped_type<In>::type; using out_type = wrapped_type<Out>::type;
    ctor(const in_type& in, const out_type& out);
};

struct detail::combine_traits<In,Out> { using type=combination<unwrapped_type<In>::type,unwrapped_type<Out>::type>; };
combine_traits<In,Out>::type combine<In,Out>(const In& in, const Out& out);

// compose
struct detail::composite_mode<T1,T2,Mode1=mode_of<T1>::type,Mode2=mode_of<T2>::type>
    : select< is_convertible<Mode2,Mode1>, Mode1,
              is_convertible<Mode1,Mode2>, Mode2,
              is_convertible<Mode2,input>, input, else_, output> {};
class detail::composite_device<Filter,Device,Mode=composite_mode<Filter,Device>::type> {
    using param_type=param_type<Device>::type;
    using filter_mode=mode_of<Filter>::type; using device_mode=mode_of<Device>::type;
    using value_type = select<is_direct<Device>, direct_adapter<Device>,
                              is_std_io<Device>, Device&, else_, Device>::type;
public: using char_type = char_type_of<Filter>::type;
    struct category : Mode, device_tag, closable_tag, flushable_tag, localizable_tag, optimally_buffered_tag {};
    ctor(const Filter& flt, param_type dev);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    streampos seek(stream_offset off, ios::seekdir way, ios::openmode which=ios::in|ios::out);
    void close(); void close(ios::openmode which);
    bool flush();
    streamsize optimal_buffer_size() const;
    void imbue<Locale>(const Locale& loc);
    Filter& first(); Device& second();
private: Filter filter_; value_type device_;
};
class detail::composite_filter<Filter1,Filter2,Mode=composite_mode<Filter1,Filter2>::type> {
    using filter_ref = reference_wrapper<Filter2>;
    using first_mode=mode_of<Filter1>::type; using second_mode=mode_of<Filter2>::type;
public: using char_type = char_type_of<Filter1>::type;
    struct category : Mode, filter_tag, multichar_tag, closable_tag, flushable_tag, localizable_tag, optimally_buffered_tag {};
    ctor(const Filter1& flt1, const Filter2& flt2);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
    streampos seek<Device>(Device& dev, stream_offset off, ios::seekdir way, ios::openmode which=ios::in|ios::out);
    void close<Device>(Device& dev); void close<Device>(Device& dev, ios::openmode which);
    bool flush<Device>(Device& dev);
    streamsize optimal_buffer_size() const;
    void imbue<Locale>(const Locale& loc);
    Filter1& first(); Filter2& second();
private: Filter1 filter1_; Filter2 filter2_;
};
struct detail::composite_traits<Filter,FltOrDev> : if_<is_device<FltOrDev>, composite_device<Filter,FltOrDev>, composite_filter<Filter,FltOrDev>>{};
struct composite<Filter,FltOrDev> : composite_traits<Filter,FltOrDev>::type {
    using param_type=param_type<FltOrDev>::type;
    ctor(const Filter& flt, param_type dev);
};
composite<Filter,FltOrDev> compose<Filter,FltOrDev>(const Filter& filter, const FltOrDev& fod);
composite<Filter,std::basic_streambuf<Ch,Tr>> compose<Filter,Ch,Tr>(const Filter& filter, std::basic_streambuf<Ch,Tr>& sb);
composite<Filter,std::basic_istream<Ch,Tr>> compose<Filter,Ch,Tr>(const Filter& filter, std::basic_istream<Ch,Tr>& is);
composite<Filter,std::basic_ostream<Ch,Tr>> compose<Filter,Ch,Tr>(const Filter& filter, std::basic_ostream<Ch,Tr>& os);
composite<Filter,std::basic_iostream<Ch,Tr>> compose<Filter,Ch,Tr>(const Filter& filter, std::basic_iostream<Ch,Tr>& io);

// invert
class inverse<Filter> {
    using base_category=category_of<Filter>::type;
    using filter_ref = reference_wrapper<Filter>;
public: using char_type = char_type_of<Filter>::type; using int_type = int_type_of<Filter>::type;
    using traits_type = char_traits<char_type>;
    using mode = if_<is_convertible<base_category,input>, output,input>::type;
    struct category: mode, filter_tag, multichar_tag, closable_tag {};
    explicit ctor(const Filter& filter, streamsize buffer_size=default_filter_buffer_size);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
    void close<Device>(Device& dev);
private: filter_ref filter(); buffer<char_type>& buf(); int& flags();
    enum flags_ { f_read=1, f_write=2 };
    struct impl {
        ctor(const Filter& filter, streamsize n) : filter_{filter}, buf_{n}, flags_{0}{buf_.set(0,0);}
        Filter filter_; buffer<char_type> buf_; int flags_;
    };
    shared_ptr<impl> pimpl_;
};
inverse<Filter> invert<Filter>(const Filter& f);

// restrict/slice
restriction<T> slice<T>(const T& t, stream_offset off, stream_offset len=-1) requires !is_std_io<T>;
restriction<std::basic_streambuf<Ch,Tr>> slice<Ch,Tr>(std::basic_streambuf<Ch,Tr>& sb, stream_offset off, stream_offset len=-1);
restriction<std::basic_istream<Ch,Tr>> slice<Ch,Tr>(std::basic_istream<Ch,Tr>& is, stream_offset off, stream_offset len=-1);
restriction<std::basic_ostream<Ch,Tr>> slice<Ch,Tr>(std::basic_ostream<Ch,Tr>& os, stream_offset off, stream_offset len=-1);
restriction<std::basic_iostream<Ch,Tr>> slice<Ch,Tr>(std::basic_iostream<Ch,Tr>& io, stream_offset off, stream_offset len=-1);

// tee
struct tee_filter<Device> : filter_adapter<Device> {
    using param_type=param_type<Device>::type; using char_type = char_type_of<Device>::type;
    struct category : dual_use_filter_tag, multichar_tag, closable_tag, flushable_tag, localizable_tag, optimally_buffered_tag {};
    explicit ctor(param_type dev);
    streamsize read<Source>(Source& src, char_type* s, streamsize n);
    streamsize write<Sink>(Sink& snk, const char_type* s, streamsize n);
    void close<Next>(Next& dev, ios::openmode);
    bool flush<Sink>(Sink& snk);
};
pipeline<pipeline_segment<tee_filter<T0>>, C> operator|( const tee_filter<T0>& f, const C& c);

struct tee_device<Device,Sink> {
    using device_param=param_type<Device>::type; using sink_param=param_type<Sink>::type;
    using device_value=value_type<Device>::type; using sink_value=value_type<Sink>::type;
    using char_type = char_type_of<Device>::type;
    using mode = if_<is_convertible<category_of<Device>::type,output>, output, input>::type;
    struct category : mode, device_tag, closable_tag, flushable_tag, localizable_tag, optimally_buffered_tag {};
    ctor(device_param device, sink_param sink);
    streamsize read(char_type* s, streamsize n);
    streamsize write(const char_type* s, streamsize n);
    void close();
    bool flush();
    void imbue<Locale>(const Locale& loc);
    streamsize optimal_buffer_size() const;
private: device_value dev_; sink_value sink_;
};

tee_filter<Sink> tee<Sink>(<const> Sink& snk);
tee_device<Device,Sink> tee<Device,Sink>(<const> Device& dev, <const> Sink& sink); // 2x2
```

concepts

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
