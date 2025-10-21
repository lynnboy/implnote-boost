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
### Redis Connection

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
