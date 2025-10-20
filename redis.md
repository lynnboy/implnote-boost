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
