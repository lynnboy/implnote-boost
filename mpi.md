# Boost.MPI

* lib: `boost/libs/mpi`
* repo: `boostorg/mpi`
* commit: `2fe4c4e`, 2025-05-06

------
#### Common Parts

```c++
// error
struct exception : public std::exception {
  ctor(const char* routine, int result_code); ~dtor() noexcept override(){}
  const char* what() const noexcept override { return message.c_str(); }
  const char* routine() const { return routine_; }
  int result_code() const { return result_code_; }
  int error_class() const; // MPI_Error_class(result_code_)
protected: const char* routine_; int result_code_; std::string message; // "<routine>: <error_string(result_code)>"
};
std::string error_string(int err); // MPI_Error_string
#define CHECK_RESULT(Func,Args) { int r=Func Args; assert(r==MPI_SUCCESS); if (r!=MPI_SUCCESS) throw_exception(exception{#Func,r}); }

struct allocator<T> {
  using value_type=T; using size_type=size_t; using differerence_type=ptrdiff_t;
  using <const>_pointer=<const>T*; using <const>_reference=<const>T&;
  struct rebind<U>{using other=allocator<U>;};
  ctor<U>(const self<U>&)noexcept{} // all spec-mem=default
  <const>_pointer address(<const>_reference x) const { return &x; }
  pointer allocate(size_type n, void*=0);// MPI_Alloc_mem
  void deallocate(pointer p, size_type); // MPI_Free_mem
  size_type max_size() const noexcept { return std::numeric_limits<size_t>::max() / sizeof(T); }
  void construct(pointer p, const T& val) { new (p) T{val}; }
  void destroy(pointer p) { ((T*)p)->~T(); }
};
struct allocator<void>{ using <const>_pointer=<const>void*; using value_type=void; struct rebind<U>{using other=allocator<U>;}; };
bool operator{==|!=}<T1,T2>(const allocator<T1>&, const allocator<T2>&) noexcept; // always equal

// inplace
struct inplace_t<T> { T& buffer; ctor(T& inout):buffer{inout}{} };
struct inplace_t<T*> { T* buffer; ctor(T* inout):buffer{inout}{} };
inplace_t<T> inplace<T>(T& input) { return {inout}; }
inplace_t<T*> inplace<T>(T* input) { return {inout}; }

// datatypes
struct detail::mpi_datatype_primitive {
  ctor(); ctor(void const*orig); // MPI_Get_address(orig) to origin
  using use_array_optimization=is_mpi_datatype<mpl::_1>;
  MPI_Datatype get_mpi_datatype(); // MPI_Type_create_struct with vectors, guard/set is_committed
  void save<T>()// push_back (&t, get_mpi_datatype(t), 1) to vectors
  void save_binary(void const* address, size_t count); // push_back (address, MPI_BYTE, count)
  void save_array<T>(serialization::array_wrapper<T> const& x, unsigned); // if x not empty: push_back (x.address(), get_mpi_datatype(x[0]), x.count())
private: std::vector<MPI_Aint> addresses; std::vector<MPI_Datatype> types; std::vector<int> lengths;
  bool is_committed{false}; MPI_Datatype datatype_; MPI_Aint origin{0};
};
struct detail::type_info_compare { bool operator()(std::type_info const* l, std::type_info const* r) const { return l->before(*r); } };
struct detail::mpi_datatype_map : noncopyable {
  ctor(){impl=new{};} ~dtor(){clear(); delete impl;}
  MPI_Datatype datatype<T>(const T& x={}) {
    if constexpr (is_mpi_builtin_datatype<T>()) return get_mpi_datatype<T>(x);
    else { auto t=&typeid(T); auto dt=get(&t); if (dt==MPI_DATATYPE_NULL) {mpi_datatype_oarchive ar(x); dt=ar.get_mpi_datatype(); set(t,dt); } return dt; }
  }
  void clear(); // MPI_Finalize, then MPI_Type_free each value-of-map, as well as get_mpi_datatype(bool{})
private: struct implementation { std::map<std::type_info const*, MPI_Datatype, type_info_compare> map; } *impl;
  MPI_Datatype get(const std::type_info* t); // map[t] or MPI_DATATYPE_NULL
  void set(const std::type_info* t, MPI_Datatype datatype); // map[t]=datatype
};
mpi_datatype_map& detail::mpi_datatype_cache();

struct is_mpi_builtin_datatype<T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_floating_point_datatype<T>,is_mpi_logical_datatype<T>,is_mpi_complex_datatype<T>,is_mpi_byte_datatype<T>>{};
struct is_mpi_integer_datatype<T> : mpl::false_{};
struct is_mpi_floating_point_datatype<T> : mpl::false_{};
struct is_mpi_logical_datatype<T> : mpl::false_{};
struct is_mpi_complex_datatype<T> : mpl::false_{};
struct is_mpi_byte_datatype<T> : mpl::false_{};
struct is_mpi_datatype<T> : is_mpi_builtin_datatype<T>{};
struct is_mpi_datatype<std::pair<T,U>> : mpl::and_<is_mpi_datatype<T>,is_mpi_datatype<U>>{};
struct is_mpi_datatype<std::array<T,n>> : is_mpi_datatype<T>{};

struct packed{};
MPI_Datatype get_mpi_datatype<T>(const T& x) { return mpi_datatype_cache().datatype(x); }
MPI_Datatype get_mpi_datatype<T>() { return get_mpi_datatype(T{}); }
// (CppType,MPIType,Kind): (packed, MPI_PACKED, builtin), (char, MPI_CHAR,builtin), (bool,MPI_C_BOOL,logical),
//    (short,MPI_SHORT,integer), (int,MPI_INT,integer), (long,MPI_LONG,integer),
//    (float,MPI_FLOAT,floating_point), (double,MPI_DOUBLE,floating_point), (long double,MPI_LONG_DOUBLE,floating_point),
//    (unsigned char,MPI_UNSIGNED_CHAR,builtin), (unsigned short,MPI_UNSIGNED_SHORT,integer),
//    (unsigned,MPI_UNSIGNED,integer), (unsigned long,MPI_UNSIGNED_LONG,integer),
//    (std::pair<float,int>,MPI_FLOAT_INT,builtin), (std::pair<double,int>,MPI_DOUBLE_INT,builtin),
//    (std::pair<long double,int>,MPI_LONG_DOUBLE_INT,builtin), (std::pair<long,int>,MPI_LONG_INT,builtin),
//    (std::pair<short,int>,MPI_SHORT_INT,builtin), (std::pair<int,int>,MPI_2INT,builtin),
//    (wchar_t,MPI_WCHAR,builtin), (long long,MPI_LONG_LONG_INT,builtin), (unsigned long long,MPI_UNSIGNED_LONG_LONG,builtin),
//    (signed char,MPI_SIGNED_CHAR,builtin),
//    (serialization::library_version_type,get_mpi_datatype(uint_least16_t{}),integer),
//    (archive::version_type, get_mpi_datatype(uint_least8_t()), integer),
//    (archive::class_id_type, get_mpi_datatype(int_least16_t()), integer),
//    (archive::class_id_reference_type, get_mpi_datatype(int_least16_t()), integer),
//    (archive::class_id_optional_type, get_mpi_datatype(int_least16_t()), integer),
//    (archive::object_id_type, get_mpi_datatype(uint_least32_t()), integer),
//    (archive::object_reference_type, get_mpi_datatype(uint_least32_t()), integer),
//    (archive::tracking_type, get_mpi_datatype(bool()), builtin),
//    (serialization::collection_size_type, get_mpi_datatype(std::size_t()), integer),
//    (serialization::item_version_type, get_mpi_datatype(uint_least8_t()), integer),
#define MPI_DATATYPE(CppType,MPIType,Kind) \
MPI_Datatype get_mpi_datatype<CppType>(const CppType&) { return MPIType; } \
struct is_mpi_<Kind>_datatype<CppType> : mpl::true_ {}
#define IS_MPI_DATATYPE(T) struct is_mpi_datatype<T>: mpl::true_{};

// status
struct status {
  ctor(){}  ctor(MPI_Status const& s) : m_status{s}{}
  int source() const { return m_status.MPI_SOURCE; }
  int tag() const { return m_status.MPI_TAG; }
  int error() const { return m_status.MPI_ERROR; }
  bool cancelled() const; // MPI_Test_cancelled
  optional<int> count() const { if (m_count!=-1) return m_count; // cache
    if constexpr (is_mpi_datatype<T>()) {
      // MPI_Get_count; return {} if failed, cache in m_count
    } else { return {}; }
  }
  operator <const> MPI_Status&() <const> { return m_status; }
private: mutable MPI_Status m_status; mutable int m_count{-1}
};

// timer
struct timer {
  ctor() {restart();}
  void restart() {start_time=MPI_Wtime();}
  double elapsed() const { return MPI_Wtime()-start_time;}
  double elapsed_max() const { return std::numeric_limits<double>::max(); } double elapsed_min() const { return MPI_Wtick(); }
  static bool time_is_global(); // MPI_Comm_get_attr(MPI_COMM_WORLD,MPI_WTIME_IS_GLOBAL)
private: double start_time;
};

// environment
enum threading::level{ single, funneled, serialized, multiple }; // << and >> for level
struct environment : noncopyable {
  explicit ctor(<threading::level mt_level>, bool abort_on_exception=true);
  ctor(int& argc, char**& argv, <threading::level mt_level>, bool abort_on_exception=true); // MPI_Init_<thread>, MPI_Comm_set_errhandler
  ~dtor() { if (i_initialized) if (uncaught_exceptions()>0 && abort_on_exception) abort(-1);
      else if (!finalized()) { mpi_datatype_cache().clear(); CHECK_RESULT(MPI_Finalize,()); } }
  [[noreturn]] static void abort(int errcode); // MPI_Abort
  static bool initialized(); // MPI_Initialized
  static bool finalized(); // MPI_Finalized
  static int max_tag(); // MPI_Comm_Attr_get(MPI_COMM_WORLD,MPI_TAG_UB) - num_reserved_tags
  static int collectives_tag() { return max_tag() + 1; }
  static optional<int> host_rank(); // MPI_Comm_Attr_get(MPI_COMM_WORLD,MPI_HOST)
  static optional<int> io_rank(); // MPI_Comm_Attr_get(MPI_COMM_WORLD,MPI_IO)
  static std::string processor_name(); // MPI_Get_processor_name
  static threading::level thread_level(); // MPI_Query_thread
  static bool is_main_thread(); // MPI_Is_thread_main
  static std::pair<int,int> version(); // MPI_Get_version
  static std::string library_version(); // MPI_Get_library_version
private: bool i_initialized, abort_on_exception; static const int num_reserved_tags=1;
};
```

#### Request

```c++
struct serialized_irecv_data<T> { size_t m_count; packed_iarchive m_ia; T& m_value; };
struct serialized_irecv_data<packed_iarchive> { size_t m_count; packed_iarchive& m_ia; };
struct serialized_array_irecv_data<T> { size_t m_count; packed_iarchive m_ia; T* m_value; int m_nb; };
struct dynamic_array_irecv_data<T,A> { size_t m_count; std::vector<T,A>& m_values; };
struct serialized_irecv_data<const skeleton_proxy<T>>
{ size_t m_count; packed_skeleton_iarchive m_isa; packed_iarchive& m_ia; skeleton_proxy<T> m_proxy; };
struct serialized_irecv_data<skeleton_proxy<T>> : self<const skeleton_proxy<T>>{};

struct detail::dynamic_primitive_array_data<A> { A& m_buffer; };
struct detail::serialized_data<T> { packed_iarchive m_archive; T& m_value; };
struct detail::serialized_data<packed_iarchive> { packed_iarchive& m_archive; };
struct detail::serialized_data<const skeleton_proxy<T>> { skeleton_proxy<T> m_proxy; packed_skeleton_iarchive m_archive; };
struct detail::serialized_data<skeleton_proxy<T>> : self<const skeleton_proxy<T>>{};
struct detail::serialized_array_data<T> { packed_iarchive m_archive; T* m_values; int m_nb; };

// request
struct request {
  ctor();
  static request make_trivial_send<T>(communicator const& comm, int dest, int tag, T const& value); // MPI_Isend
  static request make_trivial_send<T>(communicator const& comm, int dest, int tag, T const* values, int n);
  static request make_packed_send(communicator const& comm, int dest, int tag, void const* values, size_t n);
  static request make_bottom_send(communicator const& comm, int dest, int tag, MPI_Datatype tp);
  static request make_empty_send(communicator const& comm, int dest, int tag);

  static request make_trivial_recv<T>(communicator const& comm, int dest, int tag, T& value); // MPI_Irecv
  static request make_trivial_recv<T>(communicator const& comm, int dest, int tag, T* values, int n);
  static request make_bottom_recv(communicator const& comm, int dest, int tag, MPI_Datatype tp);
  static request make_empty_recv(communicator const& comm, int dest, int tag);

  static request make_dynamic(); // dynamic_handler
  // use probe_handler<XXX_data<>> on newer version, else use legacy_XXX_handler<>
  static request make_serialized<T>(communicator const& comm, int source, int tag, T& value); // 
  static request make_serialized_array<T>(communicator const& comm, int source, int tag, T* values, int n);
  static request make_dynamic_primitive_array_recv<T,A>(communicator const& comm, int source, int tag, std::vector<T,A>& values);
  static request make_dynamic_primitive_array_send<T,A>(communicator const& comm, int source, int tag, std::vector<T,A> const& values);

  status wait() { return m_handler ? m_handler->wait() : {}; }
  optional<status> test() { return active() ? m_handler->test() : {}; }
  void cancel() { if (m_handler) { m_handler->cancel(); } m_preserved.reset(); }
  optional<MPI_Request&> trivial() { return m_handler ? m_handler->trivial() : {}; }
  bool active() const { return m_handler && m_handler->active(); }
  void preserve(shared_ptr<void> d);

  struct handler { virtual ~dtor() =0;
    virtual status wait() =0;
    virtual optional<status> test() =0;
    virtual void cancel() =0;
    virtual bool active() const =0;
    virtual optional<MPI_Request&> trivial() =0;
  };
private:
  struct legacy_handler : handler {
    void cancel(); // MPI_Cancel the 2 requests
    bool active() const;
    optional<MPI_Request&> trivial();
    MPI_Request m_requests[2]; communicator m_comm; int m_source, m_tag;
  };
  struct trivial_handler : handler {
    status wait(); // MPI_Wait
    optional<status> test(); // MPI_Test
    void cancel();
    bool active() const;
    optional<MPI_Request&> trivial();
  private: MPI_Request m_request;
  };
  struct dynamic_handler : handler {
    status wait(); // MPI_Waitall
    optional<status> test(); // MPI_Testall
    void cancel();
    bool active() const;
    optional<MPI_Request&> trivial();
  private: MPI_Request m_request[2];
  };
  struct legacy_serialized_handler<T> : legacy_handler, protected serialized_irecv_data<T> {
    status wait(); // if req[1] is null, MPI_Wait then MPI_Irecv; MPI_Wait req[1]
    optional<status> test(); // if req[1] is null, MPI_Wait then MPI_Irecv; MPI_Test req[1]
  };
  struct legacy_serialized_array_handler<T> : legacy_handler, protected serialized_array_irecv_data<T> {
    status wait(); // if req[1] is null, MPI_Wait then MPI_Irecv; MPI_Wait req[1]
    optional<status> test(); // if req[1] is null, MPI_Wait then MPI_Irecv; MPI_Test req[1]
  };
  struct legacy_dynamic_primitive_array_handler<T,A> : legacy_handler, protected dynamic_array_irecv_data<T,A> {
    status wait(); // if req[1] is null, MPI_Wait then MPI_Irecv; MPI_Wait req[1]
    optional<status> test(); // if req[1] is null, MPI_Wait then MPI_Irecv; MPI_Test req[1]
  };
  struct probe_handler<Data> : handler, protected Data {
    bool active() const { return m_source != MPI_PROC_NULL; }
    optional<MPI_Request&> trivial() { return none; }
    void cancel() { m_source = MPI_PROC_NULL; }
    status wait(); // MPI_Mprobe, then unpack
    optional<status> test(); // MPI_Improbe, then unpack
  protected: communicator const& m_comm; int m_source, m_tag;
    status unpack(MPI_Message& msg, status& status); // MPI_Get_count, MPI_Mrecv
  };

  shared_ptr<handler> m_handler; shared_ptr<void> m_preserved;
};
```

#### Serialization

```c++
struct detail::ignore_skeleton_oarchive<Archive> : archive::detail::common_oarchive<Archive> {
  ctor():base{no_header}{}
protected:
  void save_override<T>(T const& t) { archive::save(*This(), t); }
  void save_override(std:;string const& t) { if (s.size()) save_override(serialization::make_array(s.data(),s.size())); }
  void save_override(#Type const&){}
      // archive::{class_id_<optional,reference>_type,version_type,object_id_type,object_reference_type,tracking_type,class_name_type}
      // serialization::{collection_size_type,library_version_type,item_version_type}
};
struct detail::mpi_datatype_oarchive : public mpi_datatype_primitive, public ignore_skeleton_oarchive<self> {
  ctor<T>(const T& x) : base{&x}{*this << x;}
  void save_orerride<T>(T const& t) {
    if constexpr (is_enum<T>()) { using int_type = uint_t<8*sizeof(T)>::least; save(*(int_type const*)&t); }
    else { base::save_orerride(t); }
  }
};

struct packed_iprimitive {
  using buffer_type = std::vector<char,allocator<char>>;
  ctor(buffer_type& b, MPI_Comm const& comm, int position=0);
  void const* address() const { return buffer_.data(); }
  const size_t& size() const { return size_=buffer_.size(); }
  void resize(size_t s) { buffer_.resize(s); }
  void load_binary(void* address, size_t count);
  void load_array<T>(serialization::array_wrapper<T> const& x, unsigned);
  using use_array_optimization = is_mpi_datatype<_1>;
  void load<T>(T& t);
  void load<Ch>(std::basic_string<Ch>& s); // size and Ch array
private: buffer_type& buffer_; mutable size_t size_; MPI_Comm comm; int position;
  void load_impl(void* p, MPI_Datatype t, int l); // MPI_Unpack
};
struct packed_oprimitive {
  using buffer_type = std::vector<char,allocator<char>>;
  ctor(buffer_type& b, MPI_Comm const& comm);
  void const* address() const { return buffer_.data(); }
  const size_t& size() const { return size_=buffer_.size(); }
  const size_t* size_ptr() const { return &size(); }
  void save_binary(void const* address, size_t count);
  void save_array<T>(serialization::array_wrapper<T> const& x, unsigned);
  using use_array_optimization = is_mpi_datatype<_1>;
  void save<T>(const T& t);
  void save<Ch>(const std::basic_string<Ch>& s); // size and Ch array
private: buffer_type& buffer_; mutable size_t size_; MPI_Comm comm;
  void save_impl(void const* p, MPI_Datatype t, int l); // MPI_Unpack
};

struct binary_buffer_iprimitive {
  using buffer_type = std::vector<char,allocator<char>>;
  ctor(buffer_type& b, MPI_Comm const&, int position=0);
  void <const>* address() <const> { return buffer_.data(); }
  const size_t& size() const { return size_=buffer_.size(); }
  void resize(size_t s) { buffer_.resize(s); }
  void load_binary(void* address, size_t count);
  void load_array<T>(serialization::array_wrapper<T> const& x, unsigned);
  using use_array_optimization = is_bitwise_serializable<_1>;
  void load<T>(serialization::array_wrapper<T> const& x) { load_array(x,0); }
  void load<T>(T& t);
  void load<Ch>(std::basic_string<Ch>& s); // size and Ch array
private: buffer_type& buffer_; mutable size_t size_; int position;
  void load_impl(void* p, int l); // memcpy from buffer_
};
struct binary_buffer_oprimitive {
  using buffer_type = std::vector<char,allocator<char>>;
  ctor(buffer_type& b, MPI_Comm const&);
  void const* address() const { return buffer_.data(); }
  const size_t& size() const { return size_=buffer_.size(); }
  const size_t* size_ptr() const { return &size(); }
  void save_binary(void const* address, size_t count);
  void save_array<T>(serialization::array_wrapper<T> const& x, unsigned);
  void save<T>(serialization::array_wrapper<T> const& x) { save_array(x,0); }
  using use_array_optimization = is_bitwise_serializable<_1>;
  void save<T>(const T& t);
  void save<Ch>(const std::basic_string<Ch>& s); // size and Ch array
private: buffer_type& buffer_; mutable size_t size_;
  void save_impl(void const* p, int l); // buffer_.insert
};

using iprimitive = binary_buffer_iprimitive; // HOMOGENEOUS
struct packed_iarchive : public iprimitive, archive::detail::common_iarchive<self> {
  ctor(MPI_Comm const& comm, buffer_type& b, unsigned flags=no_header, int position=0);
  ctor(MPI_Comm const& comm, size_t s=0, unsigned flags=no_header); // use internal_buffer_
  void load_override<T>(T& x) {
    if constexpr (apply<use_array_optimization,remove_const_t<T>>()) iprimitive::load(x);
    else common_iarchive<self>::load_override(x);
  }
  void load_override(archive::class_id_optional_type&){}
  void load_override(archive::class_id_type& t); // *This()>>t
  void load_override(archive::version_type& t); // *This()>>t
  void load_override(archive::class_id_reference_type& t); // *This()>>t
  void load_override(archive::class_name_type& t); // *This()>>str
private: buffer_type internal_buffer_;
};
using iprimitive = packed_iprimitive; // Otherwise
struct packed_oarchive : public oprimitive, archive::detail::common_oarchive<self> {
  ctor(MPI_Comm const& comm, buffer_type& b, unsigned flags=no_header);
  ctor(MPI_Comm const& comm, unsigned flags=no_header); // use internal_buffer_
  void save_override<T>(T const& x) {
    if constexpr (apply<use_array_optimization,T>()) oprimitive::save(x);
    else common_oarchive<self>::save_override(x);
  }
  void save_override(const archive::class_id_optional_type&){}
  void save_override(const archive::class_id_type& t); // *This()<<t
  void save_override(const archive::version_type& t); // *This()<<t
  void save_override(const archive::class_name_type& s); // *This()<<s
private: buffer_type internal_buffer_;
};
```


collectives/: all_to_all, broadcast, <all>_gather<v>, <all>_reduce, scan, scatter<v>
detail/: broadcast_sc, communicator_sc, computation_tree, content_oarchive,
  forward_{i,o}primitive, forward_skeleton_{i,o}archive, ignore_{i,o}primitive,
	offsets, point_to_point, text_skeleton_oarchive
python/: config, serialize, skeleton_and_content

<cartesian,graph>_communicator, collectives_<fwd>, group, intercommunicator, nonblocking,
operations, python, skeleton_and_content_<fwd,types>

src/: broadcast, <cartesian,graph>_communicator, computation_tree, content_oarchive, intercommunicator,
  offsets, packed_skeleton_{i,o}archive, point_to_point, text_skeleton_oarchive

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`.

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/auto_link.hpp>`
* `<boost/limits.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`, `<boost/utility/enable_if.hpp>`
* `<boost/core/uncaught_exceptions.hpp>`
* `<boost/noncopyable.hpp>`

#### Boost.Foreach

* `<boost/foreach.hpp>`

#### Boost.Function

* `<boost/function/function1.hpp>`
* `<boost/function/function3.hpp>`

#### Boost.Graph

* `<boost/graph/graph_traits.hpp>`
* `<boost/graph/iteration_macros.hpp>`
* `<boost/graph/properties.hpp>`

#### Boost.Integer

* `<boost/integer.hpp>`.

#### Boost.Iterator

* `<boost/iterator/counting_iterator.hpp>`

#### Boost.LexicalCast

* `<boost/lexical_cast.hpp>`

#### Boost.MPL

* `<boost/mpl/and.hpp>`
* `<boost/mpl/always.hpp>`
* `<boost/mpl/assert.hpp>`
* `<boost/mpl/bool.hpp>`
* `<boost/mpl/if.hpp>`
* `<boost/mpl/or.hpp>`
* `<boost/mpl/placeholders.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.Python

* `<boost/python.hpp>`
* `<boost/python/extract.hpp>`
* `<boost/python/list.hpp>`
* `<boost/python/object.hpp>`
* `<boost/python/stl_iterator.hpp>`
* `<boost/python/str.hpp>`
* `<boost/python/suite/indexing/vector_indexing_suite.hpp>`

#### Boost.Serialization

* `<boost/archive/**.hpp>`
* `<boost/serialization/**.hpp>`

#### Boost.SmartPtr

* `<boost/scoped_array.hpp>`, `<boost/smart_ptr/scoped_array.hpp>`
* `<boost/shared_array.hpp>`
* `<boost/shared_ptr.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`.

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`.

#### Boost.TypeTraits

* `<boost/type_traits/is_enum.hpp>`
* `<boost/type_traits/is_fundamental.hpp>`
* `<boost/type_traits/remove_const.hpp>`

#### Boost.utility

* `<boost/operators.hpp>`

------
### Standard Facilities
