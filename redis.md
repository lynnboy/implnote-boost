# Boost.Redis

* lib: `boost/libs/redis`
* repo: `boostorg/redis`
* commit: `203e929`, 2025-09-22

------
### RESP3 Protocol Client

```c++
enum class resp3::type { array, push, set, map, attribute, simple_string, // [*>~%|+]
  simple_error, number, doublean, boolean, big_number, null, blob_error, // [-:,#(_!]
  verbatim_string, blob_string, streamed_string, streamed_string_part, invalid, // [=$ ; ]
};
auto resp3::to_string(type t) noexcept -> char const*;
auto operator<<(std::ostream& os, type t) -> std::ostream&;
constexpr auto resp3::is_aggregate(type t) noexcept -> bool; // array, push, set, map, attribute
constexpr auto resp3::element_multiplicity(type t) noexcept -> size_t; // map, attribute: 2; other: 1
constexpr auto resp3::to_code(type t) noexcept -> char
constexpr auto resp3::to_type(char c) noexcept -> type

struct resp3::basic_node<String> {
  type data_type = type::invalid;
  size_t aggregate_size{}, depth{};
  String value{}
};
auto operator==(basic_node<String> const& a; basic_node<String> const& b);
using node = basic_node<std::string>;
using node_view = basic_node<std::string_view>;

class resp3::parser {
public: using node_type = node_view; using result = std::optional<node_type>;
  static constexpr size_t max_embedded_depth = 5; static constexpr std::string_view sep = "\r\n";
private: using sizes_type = std::array<size_t, max_embedded_depth+1>;
  static constexpr size_type default_sizes = {{2,1,1,1,1,1}}; static constexpr auto default_bulk_length = -1uz;
  sizes_type sizes_; type bulk_; size_t depth_, bulk_length_, consumed_; // state data
  auto consume_impl(type t, std::string_view elem, system::error_code& ec) -> node_type;
  void commit_elem() noexcept { --sizes_[depth_]; while (sizes_[depth_]==0) --sizes_[--depth_]; }
  [[nodiscard]] auto bulk_expected() const noexcept -> bool { return bulk_ != type::invalid; }
public: ctor() {reset();}
  [[nodiscard]] auto done() const noexcept -> bool { return depth_ == 0 && bulk_ == invalid && consumed_ != 0; }
  auto get_consumed() const noexcept -> size_t;
  auto consume(std::string_view view, system::error_code& ec) noexcept -> result;
  void reset() { depth_ = 0; sizes_ = default_sizes; bulk_length_ = default_bulk_length; bulk_ = invalid; consumed_ = 0; }
  bool is_parsing() const noexcept;
};
bool resp3::parse <Adapter> (parser& p, std::string_view const& msg, Adapter& adapter, system::error_code& ec) {
  if (!p.is_parsing()) adapter.on_init();
  while (!p.done()) {
    auto const res = p.consume(msg, ec); if (ec) return true; if (!res) return false;
    adapter.on_node(res.value(), ec); if (ec) return true;
  }
  adapter.on_done(); return true;
}

void resp3::boost_redis_to_bulk(std::string& payload, std::string_view data);
void resp3::boost_redis_to_bulk<T>(std::string& payload, T n) requires integral<T> { boost_redis_to_bulk(payload, std::to_string(n)); }
struct resp3::add_bulk_impl<T>
{ static void add(std::string& payload, T const& from) { using namespace resp3; boost_redis_to_bulk(payload, from); } };
struct resp3::add_bulk_impl<std::tuple<Ts...>>
{ static void add(std::string& payload, std::tuple<Ts...> const& from) {
  std::apply([&](auto const&...vs){ using namespace resp3; (boost_redis_to_bulk(payload, vs), ...); }, t);
} };
struct resp3::add_bulk_impl<std::pair<U,V>>
{ static void add(std::string& payload, std::pair<U,V> const& from) {
  std::apply([&](auto const&...vs){ using namespace resp3; boost_redis_to_bulk(payload, from.first); boost_redis_to_bulk(payload, from.second); }, t);
} };
void resp3::add_header(std::string& payload, type t, size_t size);
void resp3::add_bulk<T>(std::string& payload, T const& data) { add_bulk_impl<T>::add(payload, data); }

struct resp3::bulk_counter<T> { static constexpr auto size = 1U; };
struct resp3::bulk_counter<std::pair<T,U>> { static constexpr auto size = 2U; };
void resp3::add_blob(std::string& payload, std::string_view blob);
void resp3::add_separator(std::string& payload);
void resp3::detail::deserialize <Adapter> (std::string_view const& data, Adapter adapter, system::error_code& ec) {
  adapter.on_init();
  parser parser{};
  while (!parser.done()) {
    auto const res = parser.consume(data, ec); if (ec) return;
    adapter.on_node(res.value(), ec); if (ec) return;
  }
  adapter.on_done();
}
void resp3::detail::deserialize<Adapter> (std::string_view const& data, Adapter adapter) {
  system::error_code ec; deserialize(data, adapter, ec); if (ec) BOOST_THROW_EXCEPTION(system::system_error{ec});
}

enum class error { invalid_data_type = 1,
  not_a_number, exceeds_max_nested_depth, unexpected_bool_value, empty_field,
  expects_resp3_simple_type, expects_resp3_aggregate, expects_resp3_map, expects_resp3_set,
  nested_aggregate_not_supported, resp3_simple_error, resp3_blob_error, incompatible_size,
  not_a_double, resp3_null, not_connected, resolve_timeout, pong_timeout, ssl_handshake_timeout,
  sync_receive_push_failed, incompatible_node_depth, resp3_hello, unix_sockets_unsupported,
  unix_sockets_ssl_unsupported, exceeds_maximum_read_buffer_size,
};
auto make_error_code(error e) -> system::error_code;
struct std::is_error_code_enum<error> : std::true_type {};
```

------
### Redis Request and Response

```c++
struct address { std::string host="127.0.0.1", port="6379"; };
struct config {
  bool use_ssl = false;
  address addr = address{};
  std::string unix_socket, username="default", password, clientname="Boost.Redis", health_check_id="Boost.Redis", log_prefix="(Boost.Redis)";
  std::optional<int> database_index ={0};
  std::chrono::steady_clock::duration resolve_timeout=10s, connect_timeout=10s, ssl_handshake_timeout=10s, health_check_interval=2s, reconnect_wait_interval=1s;
  size_t max_read_size=std::numeric_limits<size_t>::max(), read_buffer_append_size=4096;
  bool use_setup = false;
  request setup = make_hello_request();
};

auto detail::has_response(std::string_view cmd) -> bool;
request detail::make_hello_request();

class request {
  config cfg_; std::string payload_; size_t commands_{0}, expected_responses_{0}; bool has_hello_priority_=false;
  void check_cmd(std::string_view cmd) {
    ++commands_;
    if (!has_response(cmd)) ++expected_responses_;
    if (cmd=="HELLO") has_hello_priority_ = cfg_.hello_with_priority;
  }
public: struct config { bool cancel_on_connection_lost=true, cancel_if_not_connected=false, cancel_if_unresponded=true, hello_with_priority=true; };
  explicit ctor(config cfg=config{}) : cfg_{cfg} {}
  [[nodiscard]] auto get_expected_responses() const noexcept -> size_t;
  [[nodiscard]] auto get_commands() const noexcept -> size_t;
  [[nodiscard]] auto payload() const noexcept -> std::string_view;
  void clear() { payload_.clear(); commands_=0; expected_responses_=0; has_hello_priority_=false; }
  void reserve(size_t new_cap=0) { payload_.reserve(new_cap); }
  [[nodiscard]] auto get_config() <const> noexcept -> config <const>& { return cfg_; }
  void push<...Ts> (std::string_view cmd, Ts const&... args) {
    resp3::add_header(payload_, type::array, sizeof...(Ts) + 1);
    resp3::add_bulk(payload_, cmd);
    resp3::add_bulk(payload_, std::tie(args...));
    check_cmd(cmd);
  }
  void push_range<FwdIt> ( std::string_view cmd, <std::string_view key>, FwdIt begin, FwdIt end, std::iterator_traits<FwdIt>::value_type*=nullptr) {
    if (begin == end) return;
    auto constexpr size = resp3::bulk_counter<value_type>::size; auto const distance = std::distance(begin, end);
    resp3::add_header(payload_, type::array, size*distance + 2);
    resp3::add_bulk(payload_, cmd);
    <* resp3::add_bulk(payload_, key); *>
    for (; begin != end; ++begin) resp3::add_bulk(payload_, *begin);
    check_cmd(cmd);
  }
  void push_range<Range> ( std::string_view cmd, <std::string_view key>, Range const& range, decltype(std::begin(range))*=nullptr)
  { push_range(cmd, <key>, begin(range), end(range)); }
};

struct detail::request_access {
  static void set_priority(request& r, bool value) { r.has_hello_priority_ = value; }
  static bool has_priority(const request& r) { return r.has_hello_priority_; }
};

using response<...Ts> = std::tuple<adapter::result<Ts>...>;
using generic_response = adapter::result<std::vector<resp3::node>>;
void consume_one(generic_response& r, system::error_code& ec);
void consume_one(generic_response& r);
```

------
### Adapter

```c++
using ignore_t = std::decay_t<decltype(std::ignore)>;
ignore_t ignore;

struct adapter::error { resp3::type data_type = invalid; std::string diagnostic; };
bool operator==(error const& a, error const& b); // and !=
using result<Value> = system::result<Value, error>;
[[noreturn]] void throw_exception_from_error(error const& e, source_location const&) {
  system::error_code ec;
  switch (e.data_type) {
    case simple_error: ec = resp3_simple_error; break;
    case blob_error: ec = resp3_blob_error; break;
    case null: ec = resp3_null; break;
  }
  throw system::system_error(ec, e.diagnostic);
}

struct adapter::detail::is_integral_number<T> : std::is_integral<T> {};
struct adapter::detail::is_integral_number<{bool|char|char16_t|char32_t|wchar_t|char8_t}> : std::false_type {};
struct adapter::detail::converte<T,isIntNum=is_integral_number<T>::value>;
struct adapter::detail::converte<T,true> {
  static void apply<String>(T& i, resp3::basic_node<String> const& node, system::error_code& ec)
  { auto res = from_chars(node.value, i); if (res.ec) ec = not_a_number; }
};
struct adapter::detail::converte<{bool|double|std::basic_string<Ch,Tr,A>},false>
{ static void apply<String>({bool|double|std::basic_string<Ch,Tr,A>}& i, resp3::basic_node<String> const& node, system::error_code& ec); };

struct adapter::detail::from_bulk_impl<T>
{ static void apply<String>({bool|double|std::basic_string<Ch,Tr,A>}& i, resp3::basic_node<String> const& node, system::error_code& ec); };
struct adapter::detail::from_bulk_impl<std::optional<T>>
{ static void apply<String>(std::optional<T>& i, resp3::basic_node<String> const& node, system::error_code& ec); };
void adapter::detail::boost_redis_from_bulk<T,String>(T& t, resp3::basic_node<String> const& node, system::error_code& ec)
{ from_bulk_impl<T>::apply(t, node, ec); }

class adapter::detail::general_aggregate<Result> { Result* result_;
public: explicit ctor(Result* c=nullptr) : result_{c} {}
  void on_init() {} void on_done() {}
  void on_node<String>(resp3::basic_node<String> const&nd, system::error_code&) {
    switch (nd.data_type) {
      case blob_error: case simple_error: *result_=error{nd.data_type, std::string{cbegin(nd.value), cend(nd.value)}}; break;
      default: if (result_->has_value()) { (**result_).push_back({nd.data_type, nd.aggregate_size, nd.depth, std::string{cbegin(nd.value), cend(nd.value)}}); }
    }
  }
};
class adapter::detail::general_simple<Node> { Node* result_;
public: explicit ctor(Node* t=nullptr) : result_{t} {}
  void on_init() {} void on_done() {}
  void on_node<String>(resp3::basic_node<String> const&nd, system::error_code&) {
    switch (nd.data_type) {
      case blob_error: case simple_error: *result_=error{nd.data_type, std::string{cbegin(nd.value), cend(nd.value)}}; break;
      default: result_->value() = {nd.data_type, nd.aggregate_size, nd.depth, assign(nd.value)};
    }
  }
};
class adapter::detail::simple_impl<Result> {
public: void on_value_available(Result&) {}
  void on_init() {} void on_done() {}
  void on_node<String>(Result& result, resp3::basic_node<String> const&nd, system::error_code&) {
    if (is_aggregate(node.data_type)) { ec=expects_resp3_simple_type; return; }
    boost_redis_from_bulk(result, node, ec);
  }
};
class adapter::detail::set_impl<Result> { Result::iterator hint_;
public: void on_value_available(Result&) { hint_ = std::end(result); }
  void on_init() {} void on_done() {}
  void on_node<String>(Result& result, resp3::basic_node<String> const&nd, system::error_code&) {
    if (is_aggregate(node.data_type)) { if (nd.data_type!=set) ec=expects_resp3_set; ec=expects_resp3_simple_type; return; }
    if (nd.depth<1) { ec=expects_resp3_set; return; }
    Result::key_type obj; boost_redis_from_bulk(obj, nd, ec); hint_ = result.insert(hint_, std::move(obj));
  }
};
class adapter::detail::map_impl<Result> { Result::iterator current_; bool on_key_ = true;
public: void on_value_available(Result&) { current_ = std::end(result); }
  void on_init() {} void on_done() {}
  void on_node<String>(Result& result, resp3::basic_node<String> const&nd, system::error_code&) {
    if (is_aggregate(node.data_type)) { if (element_multiplicity(nd.data_type)!=2) ec=expects_resp3_set; ec=expects_resp3_map; return; }
    if (nd.depth<1) { ec=expects_resp3_map; return; }
    if (on_key_) { Result::key_type obj; boost_redis_from_bulk(obj, nd, ec); current_ = result.insert(current_, {std::move(obj), {}}); }
    else {Result::mapped_type obj; boost_redis_from_bulk(obj, nd, ec); current_->second = std::move(obj); }
    on_key_ = !on_key_;
  }
};
class adapter::detail::vector_impl<Result> {
public: void on_value_available(Result&) {}
  void on_init() {} void on_done() {}
  void on_node<String>(Result& result, resp3::basic_node<String> const&nd, system::error_code&) {
    if (is_aggregate(node.data_type)) { result.reserve(result.size() + element_multiplicity(nd.data_type)*nd.aggregate_size); }
    else { result.push_back({}); boost_redis_from_bulk(result.back(), nd, ec); }
  }
};
class adapter::detail::array_impl<Result> { int i_ = -1;
public: void on_value_available(Result&) {}
  void on_init() {} void on_done() {}
  void on_node<String>(Result& result, resp3::basic_node<String> const&nd, system::error_code&);
};
class adapter::detail::list_impl<Result> { int i_ = -1;
public: void on_value_available(Result&) {}
  void on_init() {} void on_done() {}
  void on_node<String>(Result& result, resp3::basic_node<String> const&nd, system::error_code&);
};

struct adapter::detail::impl_map<T> { using type = simple_impl<T>; };
struct adapter::detail::impl_map<std::[unordered]_[multi]set<K,...>> { using type = set_impl<std::[unordered]_[multi]set<K,...>>; }; // other assoc containers
struct adapter::detail::impl_map<std::[unordered]_[multi]set<K,T,...>> { using type = map_impl<std::[unordered]_[multi]map<K,T,...>>; }; // other assoc containers
struct adapter::detail::impl_map<std::vector<T,A>> { using type = vector_impl<std::vector<T,A>>; };
struct adapter::detail::impl_map<std::array<T,N>> { using type = array_impl<std::array<T,N>>; };
struct adapter::detail::impl_map<std::{list|deque}<T,A>> { using type = list_impl<std::{list|deque}<T,A>>; };

class adapter::detail::wrapper<T>;
class adapter::detail::wrapper<result<T>> { response_type* result_; impl_map<T>::type impl_; bool called_once_ = false;
  bool set_if_resp3_error<String>(resp3::basic_node<String> const& nd) noexcept
  { switch (nd.data_type) {
    case null: case simple_error: case blob_error: *result_ = error{nd.data_type, {cbegin(nd.value), cend(nd.value)}}; return true;
    default: return false;
  }}
public: using response_type = result<T>;
  explicit ctor(response_type* t=nullptr) : result_{t} { if (result_) { result_->value() = T{}; impl_.on_value_available(result_->value()); }}
  void on_init() { impl_.on_init(); } void on_done() { impl_.on_done(); }
  void on_node<String>(resp3::basic_node<String> const& nd, system::error_code& ec) {
    if (result_->has_error()) return;
    if (!exchange(called_once_,true) && set_if_resp3_error(nd)) return;
    impl_.on_node(result_->value(), nd, ec);
  }
};
class adapter::detail::wrapper<result<std::optional<T>>> { response_type* result_; impl_map<T>::type impl_{}; bool called_once_ = false;
  bool set_if_resp3_error<String>(resp3::basic_node<String> const& nd) noexcept
  { switch (nd.data_type) {
    case simple_error: case blob_error: *result_ = error{nd.data_type, {cbegin(nd.value), cend(nd.value)}}; return true;
    default: return false;
  }}
public: using response_type = result<std::optional<T>>;
  explicit ctor(response_type* t=nullptr) : result_{t} {}
  void on_init() { impl_.on_init(); } void on_done() { impl_.on_done(); }
  void on_node<String>(resp3::basic_node<String> const& nd, system::error_code& ec) {
    if (result_->has_error()) return;
    if (set_if_resp3_error(nd)) return;
    if (!exchange(called_once_,true) && nd.data_type == null) return;
    if (!result_->value().has_value()) { result_->value() = T{}; impl_.on_value_available(result_->value().value()); }
    impl_.on_node(result_->value(), nd, ec);
  }
};

struct adapter::ignore {
  void on_init() { } void on_done() { }
  void on_node<String>(resp3::basic_node<String> const& nd, system::error_code& ec) {
    switch(nd.data_type) {
      case simple_error: ec = resp3_simple_error; break;
      case blob_error: ec = resp3_blob_error; break;
      case null: ec = resp3_null; break;
    }
  }
};

struct adapter::detail::result_traits;
struct adapter::detail::result_traits<ignore_t> {
};
struct adapter::detail::result_traits<Result> {
  using adapter_type = wrapper<std::decay_t<Result>>;
  static auto adapt(Result& r) noexcept { return adapter_type{&r}; }
};
struct adapter::detail::result_traits<result<ignore_t>> {
  using response_type = result<ignore_t>; using adapter_type = ignore;
  static auto adapt(response_type) noexcept { return adapter_type{}; }
};
struct adapter::detail::result_traits<ignore_t> {
  using response_type = ignore_t; using adapter_type = ignore;
  static auto adapt(response_type) noexcept { return adapter_type{}; }
};
struct adapter::detail::result_traits<result<resp3::basic_node<T>>> {
  using response_type = result<resp3::basic_node<T>>; using adapter_type = general_simple<response_type>;
  static auto adapt(response_type& v) noexcept { return adapter_type{&v}; }
};
struct adapter::detail::result_traits<result<std::vector<resp3::basic_node<String>,Alloc>>> {
  using response_type = result<std::vector<resp3::basic_node<String>,Alloc>>; using adapter_type = general_aggregate<response_type>;
  static auto adapt(response_type& v) noexcept { return adapter_type{&v}; }
};

using adapter::detail::adapter_t<T> = result_traits<std::decay_t<T>>::adapter_type;
auto adapter::detail::internal_adapt <T>(T& t) noexcept { return result_traits<std::decay_t<T>>::adapt(t); }
struct adapter::detail::assigner<n>{ static void assign<T1,T2>(T1& dest, T2& from) {
  dest[n].emplace<n>(internal_adapt(std::get<n>(from)));
  if constexpr (n>0) assigner<n-1>::assign(dest, from);
} }

class adapter::detail::static_aggregate_adapter<Tuple>;
class adapter::detail::static_aggregate_adapter<result<Tuple>> {
  using adapters_array_type = std::array<mp_rename<mp_transform<adapter_t,Tuple>, std::varaint>, std::tuple_size_v<Tuple>>;
  size_t i_ = 0, aggregate_size_ = 0;
  adapters_array_type adapters_; result<Tuple>* res_ = nullptr;
public: explicit ctor(result<Tuple>* r=nullptr) { if (r) { res_=r; assigner<tuple_size_v<Tuple>-1>::assign(adapters_,r->value()); } }
  void count<String>(resp3::basic_node<String> const& elem) {
    if (elem.depth == 1 && is_aggregate(elem.data_type)) aggregate_size_=element_multiplicity(elem.data_type) * elem.aggregate_size;
    if (aggregate_size_ == 0) i_ += 1 else aggregate_size_ -= 1;
  }
  void on_init() { for (auto& adaper: adapters_) visit([&](auto& arg) {arg.on_init();}, adapter); }
  void on_done() { for (auto& adaper: adapters_) visit([&](auto& arg) {arg.on_done();}, adapter); }
  void on_node<String>(resp3::basic_node<String> const& elem, system::error_code& ec) {
    if (elem.depth == 0) {
      if (elem.aggregate_size * element_multiplicity(elem.data_type) != std::tuple_size_v<Tuple>) ec = incompatible_size;
      return;
    }
    visit([&](auto& arg) {arg.on_node(elem, ec);}, adapters_.at(i_));
    count(elem);
  }
};

struct adapter::detail::result_traits<result<std::tuple<Ts...>>> {
  using response_type = result<std::tuple<Ts...>>>; using adapter_type = static_aggregate_adapter<response_type>;
  static auto adapt(response_type& r) noexcept { return adapter_type{&r}; }
};

class adapter::detail::static_adapter<Response> { static constexpr auto size = std::tuple_size_v<Response>;
  using adapters_array_type = std::array<mp_rename<mp_transform<adapter_t,Response>>, size>;
  adapters_array_type adapters_; size_t i_ = 0;
public: explicit ctor(Response& r) { assigner<size-1>::assign(adapters_, r); }
  void on_init() { visit([&](auto& arg) {arg.on_init();}, adapters_.at(i_)); }
  void on_done() { visit([&](auto& arg) {arg.on_done();}, adapters_.at(i_)); i_ += 1; }
  void on_node<String>(resp3::basic_node<String> const& nd, system::error_code& ec) {
    visit([&](auto& arg) {arg.on_node(nd, ec);}, adapters_.at(i_));
  }
};

struct adapter::detail::result_traits<response<Ts...>> {
  using response_type = response<Ts...>>; using adapter_type = static_adapter<response_type>;
  static auto adapt(response_type& r) noexcept { return adapter_type{r}; }
};

auto adapter::boost_redis_adapt(T& t) noexcept { return response_traits<T>::adapt(t); }
auto adapter::adapt2<T> (T& t = ignore) noexcept { return result_traits<T>::adapt(t); }

class any_adapter { impl_t impl_;
public: enum class parse_event { init, node, done };
  using impl_t = std::function<void(parse_event, resp3::node_view const&, system::error_code&)>;
  static auto create_impl<T> (T& resp) -> impl_t {
    return [adapter2=boost_redis_adapt(resp)](parse_event ev, resp3::node_view const& nd, system::error_code& ec) mutable
    { switch (ev) {
      case init: adapter2.on_init(); break; case node: adapter2.on_node(nd,ec); break; case done: adapter2.on_done(); break;
    }}
  }
  ctor(impl_t fn=[](auto,auto,auto){}) : impl_{std::move(fn)} {}
  explicit ctor<T> (T& resp) requires !same_as<T,self> : impl_(create_impl(resp)) {}
  void on_init() { system::error_code ec; impl_(init, {}, ec); }
  void on_done() { system::error_code ec; impl_(done, {}, ec); }
  void on_node(resp3::node_view const&nd, system::error_code& ec) { impl_(node, nd, ec); }
};
```

------
### Redis Connection

```c++
enum class operation { resolve, connect, ssl_handshake, exec, run, receive, reconnection, health_check, all };

struct usage {
  size_t commands_sent=0, bytes_sent=0, responses_received=0, pushes_received=0, response_bytes_received=0, push_bytes_received=0;
};

struct logger {
  enum class level { disabled, emerg, alert, crit, err, warning, notice, info, debug };
  level lvl; std::function<void(level, std::string_view)> fn;

  ctor(level l=info);
  ctor(level l, std::function<void(level, std::string_view)> fun) : lvl{l}, fn{std::move(fn)} {}
};

class detail::connection_logger { logger logger_; std::string msg_;
public: ctor(logger&& logger) noexcept : logger_(std::move(logger)) {}
  void reset(logger&& logger) { logger_ = std::move(logger); }

  void on_resolve(std::error_code const& ec, asio::ip::tcp::resolver::results_type const& res);
  void on_connect(std::error_code const& ec, asio::ip::tcp::endpoint const& ep);
  void on_connect(std::error_code const& ec, std::string_view unix_socket_ep);
  void on_ssl_handshake(std::error_code const& ec);
  void on_write(std::error_code const& ec, size_t n);
  void on_fsm_resume(reader_fsm::action const& action);
  void on_setup(std::error_code const& ec, generic_response const& resp);

  void log(logger::level lvl, std::string_view msg);
  void log(logger::level lvl, std::string_view op, system::error_code const& ec);
  void trace(std::string_view msg) { log(logger::debug, msg); }
  void trace(std::string_view op, system::error_code const& ec) { log(logger::debug, op, ec); }
};

class detail::read_buffer {
  config cfg_{}; std::vector<char> buffer_; size_t append_buf_begin_=0;
public: using span_type = span<char>;
  struct config { size_t read_buffer_append_size=4096u, max_read_size=-1uz; };
  [[nodiscard]] auto prepare_append() -> system::error_code;
  [[nodiscard]] auto get_append_buffer() noexcept -> span_type;
  void commit_append(size_t read_size);
  [[nodiscard]] auto get_committed_buffer() const noexcept -> std::string_view;
  [[nodiscard]] auto get_committed_size() const noexcept -> size_t;
  void clear();
  auto consume_comitted(size_t size) -> size_t;
  void reserve(size_t n);
  friend bool operator{==|!=}(self const& lhs, self const& rhs);
  void set_config(config const& cfg) noexcept { cfg_ = cfg; }
};

enum class detail::consume_result { needs_more, got_response, got_push };
class detail::multiplexer {
  std::string writer_buffer_; std::deque<std::shared_ptr<elem>> reqs_; resp3::parser parser_{};
  bool on_push_=false, cancel_run_called_=false; usage usage_; any_adapter set_receive_adapter_;

  void commit_usage(bool is_push, size_t size);
  [[nodiscard]] auto is_next_push(std::string_view data) const noexcept -> bool;
  [[nodiscard]] auto release_push_requests() -> size_t;
  [[nodiscard]] consume_result consume_next_impl(std::string_view data, system::error_code& ec);

public: class elem {
    enum class status { waiting, staged, written, done };
    request const* req_; any_adapter adapter_; std::function<void()> done_;
    size_t remaining_responses_; status status_; system::error_code ec_; size_t read_size_;
  public: explicit ctor(request const& req, any_adapter adapter);
    void set_done_callback(std::function<void()> f) noexcept { done_ = std::move(f); }
    void notify_done() noexcept { status_ = done; done_(); }
    void notify_error(system::error_code ec) noexcept;
    [[nodiscard]] auto is_{waiting|written|staged|done}() const noexcept { return status_=={waiting|written|staged|done}; }
    void mark_{written|staged|waiting}() noexcept { status_ = {written|staged|waiting}; }
    auto get_error() const -> system::error_code const& { return ec_; }
    auto get_request() const -> request const& { return *req_; }
    auto get_read_size() const -> size_t { return read_size_; }
    auto get_remaining_responses() const -> size_t { return remaining_responses_; }
    void commit_response(size_t read_size);
    auto get_adapter() -> any_adapter& { return adaper_; }
  };

  [[nodiscard]] auto prepare_write() -> size_t;
  auto commit_write() -> size_t;
  [[nodiscard]] auto consume_next(std::string_view data, system::error_code& ec) -> std::pair<consume_result, size_t>;
  void add(std::shared_ptr<elem> const& ptr);
  bool remove(std::shared_ptr<elem> const& ptr);
  void reset();
  [[nodiscard]] auto const& get_parser() const noexcept { return parser_; }
  auto cancel_waiting() -> size_t;
  void cancel_on_conn_lost();
  [[nodiscard]] auto get_write_buffer() noexcept -> std::string_view { return {write_buffer_}; }
  void set_receive_adapter(any_adapter adapter);
  [[nodiscard]] auto get_usage() const noexcept -> usage { return usage_; }
  [[nodiscard]] auto is_writing() const noexcept -> bool;
};
auto make_elem(request const& req, any_adapter adapter) -> std::shared_ptr<multiplexer::elem>;

auto detail::is_cancelled<T>(T const& self) { return self.get_cancellation_state().cancelled() != none; }

enum class detail::exec_action_type { setup_cancellation, immediate, done, notify_writer, wait_for_response, cancel_run };
class detail::exec_action {
  exec_action_type type_; system::error_code ec_; size_t bytes_read_;
public: ctor(system::error_code ec, size_t bytes_read=0u) noexcept : type_{done}, ec_{ec}, bytes_read_{bytes_read} {}
  exec_action_type type() const { return type_; }
  system::error_code error() const { return ec_; }
  size_t bytes_read() const { return bytes_read_; }
};
class detail::exec_fsm {
  int resume_point_{0}; multiplexer* mpx_{nullptr}; std::shared_ptr<multiplexer::elem> elem_;
public: ctor(multiplexer& mpx, std::shared_ptr<multiplexer::elem> elem) noexcept : mpx_{&mpx}, elem_{std::move(elem)} {}
  exec_action resume(bool connection_is_open, asio::cancellation_type_t cancel_state);
};

class detail::reader_fsm {
  int resume_point_{0}; read_buffer* read_buffer_=nullptr;
  action action_after_resume_; action::type next_read_type_ = append_some;
  multiplexer* mpx_=nullptr; std::pair<consume_result,size_t> res{needs_more, 0u};
public: struct action {
    enum class type { setup_cancellation, append_some, needs_more, notify_push_receiver, cancel_run, done };
    type type_ = setup_cancellation; size_t push_size_t=0u; system::error_code ec_{};
  };
  explicit ctor(read_buffer& rbuf, multiplexer& mpx) noexcept;
  action resume(size_t bytes_read, system::error_code ec, asio::cancellation_type_t);
};

auto detail::write<SyncWriteStream>(SyncWriteStream& stream, request const& req) { return asio::write(stream, asio::buffer(req.payload()), ec); }
auto detail::async_write<AsyncWriteStream,CompletionToken=asio::default_completion_token_t<AsyncWriteStream::executor_type>>
  (AsyncWriteStream& stream, request const& req, CompletionToken&& token={}) { return asio::async_write(stream, asio::buffer(req.payload()), token); }

void detail::compose_setup_request(config& cfg);
void detail::clear_response(generic_response& res);
system::error_code detail::check_setup_response(system::error_code io_ec, const generic_response&);

struct detail::ping_op<HealthChecker,ConnectionImpl> {
  HealthChecker* checker_=nullptr; ConnectionImpl* conn_=nullptr; asio::coroutine coro_{};
  void operator() <Self>(Self& self, system::error_code ec={}, size_t=0);
};
struct detail::check_timeout_op<HealthChecker,Connection> {
  HealthChecker* checker_=nullptr; Connection* conn_=nullptr; asio::coroutine coro_{};
  void operator() <Self>(Self& self, system::error_code ec={});
};
class health_checker<Executor> {
  using timer_type = asio::basic_waitable_timer<std::chrono::steady_clock, asio::wait_traits<steady_clock>, Executor>;
  timer_type ping_timer_, wait_timer_; request req_; generic_response resp_;
  std::chrono::steady_clock::duration ping_interval_ = 5s; bool checker_has_exited_=false;
public: ctor(Executor ex) : ping_timer_{ex}, wait_timer_{ex} { req_.push("PING", "Boost.Redis"); }
  void set_config(config const& cfg) { req_.clear(); req_.push("PING", cfg.health_check_id); ping_interval_=cfg.health_check_interval; }
  void cancel() { ping_timer_.cancel(); wait_timer_.cancel(); }
  auto async_ping<ConnectionImpl,CompletionToken>(ConnectionImpl& conn, CompletionToken token)
  { return asio::async_compose<CompletionToken, void(system::error_code)>
    (ping_op<health_checker, ConnectionImpl>{this, &conn}, token, conn, ping_timer_); }
  auto async_check_timeout<Connection,CompletionToken>(Connection& conn, CompletionToken token)
  { checker_has_exited_=false;
    return asio::async_compose<CompletionToken, void(system::error_code)>
    (check_timeout_op<health_checker, Connection>{this, &conn}, token, conn, wait_timer_); }
};

struct detail::connection_impl<Executor> {
  using clock_type = std::chrono::steady_clock; using clock_traits_type = asio::wait_traits<clock_type>;
  using timer_type = asio::basic_waitable_timer<clock_type, clock_traits_type, Executor>;
  using receive_channel_type = asio::experimental::channel<Executor, void(system::error_code,size_t)>;
  using health_checker_type = health_checker<Executor>;
  using exec_notifier_type = asio::experimental::channel<Executor, void(system::error_code,size_t)>;
  using executor_type = Executor;

  redis_stream<Executor> stream_;
  timer_type writer_timer_, reconnect_timer_; receive_channel_type receive_channel_; health_checker_type health_checker_;
  config cfg_; multiplexer mpx_; connection_logger logger_; read_buffer read_buffer_; generic_response setup_resp_;

  executor_type get_executor() noexcept { return writer_timer_.get_executor(); }

  struct exec_op {
    connection_impl* obj_=nullptr; std::shared_ptr<exec_notifier_type> notifier_=nullptr; exec_fsm fsm_;
    void operator() <Self>(Self& self, system::error_code={}, size_t=0);
  };

  ctor(Executor&& ex, asio::ssl::context&& ctx, logger&& lgr);
  void cancel(operation op);
  void cancel_run();
  bool is_open() const noexcept { return stream_.is_open(); }
  bool will_reconnect() const noexcept { return cfg_.reconnect_wait_interval != 0s; }
  auto async_exec<CompletionToken> (request const& req, any_adapter adapter, CompletionToken&& token);
};

struct detail::writer_op<Executor> {
  connection_impl<Executor>* conn_; asio::coroutine coro{};
  void operator() <Self>(Self& self, system::error_code ec={}, size_t n=0);
};
struct detail::reader_op<Executor> {
  connection_impl<Executor>* conn_; reader_fsm fsm_;
  ctor(connection_impl<Executor>& conn) noexcept : conn_{&conn}, fsm_{conn.read_buffer_, conn.mpx_} {}
  void operator() <Self>(Self& self, system::error_code ec={}, size_t n=0);
};
system::error_code detail::check_config(const config& cfg);
class detail::run_op<Executor> {
  connection_impl<Executor>* conn_; asio::coroutine coro_{}; system::error_code stored_ec_;
  using order_t = std::array<size_t, 5>;
  static system::error_code on_setup_finished(connection_impl<Executor>& conn, system::error_code ec);
  auto send_setup<CompletionToken>(CompletionToken&& token);
  auto reader<CompletionToken>(CompletionToken&& token);
  auto writer<CompletionToken>(CompletionToken&& token);
public: ctor(connection_impl<Executor>* conn) noexcept : conn_{conn} {}
  void operator()<Self>(Self& self, order_t order, error_code ec0, ... ec5);
  void operator()<Self>(Self& self, system::error_code ec={});
};

logger detail::make_stderr_logger(logger::level lvl, std::string prefix);

class basic_connection<Executor> {
  using clock_type = std::chrono::steady_clock; using clock_traits_type = asio::wait_traits<clock_type>;
  using timer_type = asio::basic_waitable_timer<clock_type, clock_traits_type, executor_type>;
  using receive_channel_type = asio::experimental::channel<executor_type, void(system::error_code,size_t)>;
  using health_checker_type = health_checker<executor_type>;
  std::unique_ptr<connection_impl<Executor>> impl_; // PImpl
  bool use_ssl() const noexcept { return impl_->cfg_.use_ssl; }
  void set_stderr_logger(lgoger::level lvl, const config& cfg) { impl_->logger_.reset(make_stderr_logger(lvl, cfg.log_prefix)); }
public: using this_type = basic_connection<Executor>; using executor_type = Executor;
  struct rebind_executor<Executor1> { using other=basic_connection<Executor1>; };

  explicit ctor(executor_type ex, asio::ssl::context ctx={tlsv12_client}, logger lgr={})
    : impl_(std::make_unique<connection_impl<Executor>>(std::move(ex), std::move(ctx), std::move(lgr))) {}
  ctor(executor_type ex, logger lgr) : self(std::move(ex), asio::ssl::context{tlsv12_client}, std::move(lgr)) {}
  explicit ctor(asio::io_context& ioc, asio::ssl::context ctx={tlsv12_client}, logger lgr={}) : self(io.get_executor(), std::move(ctx), std::move(lgr)) {}
  ctor(asio::io_context& ioc, logger lgr) : self(ioc.get_executor(), asio::ssl::context{tlsv12_client}, std::move(lgr)) {}

  executor_type get_executor() noexcept { return impl_->writer_timer_.get_executor(); }

  auto async_run<CompletionToken=asio::default_completion_token_t<executor_type>>(config const& cfg, CompletionToken&& token={}) {
    impl_->cfg_ = cfg; impl_->health_checker_.set_config(cfg); impl_->read_buffer_.set_config({cfg.read_buffer_append_size, cfg.max_read_size});
    return asio::async_compose<CompletionToken, void(system::error_code)>(run_op<Executor>{impl_.get()}, token, impl_->writer_timer_);
  }
  auto async_receive<CompletionToken=asio::default_completion_token_t<executor_type>>(CompletionToken&& token={})
  { return impl_->receive_channel_.async_receive(std::forward<CompletionToken>(token)); }
  size_t receive(system::error_code& ec) { size_t size=0;
    auto f=[&](system::error_code const& ec2, size_t n) { ec=ec2; size=n; };
    auto const res = impl_->receive_channel_.try_receive(f);
    if (ec) return 0; if (!res) ec = sync_receive_push_failed; return size;
  }
  auto async_exec<Response=ignore_t,CompletionToken=asio::default_completion_token_t<executor_type>>(request const& req, Response& resp=ignore, CompletionToken&& token={})
  { return this->async_exec(req, any_adapter{resp}, std::forward<CompletionToken>(token)); }
  auto async_exec<CompletionToken=asio::default_completion_token_t<executor_type>>(request const& req, any_adapter adapter, CompletionToken&& token={})
  { return this->async_exec(req, std::move(adaper), std::forward<CompletionToken>(token)); }
  void cancel(operation op=all) { impl_->cancel(op); }
  bool will_reconnect() const noexcept { return impl_->will_reconnect(); }
  void set_receive_response<Response>(Response&resp) { impl_->set_receive_adapter(any_adapter{resp}); }
  usage get_usage() const noexcept { return impl_->mpx_.get_usage(); }
};

class connection {
  basic_connection<executor_type> impl_;
  void async_run_impl(config const& cfg, logger&& l, asio::any_completion_handler<void(system::error_code)> token);
  void async_run_impl(config const& cfg, asio::any_completion_handler<void(system::error_code)> token);
  void async_run_impl(request const& req, any_adapter&& adapter, asio::any_completion_handler<void(system::error_code)> token);
public: using executor_type = asio::any_io_executor;
  explicit ctor(executor_type ex, asio::ssl::context ctx={tlsv12_client}, logger lgr={});
  ctor(executor_type ex, logger lgr) : self(std::move(ex), asio::ssl::context{tlsv12_client}, std::move(lgr)) {}
  explicit ctor(asio::io_context& ioc, asio::ssl::context ctx={tlsv12_client}, logger lgr={}) :self{ioc.get_executor(), std::move(ctx), std::move(lgr)} {}
  ctor(asio::io_context& loc, logger lgr) : self(ioc.get_executor(), asio::ssl::context{tlsv12_client}, std::move(lgr)) {}

  executor_type get_executor() noexcept { return impl_.get_executor(); }

  auto async_run<CompletionToken=asio::deferred_t>(config const& cfg, CompletionToken&& token={}) {
    return asio::async_initiate<CompletionToken, void(system::error_code)>(
      [](auto handler, connection* self, config const* cfg){ self->async_run_impl(*cfg, std::move(handler)); }, token, this, &cfg);
  }
  auto async_receive<CompletionToken=asio::deferred_t>(CompletionToken&& token={})
  { return impl_.async_receive(std::forward<CompletionToken>(token)); }
  size_t receive(system::error_code& ec) { return impl_.receive(ec); }
  auto async_exec<Response=ignore_t,CompletionToken=asio::deferred_t>(request const& req, Response& resp=ignore, CompletionToken&& token={})
  { return this->async_exec(req, any_adapter{resp}, std::forward<CompletionToken>(token)); }
  auto async_exec<CompletionToken=asio::deferred_t>(request const& req, any_adapter adapter, CompletionToken&& token={})
  { return asio::async_initiate<CompletionToken, void(system::error_code, size_t)>(
      [](auto handler, connection* self, request const* req, any_adapter&& adapter) { self->async_exec_impl(*req, std::move(adapter), std::move(handler));},
      token, this, &req, std::move(adapter)
  ); }
  void cancel(operation op=all);
  bool will_reconnect() const noexcept { return impl_.will_reconnect(); }
  void set_receive_response<Response>(Response& response) { impl_.set_receive_response(response); }
  usage get_usage() const noexcept { return impl_.get_usage(); }
};
```

------
### Dependency

#### Boost.Asio

* `<boost/asio/**.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Core

* `<boost/core/ignore_unused.hpp>`
* `<boost/core/make_span.hpp>`
* `<boost/core/span.hpp>`

#### Boost.MP11

* `<boost/mp11.hpp>`

#### Boost.System

* `<boost/system/error_code.hpp>`
* `<boost/system/result.hpp>`
* `<boost/system/system_error.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

### CMake Module Targets

* `Threads::Threads`
* `OpenSSL::Crypto`
* `OpenSSL::SSL`

------
### Standard Facilities

* Proposals:
  * N4361 - Concept Lite TS
