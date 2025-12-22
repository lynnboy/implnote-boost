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

// operations
struct is_mpi_op<Op,T> : mpl::false_{};
struct is_commutative<Op,T> : mpl::false_{};
struct maximum<T>; // functor of max()
struct minimum<T>; // functor of min()
struct bitwise_and<T>; // x&y
struct bitwise_or<T>; // x|y
struct logical_xor<T>; // (x||y)&&!(x&&y)
struct bitwise_xor<T>; // x^y
struct is_mpi_op<maximum<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_floating_point_datatype<T>>{ static MPI_Op op() { return MPI_MAX;} };
struct is_mpi_op<minimum<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_floating_point_datatype<T>>{ static MPI_Op op() { return MPI_MIN;} };
struct is_mpi_op<std::plus<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_floating_point_datatype<T>,is_mpi_complex_datatype<T>>{ static MPI_Op op() { return MPI_SUM;} };
struct is_mpi_op<std::multiplies<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_floating_point_datatype<T>,is_mpi_complex_datatype<T>>{ static MPI_Op op() { return MPI_PROD;} };
struct is_mpi_op<std::logical_and<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_logical_datatype<T>>{ static MPI_Op op() { return MPI_LAND;} };
struct is_mpi_op<std::logical_or<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_logical_datatype<T>>{ static MPI_Op op() { return MPI_LOR;} };
struct is_mpi_op<logical_xor<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_logical_datatype<T>>{ static MPI_Op op() { return MPI_LXOR;} };
struct is_mpi_op<bitwise_and<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_byte_datatype<T>>{ static MPI_Op op() { return MPI_BAND;} };
struct is_mpi_op<bitwise_or<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_byte_datatype<T>>{ static MPI_Op op() { return MPI_BOR;} };
struct is_mpi_op<bitwise_xor<T>,T> : mpl::or_<is_mpi_integer_datatype<T>,is_mpi_byte_datatype<T>>{ static MPI_Op op() { return MPI_BXOR;} };
struct detail::user_op<Op,T> {
  ctor(); // MPI_Op_create
  ~dtor(); // MPI_Op_free
  MPI_Op& get_mpi_op() { return mpi_op; }
private: MPI_Op mpi_op;
  static void perform(void* vinvec, void* voutvec, int* plen, MPI_Datatype*) { std::transform(invec,invec+*plen, outvec, outvec, Op{}); }
};
```

#### Collective Operations

```c++
void detail::size2offsets(int const* sizes, int* offsets, int n);
void detail::sizes2offsets(std::vector<int> const& sizes, std::vector<int>& offsets);
void detail::offsets2skipped(int const* sizes, int const* offsets, int* skipped, int n);
int* detail::make_offsets(communicator const& comm, int const* sizes, int const* displs, int root = -1);
int* detail::make_skipped_slots(communicator const& comm, int const* sizes, int const* displs, int root = -1);

struct detail::computation_tree {
  ctor(int rank, int size, int root, int branching_factor=-1);
  int branching_factor() const { return branching_factor_; }
  int level() const { return level_; }
  int level_index(int n) const;
  int parent() const;
  int child_begin() const;
  static int default_branching_factor = 3;
protected: int rank, size, root, branching_factor_, level_;
};

void all_gather<T>(const communicator& comm, const T& in_value, std::vector<T>& out_values);
void all_gather<T>(const communicator& comm, const T& in_value, T* out_values);
void all_gather<T>(const communicator& comm, const T* in_values, int n, std::vector<T>& out_values);
void all_gather<T>(const communicator& comm, const T* in_values, int n, T* out_values);
void all_gatherv<T>(const communicator& comm, const T& in_value, T* out_values, const std::vector<int>& sizes);
void all_gatherv<T>(const communicator& comm, const T* in_values, T* out_values, const std::vector<int>& sizes);
void all_gatherv<T>(const communicator& comm, std::vector<T> const& in_values,  std::vector<T>& out_values, const std::vector<int>& sizes);
void all_gatherv<T>(const communicator& comm, const T& in_value, T* out_values, const std::vector<int>& sizes, const std::vector<int>& displs);
void all_gatherv<T>(const communicator& comm, const T* in_values, T* out_values, const std::vector<int>& sizes, const std::vector<int>& displs);
void all_gatherv<T>(const communicator& comm, std::vector<T> const& in_values, std::vector<T>& out_values, const std::vector<int>& sizes, const std::vector<int>& displs);

void all_reduce<T,Op>(const communicator& comm, const T* value, int n, T* out_value, Op op);
void all_reduce<T,Op>(const communicator& comm, const T& value, T& out_value, Op op);
T    all_reduce<T,Op>(const communicator& comm, const T& value, Op op);
void all_reduce<T,Op>(const communicator& comm, inplace_t<T*> value, int n, Op op);
void all_reduce<T,Op>(const communicator& comm, inplace_t<T> value, Op op);

void all_to_all<T>(const communicator& comm, const std::vector<T>& in_values, std::vector<T>& out_values);
void all_to_all<T>(const communicator& comm, const T* in_values, T* out_values);
void all_to_all<T>(const communicator& comm, const std::vector<T>& in_values, int n, std::vector<T>& out_values);
void all_to_all<T>(const communicator& comm, const T* in_values, int n, T* out_values);

void broadcast<T>(const communicator& comm, T& value, int root);
void broadcast<T>(const communicator& comm, T* values, int n, int root);
void broadcast<T>(const communicator& comm, skeleton_proxy<T>& value, int root);
void broadcast<T>(const communicator& comm, const skeleton_proxy<T>& value, int root);

void gather<T>(const communicator& comm, const T& in_value, std::vector<T>& out_values, int root);
void gather<T>(const communicator& comm, const T& in_value, T* out_values, int root);
void gather<T>(const communicator& comm, const T& in_value, int root);
void gather<T>(const communicator& comm, const T* in_values, int n, std::vector<T>& out_values, int root);
void gather<T>(const communicator& comm, const T* in_values, int n, T* out_values, int root);
void gather<T>(const communicator& comm, const T* in_values, int n, int root);
void gatherv<T>(const communicator& comm, const std::vector<T>& in_values, T* out_values, const std::vector<int>& sizes, const std::vector<int>& displs, int root);
void gatherv<T>(const communicator& comm, const T* in_values, int in_size, T* out_values, const std::vector<int>& sizes, const std::vector<int>& displs, int root);
void gatherv<T>(const communicator& comm, const std::vector<T>& in_values, int root);
void gatherv<T>(const communicator& comm, const T* in_values, int in_size, int root);
void gatherv<T>(const communicator& comm, const T* in_values, int in_size, T* out_values, const std::vector<int>& sizes, int root);
void gatherv<T>(const communicator& comm, const std::vector<T>& in_values, T* out_values, const std::vector<int>& sizes, int root);

void scatter<T>(const communicator& comm, const std::vector<T>& in_values, T& out_value, int root);
void scatter<T>(const communicator& comm, const T* in_values, T& out_value, int root);
void scatter<T>(const communicator& comm, T& out_value, int root);
void scatter<T>(const communicator& comm, const std::vector<T>& in_values, T* out_values, int n, int root);
void scatter<T>(const communicator& comm, const T* in_values, T* out_values, int n, int root);
void scatter<T>(const communicator& comm, T* out_values, int n, int root);
void scatterv<T>(const communicator& comm, const std::vector<T>& in_values, const std::vector<int>& sizes, const std::vector<int>& displs, T* out_values, int out_size, int root);
void scatterv<T>(const communicator& comm, const T* in_values, const std::vector<int>& sizes, const std::vector<int>& displs, T* out_values, int out_size, int root);
void scatterv<T>(const communicator& comm, T* out_values, int out_size, int root);
void scatterv<T>(const communicator& comm, const T* in_values, const std::vector<int>& sizes, T* out_values, int root);
void scatterv<T>(const communicator& comm, const std::vector<T>& in_values, const std::vector<int>& sizes, T* out_values, int root);

void reduce<T,Op>(const communicator& comm, const T& in_value, T& out_value, Op op, int root);
void reduce<T,Op>(const communicator& comm, const T& in_value, Op op, int root);
void reduce<T,Op>(const communicator& comm, const T* in_values, int n, T* out_values, Op op, int root);
void reduce<T,Op>(const communicator& comm, const T* in_values, int n, Op op, int root);

void scan<T,Op>(const communicator& comm, const T& in_value, T& out_value, Op op);
T    scan<T,Op>(const communicator& comm, const T& in_value, Op op);
void scan<T,Op>(const communicator& comm, const T* in_values, int n, T* out_values, Op op);
```

#### Request & Non-blocking

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

std::pair<status,FwdIt> wait_any<FwdIt>(FwdIt first, FwdIt last); // MPI_Waitany
optional<std::pair<status,FwdIt>> test_any<FwdIt>(FwdIt first, FwdIt last); // item.test()
OutIt wait_all<FwdIt,OutIt>(FwdIt first, FwdIt last, OutIt out); // MPI_Waitall
void wait_all<FwdIt>(FwdIt first, FwdIt last); // MPI_Waitall
optional<OutIt> test_all<FwdIt,OutIt>(FwdIt first, FwdIt last, OutIt out); // MPI_Testall
bool test_all<FwdIt>(FwdIt first, FwdIt last); // MPI_Testall
std::pair<OutIt,BidiIt> wait_some<BidiIt,OutIt>(BidiIt first, BidiIt last, OutIt out); // MPI_Waitsome
BidiIt wait_some<BidiIt>(BidiIt first, BidiIt last); // MPI_Waitsome
std::pair<OutIt,BidiIt> test_some<BidiIt,OutIt>(BidiIt first, BidiIt last, OutIt out); // item.test()
BidiIt test_some<BidiIt>(BidiIt first, BidiIt last); // item.test()

void detail::packed_archive_send(communicator const& comm, int dest, int tag, const packed_oarchive& ar); // MPI_Send for size and buffer
request detail::packed_archive_isend(communicator const& comm, int dest, int tag, const packed_oarchive& ar); // request::make_packed_send
request detail::packed_archive_isend(communicator const& comm, int dest, int tag, const packed_iarchive& ar); // request::make_packed_send
void detail::packed_archive_recv(communicator const& comm, int dest, int tag, const packed_iarchive& ar, MPI_Status& status); // MPI_Mprobe, MPI_Get_count, MPI_Mrecv
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

#### Communicators

```c++
const int any_source = MPI_ANY_SOURCE, any_tag = MPI_ANY_TAG;
enum comm_create_kind { comm_duplicate, comm_take_ownership, comm_attach };

struct group {
  ctor();
  ctor(const MPI_Group& in_group, bool adopt);
  optional<int> rank() const;
  int size() const;
  OutIt translate_ranks<InIt,OutIt>(InIt first, InIt last, const group& to_group, OutIt out); // MPI_Group_translate_ranks
  operator bool() const { return group_ptr; }
  operator MPI_Group() const; // *group_ptr or MPI_GROUP_EMPTY
  group include<InIt>(InIt first, InIt last); // MPI_Group_incl
  group exclude<InIt>(InIt first, InIt last); // MPI_Group_excl
protected: shared_ptr<MPI_Group> group_ptr;
  struct group_free {}; // functor of if (MPI_Finalized) MPI_Group_free
};
bool operator==(const group& g1, const group& g2);
bool operator!=(const group& g1, const group& g2) { return !(g1==g2); }
group operator|(const group& g1, const group& g2);
group operator&(const group& g1, const group& g2);
group operator-(const group& g1, const group& g2);

struct communicator {
  ctor();
  ctor(const MPI_Comm& comm, comm_create_kind kind); // MPI_Comm_dup for comm_duplicate
  ctor(const communicator& comm, const group& subgroup); // MPI_Comm_create
  int rank() const; // MPI_Comm_rank
  int size() const; // MPI_Comm_size
  group group() const; // MPI_Comm_group

  void send<T>(int dest, int tag, const T& value) const;
  void send<T,A>(int dest, int tag, const std::vector<T,A>& values) const;
  void send<T>(int dest, int tag, const skeleton_proxy<T>& proxy) const;
  void send<T>(int dest, int tag, const T* values, int n) const;
  void send(int dest, int tag) const; // MPI_Send
  status recv<T>(int source, int tag, T& value) const;
  status recv<T,A>(int source, int tag, std::vector<T,A>& values) const;
  status recv<T>(int source, int tag, const skeleton_proxy<T>& proxy) const;
  status recv<T>(int source, int tag, skeleton_proxy<T>& value) const;
  status recv<T>(int source, int tag, T* values, int n) const;
  status recv(int source, int tag) const; // MPI_Recv

  status sendrecv<T>(int dest, int stag, const T& sval, int src, int rtag, T& rval) const;

  request isend<T>(int dest, int tag, const T& value) const;
  request isend<T>(int dest, int tag, const skeleton_proxy<T>& proxy) const;
  request isend<T>(int dest, int tag, const T* values, int n) const;
  request isend<T,A>(int dest, int tag, const std::vector<T,A>& values) const;
  request isend(int dest, int tag) const;
  request irecv<T>(int source, int tag, T& value) const;
  request irecv<T>(int source, int tag, T* values, int n) const;
  request irecv<T>(int source, int tag, std::vector<T,A>& values) const;
  request irecv(int source, int tag) const;

  status probe(int source=any_source, int tag=any_tag) const; // MPI_Probe
  optional<status> iprobe(int source=any_source, int tag=any_tag) const; // MPI_Iprobe

  void barrier() const; // MPI_Barrier
  operator bool() const { return comm_ptr; }
  operator MPI_Comm() const;

  communicator split(int color, int key) const; // MPI_Comm_split
  communicator split(int color) const;
  optional<intercommunicator> as_intercommunicator() const; // MPI_Comm_test_inter
  optional<graph_communicator> as_graph_communicator() const;
  bool has_graph_topology() const;// MPI_Topo_test for MPI_GRAPH
  optional<cartesian_communicator> as_cartesian_communicator() const;
  bool has_cartesian_topology() const; // MPI_Topo_test for MPI_CART
  void abort(int errcode) const; // MPI_Abort
protected: shared_ptr<MPI_Comm> comm_ptr;
  struct comm_free {}; // functor for if (MPI_Finalized) MPI_Comm_free
  status sendrecv_impl<T>(int dest, int stag, const T& sval, int src, int rtag, T& rval, mpl::{true|false}_) const; // MPI_Sendrecv
  void send_impl<T>(int dest, int tag, const T& value, mpl::{true|false}_) const; // MPI_Send for MPI datatypes, or send packed_oarchive
  void array_send_impl<T>(int dest, int tag, const T* values, int n, mpl::{true|false}_) const;
  request isend_impl<T>(int dest, int tag, const T& value, mpl::{true|false}_) const;
  request array_isend_impl<T>(int dest, int tag, const T* values, int n, mpl::{true|false}_) const; // make_trivial_send or isend packed_oarchive
  status recv_impl<T>(int source, int tag, T& value, mpl::{true|false}_) const; // MPI_Recv or recv packed_iarchive
  status array_recv_impl<T>(int source, int tag, T* values, int n, mpl::{true|false}_) const;
  request irecv_impl<T>(int source, int tag, T& value, mpl::{true|false}_) const; // make_trivial_recv or make_serialized
  request array_irecv_impl<T>(int source, int tag, T* values, int n, mpl::{true|false}_) const; // make_trivial_recv or make_serialized_array
  void send_vector(int dest, int tag, const std::vector<T,A>& value, mpl::{true|false}_) const;
  status recv_vector(int source, int tag, std::vector<T,A>& value, mpl::{true|false}_) const;
  request isend_vector<T,A>(int dest, int tag, const std::vector<T,A>& values, mpl::{true|false}_) const; // make_dynamic_primitive_array_send
  request irecv_vector<T,A>(int source, int tag, std::vector<T,A>& values, mpl::{true|false}_) const; // make_dynamic_primitive_array_recv
};
bool operator==(const communicator& comm1, const communicator& comm2);
bool operator!=(const communicator& comm1, const communicator& comm2) { return !(comm1==comm2); }

struct intercommunicator : public communicator {
  ctor(const MPI_Comm& comm, comm_create_kind kind);
  ctor(const communicator& local, int local_leader, const communicator& peer, int remote_leader); // MPI_Intercomm_create
  int local_size() const { return size(); }
  group local_group() const { return group(); }
  int local_rank() const { return rank(); }
  int remote_size() const; // MPI_Comm_remote_size
  group remote_group() const; // MPI_Comm_remote_group
  communicator merge(bool high) const; // MPI_Intercomm_merge
private: explicit ctor(const shared_ptr<MPI_Comm>& comm_ptr);
};

struct graph_communicator : public communicator {
  ctor(const MPI_Comm& comm, comm_create_kind kind);
  ctor<Graph>(const communicator& comm, const Graph& graph, bool reorder=false);
  ctor<Graph,RankMap>(const communicator& comm, const Graph& graph, RankMap rank, bool reorder=false);
protected:
  void setup_graph<Graph,RankMap>(const communicator& comm, const Graph& graph, RankMap rank, bool reorder); // MPI_Graph_create
private: explicit ctor(const shared_ptr<MPI_Comm>& comm_ptr);
};
struct detail::comm_out_edge_iterator : public iterator_facade<self, std::pair<int,int>, random_access_traversal_tag, const std::pair<int,int>& int>;
struct detail::comm_adj_iterator : public iterator_facade<self, int, random_access_traversal_tag, int, int>;
struct detail::comm_edge_iterator : public iterator_facade<self, std::pair<int,int>, forward_traversal_tag, const std::pair<int,int>&, int>;
int source(const std::pair<int,int>& edge, const graph_communicator&);
int target(const std::pair<int,int>& edge, const graph_communicator&);
std::pair<comm_out_edge_iterator,comm_out_edge_iterator> out_edges(int vertex, const graph_communicator& comm); // MPI_Graph_neighbors
int out_degree(int vertex, const graph_communicator& comm);// MPI_Graph_neighbors_count
std::pair<comm_adj_iterator,comm_adj_iterator> adjacent_vertices(int vertex, const graph_communicator& comm); // MPI_Graph_neighbors
std::pair<counting_iterator<int>,counting_iterator<int>> vertices(const graph_communicator& comm) { return make_pair(0,comm.size()); }
int num_vertices(const graph_communicator& comm) { return comm.size(); }
std::pair<comm_edge_iterator,comm_edge_iterator> edges(const graph_communicator& comm); // MPI_Graphdims_get, MPI_Graph_get
int num_edges(const graph_communicator& comm); // MPI_Graphdims_get
identity_property_map get(vertex_index_t, const graph_communicator&) { return {}; }
inline int get(vertex_index_t, const graph_communicator&, int vertex) { return vertex; }
struct graph_traits<graph_communicator> {
  using vertex_descriptor=int; using edge_descriptor=std::pair<int,int>;
  using directed_category=directed_tag; using edge_parallel_category=disallow_parallel_edge_tag;
  struct traversal_category : public  incidence_graph_tag, adjacency_graph_tag, vertex_list_graph_tag, edge_list_graph_tag {};
  static vertex_descriptor null_vertex() { return -1; }
  using out_edge_iterator = comm_out_edge_iterator; using degree_size_type=int;
  using adjacency_iterator=comm_adj_iterator;
  using vertex_iterator=counting_iterator<int>; using vertices_size_type=int;
  using edge_iterator = comm_edge_iterator; using edges_size_type=int;
};
struct property_map<graph_communicator,vertex_index_t> { using <const>_type = identity_property_map; };

struct cartesian_dimension {
  int size; bool periodic;
  ctor(int sz=0, bool p=false);
private: void serialize<Archive>(Archive& ar, unsigned);
};
struct is_mpi_datatype<cartesian_dimension> : mpl::true_{};
bool operator{==|!=}(cartesian_dimension const& d1, cartesian_dimension const& d2);
std::ostream& operator<<(std::ostream& out, cartesian_dimension const& d);
class cartesian_topology : private std::vector<cartesian_dimension> {
  ctor()=delete; ctor(self {const&|&&})=default; self& operator=(self {const&|&&})=default; ~dtor()=default;
  ctor(int ndim); ctor(std::vector<cartesian_dimension> const& dims);
  explicit ctor<InitArr>(InitArr dims); explicit ctor(std::initializer_list<cartesian_dimension> dims);
  explicit ctor<ndim>(cartesian_dimension (&dims)[ndim]);
  ctor<DimRg,PerRg>(DimRg const& dim_rg, PerRg const& period_rg);
  ctor<DimIt,PerIt>(DimIt dit, PerIt pit, int n);
  std::vector<cartesian_dimension> <const>& stl() <const> { return *this; }
  void split(std::vector<int>& dims, std::vector<bool>& periodics) const;
};
bool operator{==|!=}(cartesian_topology const& t1, cartesian_topology const& t2);
std::ostream& operator<<(std::ostream& out, cartesian_topology const& t);
struct cartesian_communicator : public communicator {
  ctor(const MPI_Comm& comm, comm_create_kind kind);
  ctor(const communicator& comm, const cartesian_topology& dims, bool reorder=false); // MPI_Cart_create
  ctor(const cartesian_communicator& comm, const std::vector<int>& keep); // MPI_Cart_sub
  int ndims() const; // MPI_Cartdim_get
  int rank(const std::vector<int>& coords) const; // MPI_Cart_rank
  std::pair<int,int> shifted_ranks(int dim, int disp) const; // MPI_Cart_shift
  std::vector<int> coordinates(int rk) const; // MPI_Cart_coords
  void topology(cartesian_topology& dims, std::vector<int>& coords) const; // MPI_Cart_get
  cartesian_topology topology() const;
private: explicit ctor(const shared_ptr<MPI_Comm>& comm_ptr);
};
std::vector<int>& cartesian_dimensions(int nb_proc, std::vector<int>& dims); // MPI_Dims_create
```

#### Skeleton and Content

```c++
struct detail::forward_skeleton_iarchive<Ar,ImplAr> : archive::detail::common_iarchive<Ar> {
  ctor():base{no_header}, implementation_archive(ar){}
protected: ImplAr& implementation_archive;
  void load_override<T>(T& t) { archive::load(*This(), t); }
  void load_override(std:;string& t) { serialization::collection_size_type length; load_override(length); s.resize(length); }
  void load_override(#Type const&) { implementation_archive >> t; }
      // archive::{class_id_<optional,reference>_type,version_type,object_id_type,object_reference_type,tracking_type,class_name_type}
      // serialization::{collection_size_type,library_version_type,item_version_type}
};
struct detail::forward_skeleton_oarchive<Ar,ImplAr> : archive::detail::common_oarchive<Ar> {
  using implementation_archive_type=ImplAr;
  ctor(ImplAr& ar) : base(no_header), implementation_archive{ar}{}
protected: ImplAr& implementation_archive;
  void save_override<T>(T const& t) { archive::save(*This(), t); }
  void save_override(std::string const& t) { save_override(serialization::collection_size_type(t.size())); }
  void save_override<T>(T const& t) { implementation_archive << t; }
      // for T in: archive::{class_id_<optional,reference>_type,version_type,object_id_type,object_reference_type,tracking_type,class_name_type}
      // serialization::{collection_size_type,library_version_type,item_version_type}
};

struct detail::ignore_iprimitive {
  void load_binary(void*, size_t){}
  void load_array<T>(serialization::array_wrapper<T>&, unsigned){}
  using use_array_optimization=is_mpi_datatype<_1>;
protected: void load<T>(T&){}
};
struct detail::ignore_oprimitive {
  void save_binary(const void*, size_t){}
  void save_array<T>(serialization::array_wrapper<T> const&, unsigned){}
  using use_array_optimization=is_mpi_datatype<_1>;
protected: void save<T>(const T&){}
};

struct skeleton_proxy<T> { ctor(T& x):object{x}{} T& object; };
const skeleton_proxy<T> skeleton<T>(T& x) { return {x}; }
struct detail::mpi_datatype_holder : noncopyable {
  ctor(){} ctor(MPI_Datatype t, bool committed=true);
  void commit() { CHECK_RESULT(MPI_Type_commit,(&d)); is_committed=true; }
  ~dtor(); // if (MPI_Finalized) MPI_Type_free
private: MPI_Datatype d; bool is_committed=false;
};

struct content {
  ctor(){} ctor(MPI_Datatype d, bool committed=true) : holder{new mpi_datatype_holder{d,committed}}{}
  const content& operator=(MPI_Datatype d) { holder.reset(new mpi_datatype_holder{d}); return *this; }
  MPI_Datatype get_mpi_datatype() const { return holder->get_mpi_datatype(); }
  void commit() { holder->commit(); }
private: shared_ptr<mpi_datatype_holder> holder;
};
const content get_content<T>(const T& x);

struct packed_skeleton_iarchive : public ignore_iprimitive, forward_skeleton_iarchive<self,packed_iarchive> {
  ctor(MPI_Comm const& comm, unsigned flags=no_header) :base{skeleton_archive_}, skeleton_archive_{comm,flags}{}
  explicit ctor(packed_iarchive& ar) : base{ar}, skeleton_archive_{MPI_COMM_WORLD,no_header}{}
  <const> packed_iarchive& get_skeleton() <const> { return implementation_archive; }
private: packed_iarchive skeleton_archive_;
};
struct packed_skeleton_oarchive : public ignore_oprimitive, forward_skeleton_oarchive<self,packed_oarchive> {
  ctor(MPI_Comm const& comm, unsigned flags=no_header) :base{skeleton_archive_}, skeleton_archive_{comm,flags}{}
  explicit ctor(packed_oarchive& ar) : base{ar}, skeleton_archive_{MPI_COMM_WORLD,no_header}{}
  <const> packed_oarchive& get_skeleton() <const> { return implementation_archive; }
private: packed_oarchive skeleton_archive_;
};

struct detail::content_oarchive : public mpi_datatype_primitive, public ignore_skeleton_oarchive<self> {
  content get_content() { if (!committed) { c=get_mpi_datatype(); committed=true;} return c; }
private: bool committed={false}; content c;
};
const content detail::get_content<T>(const T& x) { content_oarchive ar; ar<<x; return ar.get_content(); }

struct text_skeleton_oarchive : public ignore_oprimitive, forward_skeleton_oarchive<self,archive::text_oarchive> {
  ctor(std::ostream& s, unsigned flags=0) base{skeleton_archive_}, skeleton_archive_{s,flags}{}
private: text_oarchive skeleton_archive_;
};
```

#### Python

```c++
namespace python {
void register_serialized<T>(const T& value=T{}, PyTypeObject* type=0);
void register_skeleton_and_content<T>(const T& value=T{}, PyTypeObject* type=0);
}

// serialization
namespace ::boost::python{
struct pickle {
  static object dumps(object obj, int protocol=-1);
  static object loads(object s);
private:
  static void initialize_data();
  static struct data_t* data;
};
struct has_direct_serialization<IAr,OAr> : mpl::false_{};
struct output_archiver<IAr>{};
struct input_archiver<OAr>{};
struct detail::direct_serialization_table<IAr,OAr> {
  using saver_t = function<void(OAr&, const object&, unsigned)>;
  using loader_t = function<void(IAr&, object&, unsigned)>;
  using savers_t = std::map<PyTypeobject*,std::pair<int,saver_t>>;
  using loaders_t = std::map<int,loader_t>;
  saver_t saver(const object& obj, int& descriptor);
  loader_t loader(int descriptor);
  void register_type<T>(const T& value=T{}, PyTypeObject* type=0);
  void register_type<T>(const saver_t& saver, const loader_t& loader, const T& value=T{}, PyTypeObject* type=0);
protected:
  struct default_saver<T>; // ar << extract<T>(obj)();
  struct default_loader<T>; // ar >> extract<T&>(obj)(); OR ar>>value; obj=object(value)
  savers_t savers; loaders_t loaders;
};
direct_serialization_table<IAr,OAr>& detail::get_direct_serialization_table<IAr,OAr>();
void register_serialized<IAr,OAr,T>(const T& value=T{}, PyTypeObject* type=0);
void save<Ar>(Ar& ar, const object& obj, unsigned ver);
void load<Ar>(Ar& ar, object& obj, unsigned ver);
void serialize<Ar>(Ar& ar, object& obj, unsigned ver);
}

// skeleton_and_content
namespace python {
struct content : mpi::content {
  ctor(const base& base, python::object object);
  <const> base& base() <const> { return *this; }
  python::object object;
};
struct skeleton_proxy_base { python::object object; };
struct skeleton_proxy<T> : public skeleton_proxy_base {};
object detail::skeleton_proxy_base_type;
struct detail::skeleton_saver<T>; // packed_skeleton_oarchive{ar} << extract<T&>(obj.attr("object"))();
struct detail::skeleton_loader<T>; // packed_skeleton_iarchive{ar} >> extract<T&>(obj.attr("object"))();
struct detail::skeleton_content_handler{ function<object(const object&)> get_skeleton_proxy, get_content; };
struct detail::do_get_skeleton_proxy<T>; // object{skeleton_proxy<T>{value}};
struct detail::do_get_content<T>; // content{get_content(extract<T&>(value_obj)()), value_obj}
bool detail::skeleton_and_content_handler_registered(PyTypeObject* type);
bool detail::register_skeleton_and_content_handler(PyTypeObject*, const skeleton_content_handler&);
void register_skeleton_and_content<T>(const T& value, PyTypeObject* type);
}
```

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
