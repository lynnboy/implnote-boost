# Boost.MQTT5

* lib: `boost/libs/mqtt5`
* repo: `boostorg/mqtt5`
* commit: `150ba94`, 2025-09-19

------
### Common Parts

##### Errors

```c++
enum class disconnect_rc_e : uint8_t { normal_disconnect=0, disconnect_with_will_message=4 };
enum class detail::disconnect_rc_e : uint_8_t { normal_disconnect=0, disconnect_with_will_message=4, unspecified_error=0x80, malformed_packet, protocol_error, implementation_specific_error,
    topic_name_invalid=0x90, receive_maximum_exceeded=0x93, topic_alias_invalid, packet_too_large, message_rate_too_high, quota_exceeded, administrative_action, payload_format_invalid };
enum class client::error : int { malformed_packet = 100, packet_too_large, session_expired, pid_overrun, invalid_topic, qos_not_supported,
    retain_not_available, topic_alias_maximum_reached, wildcard_subscription_not_available, subscription_identifier_not_available, shared_subscription_not_available };
std::string client::client_error_to_string(error err);
struct client::client_ec_category : system::error_category {
    const char* name() const noexcept override { return "mqtt_client_error"; }
    std::string message(int ev) const noexcept override { return client_error_to_string((error)ev); }
};
const client_ec_category& client::get_error_code_category() { static client_ec_category cat; return cat; }
system::error_code client::make_error_code(error r) { return {(int)r, get_error_code_category()}; }
std::ostream& client::operator<<(std::ostream& os, const error& err) { return os << get_error_code_category().name() << ":" << (int)err; }
struct system::is_error_code_enum<client::error> : std::true_type {};
```

##### Reason Codes

```c++
enum class reason_codes::category : uint8_t { none, connack, puback, pubrec, pubrel, pubcomp, suback, unsuback, auth, disconnect };
class reason_code {
    uint8_t _code={0xff}; reason_codes::category _category={none};
public: constexpr ctor(); contexpr ctor(uint8_t code, category cat); const explicit ctor(uint8_t code);
    explicit operator bool() const noexcept { return _code >= 0x80; }
    constexpr uint8_t value() const noexcept { return _code; }
    friend std::ostream& operator<<(std::ostream& os, const self& rc) { return os << rc.message(); }
    friend bool operator< (const self& lhs, const self& rhs) { return lhs._code < rhs._code; }
    friend bool operator== (const self& lhs, const self& rhs) { return lhs._code == rhs._code && lhs._category == rhs._category; }
    std::string message() const;
};
constexpr reason_code reason_codes::empty{},/*0xff, invalid*/
    success{0x00}, normal_disconnection{0x00, disconnect}, granted_qos_0{0x00,suback}, granted_qos_1{0x01}, granted_qos_2{0x02},
    disconnect_with_will_message{0x04}, no_matching_subscribers{0x10}, no_subscription_existed{0x11}, continue_authentication{0x18}, reauthenticate{0x19},
    unspecified_error{0x80}, malformed_packet{0x81}, protocol_error{0x82}, implementation_specific_error{0x83}, unsupported_protocol_version{0x84},
    client_identifier_not_valid{0x85}, bad_username_or_password{0x86}, not_authorized{0x87}, server_unavailable{0x88}, server_busy{0x89}, banned{0x8a},
    server_shutting_down{0x8b}, bad_authentication_method{0x8c}, keep_alive_timeout{0x8d}, session_taken_over{0x8e}, topic_filter_invalid{0x8f},
    topic_name_invalid{0x90}, packet_identifier_in_use{0x91}, packet_identifier_not_found{0x92}, receive_maximum_exceeded{0x93}, topic_alias_invalid{0x94},
    packet_too_large{0x95}, message_rate_too_high{0x96}, quota_exceeded{0x97}, administrative_action{0x98}, payload_format_invalid{0x99},
    retain_not_supported{0x9a}, qos_not_supported{0x9b}, use_another_server{0x9c}, server_moved{0x9d}, shared_subscription_not_supported{0x9e},
    connection_rate_exceeded{0x9f}, maximum_connect_time{0xa0}, subscription_ids_not_supported{0xa1}, wildcard_subscription_not_supported{0xa2};
std::pair<reason_code*, size_t> reason_codes::detail::valid_codes<category cat>() { constexpr switch(cat)
    case connack: static reason_code valid_codes[] = {
        success, unspecified_error, malformed_packet, protocol_error, implementation_specific_error, unsupported_protocol_version, client_identifier_not_valid,
        bad_username_or_password, not_authorized, server_unavailable, server_busy, banned, bad_authentication_method, topic_name_invalid, packet_too_large, quota_exceeded,
        payload_format_invalid, retain_not_supported, qos_not_supported, use_another_server, server_moved, connection_rate_exceeded };
    case auth: static reason_code valid_codes[] = { success, continue_authentication, reauthenticate };
    case puback: case pubrec: static reason_code valid_codes[] = {
        success, no_matching_subscribers, unspecified_error, implementation_specific_error, not_authorized,
        topic_name_invalid, packet_identifier_in_use, quota_exceeded, payload_format_invalid };
    case pubrel: case pubcomp: static reason_code valid_codes[] = { success, packet_identifier_not_found };
    case suback: static reason_code valid_codes[] = {
        granted_qos_0, granted_qos_1, granted_qos_2, unspecified_error, implementation_specific_error, not_authorized, topic_filter_invalid,
        packet_identifier_in_use, quota_exceeded, shared_subscription_not_supported, subscription_ids_not_supported, wildcard_subscription_not_supported };
    case unsuback: static reason_code valid_codes[] = {
        success, no_subscription_existed, unspecified_error, implementation_specific_error, not_authorized, topic_filter_invalid, packet_identifier_in_use };
    case disconnect: static reason_code valid_codes[] = {
        normal_disconnection, unspecified_error, malformed_packet, protocol_error, implementation_specific_error, not_authorized, server_busy, server_shutting_down,
        keep_alive_timeout, session_taken_over, topic_filter_invalid, topic_name_invalid, receive_maximum_exceeded, topic_alias_invalid, packet_too_large,
        message_rate_too_high, quota_exceeded, administrative_action, payload_format_invalid, retain_not_supported, qos_not_supported, use_another_server, server_moved,
        shared_subscription_not_supported, connection_rate_exceeded, maximum_connect_time, subscription_ids_not_supported, wildcard_subscription_not_supported };
    return std::make_pair(valid_codes, sizeof(valid_codes)/sizeof(reason_code));
}
std::optional<reason_code> to_reason_code<reason_codes::category cat>(uint8_t code); // find in valid_codes<cat>(); otherwise nullopt
```

##### Property Types

```c++
namespace prop;
enum property_type : uint8_t { payload_format_indicator_t=1 /*u8*/, message_expiry_interval_t /*u32*/, content_type_t /*str*/, response_topic_t=0x8 /*str*/, correlation_data_t /*str*/,
    subscription_identifier_t=0xb, session_expiry_interval_t=0x11 /*u32*/, assigned_client_identifier_t /*str*/, server_keep_alive_t /*u16*/,
    authentication_method_t /*str*/, authentication_data_t /*str*/, request_problem_information_t /*u8*/, will_delay_interval_t /*u32*/, request_response_information_t /*u8*/, response_information_t /*str*/,
    server_reference_t=0x1c /*str*/, reason_string_t=0x1f /*str*/, receive_maximum_t=0x21 /*u16*/, topic_alias_maximum_t /*u16*/, topic_alias_t /*u16*/, maximum_qos_t /*u8*/, retain_available_t /*u8*/,
    user_property_t, maximum_packet_size_t /*u32*/, wildcard_subscription_available_t /*u8*/, subscription_identifier_available_t /*u8*/, shared_subscription_available_t /*u8*/ };

class alignas(8) subscription_identifiers : public container::small_vector<int32_t, 1> {
public: using base::ctor; ctor(int32_t v);
    bool has_value() const noexcept { return !empty(); } explicit operator bool() const noexcept { return !empty(); }
    int32_t& operator*() noexcept { return front(); } int32_t operator*() const noexcept { return front(); }
    void emplace(int32_t val=0) { *this = val; }
    int32_t value() const { return front(); } int32_t value_or(int32_t def) const noexcept { return empty() ? def : front(); }
    void reset() noexcept { clear(); }
};
using user_property_value_t = std::vector<std::pair<std::string, std::string>>;

struct property_traits<property_type p> { // specialize for each property_type enumerators
    static constexpr std::string_view name = <NAME>; // without `_t`
    using type = <TYPE>; // subscription_identifier is subscription_identifiers, user_property is user_property_value_t,
        // others are std::optional<> of uint8/uint16/uint32/std::string
};
using value_type_t<property_type p> = property_traits<p>::type;
const std::string_view name_v<property_type p> = property_traits<p>::name;

class properties<property_type ...Ps> {
    struct property<property_type p> {
        using key = std::integral_constant<property_type, p>;
        constexpr static std::string_view name = name_v<p>;
        value_type_t<p> value;
    };
    std::tuple<property<Ps>...> _props;
public:
    constexpr <const> auto& operator[]<property_type v>(std::integral_constant<property_type, v>) <const> noexcept { return std::get<property<v>>(_props).value; }
    constexpr bool apply_on<Func>(uint8_t property_id, Func&& func) noexcept (...) requires(...);
    constexpr bool visit<Func>(Func&& func) <const> noexcept(...) requires(...);
};
```

##### Data Types

```c++
struct authority_path { std::string host, port, path; };
enum class qos_e : uint8_t { at_most_once, at_least_once, exactly_once };
enum class retain_e : uint8_t { no, yes };
enum class dup_e : uint8_t { no, yes };
enum class auth_step_e { client_initial, server_challenge, server_final };
enum class no_local_e : uint8_t { no, yes };
enum class retain_as_published_e : uint8_t { dont, retain };
enum class retain_handling_e : uint8_t { send, new_subscription_only, not_send };
struct subscribe_options { qus_e max_qos{exactly_once}; no_local_e no_local{yes};
    retain_as_published_e retain_as_published{retain}; retain_handling_e retain_handling{new_subscription_only}; };
struct subscribe_topic { std::string topic_filter; subscribe_options sub_opts; };

class connect_props : public prop::properties<session_expiry_interval_t, receive_maximum_t, maximum_packet_size_t, topic_alias_maximum_t,
    request_response_information_t, request_problem_information_t, user_property_t, authentication_method_t, authentication_data_t>{};
class connack_props : public prop::properties<session_expiry_interval_t, receive_maximum_t, maximum_qos_t, retain_available_t, maximum_packet_size_t,
    assigned_client_identifier_t, topic_alias_maximum_t, reason_string_t, user_property_t, wildcard_subscription_available_t subscription_identifier_available_t,
    shared_subscription_available_t, server_keep_alive_t, response_information_t, server_reference_t, authentication_method_t, authentication_data_t>{};
class publish_props : public prop::properties<payload_format_indicator_t, message_expiry_interval_t, content_type_t,
    response_topic_t, correlation_data_t, subscription_identifier_t, topic_alias_t, user_property_t>{};
class puback_props : public prop::properties<reason_string_t, user_property_t>{};
class pubcomp_props : public prop::properties<reason_string_t, user_property_t>{};
class pubrec_props : public prop::properties<reason_string_t, user_property_t>{};
class pubrel_props : public prop::properties<reason_string_t, user_property_t>{};
class subscribe_props : public prop::properties<subscription_identifier_t, user_property_t>{};
class suback_props : public prop::properties<reason_string_t, user_property_t>{};
class unsubscribe_props : public prop::properties<user_property_t>{};
class unsuback_props : public prop::properties<reason_string_t, user_property_t>{};
class disconnect_props : public prop::properties<session_expiry_interval_t, reason_string_t, user_property_t, server_reference_t>{};
class auth_props : public prop::properties<authentication_method_t, authentication_data_t, reason_string_t, user_property_t>{};
class will_props : public prop::properties<will_delay_interval_t, payload_format_indicator_t, message_expiry_interval_t,
    content_type_t, response_topic_t, correlation_data_t, user_property_t>{};

class will : public will_props {
    std::string _topic, _message; qos_e _qos; retain_e _retain;
public: ctor(); ctor(std::string topic, std::string message, qos_e qos=at_most_once, retain_e retain=no, <will_props props>);
    std::string_view topic() const; std::string_view message() const; constexpr qos_e qos() const; constexpr retain_e retain() const;
};
```

##### Logging

```c++
class noop_logger{};
constexpr bool has_at_resolve<T>; // detect T::at_resolve(error_code, std::string_view, std::string_view, const asio::ip::tcp::resolver::results_type&)
constexpr bool has_at_tcp_connect<T>; // detect T::at_tcp_connect(error_code, asio::ip::tcp::endpoint)
constexpr bool has_at_tls_handshake<T>; // detect T::at_tls_handshake(error_code, asio::ip::tcp::endpoint)
constexpr bool has_at_ws_handshake<T>; // detect T::at_ws_handshake(error_code, asio::ip::tcp::endpoint)
constexpr bool has_at_connack<T>; // detect T::at_connack(reason_code, bool, const connack_props&)
constexpr bool has_at_disconnect<T>; // detect T::at_disconnect(reason_code, const disconnect_props&)

enum class log_level : uint8_t { error=1, warning, info, debug };
class logger { constexpr static auto prefix = "[Boost.MQTT5]"; log_level _level;
public: ctor(log_level level=info);
    void at_resolve(error_code ec, std::string_view host, std::string_view port, const asio::ip::tcp::resolver::results_type& eps);
    void at_tcp_connect(error_code ec, asio::ip::tcp::endpoint ep);
    void at_tls_handshake(error_code ec, asio::ip::tcp::endpoint ep);
    void at_ws_handshake(error_code ec, asio::ip::tcp::endpoint ep);
    void at_connack(reason_code rc, bool session_present, const connack_props& ca_props);
    void at_disconnect(reason_code rc, const disconnect_props& dc_props);
};

class detail::log_invoke<LoggerType> { LoggerType _logger;
public: explicit ctor(LoggerType logger={});
    void at_XXX(...) { if constexpr (has_at_XXX) _logger.at_XXX(...); }
};
```

------
### Concepts

```c++
concept StreamType<S> = beast::AsyncStream<S> && requires (S s) {
    typename S::lowest_layer_type;
    requires(std::derived_from<S::lowest_layer_type, boost::asio::ip::tcp::socket>);
    {s.lowest_layer()}->convertible_to<S::lowest_layer_type&>;
    async_shutdown(s);
};

concept TlsContext<Ctx> = 
```

------
### MQTT5 Client

```c++
class mqtt_client<StreamType, TlsContext=std::monostate, LoggerType=noop_logger> {
    using stream_type = StreamType; using tls_context_type = TlsContext; using logger_type = LoggerType;
    using client_service_type = client_service<stream_type, tls_context_type, logger_type>; using impl_type = std::shared_ptr<client_service_type>;
    impl_type _impl;
public: using executor_type = stream_type::executor_type;
    struct rebind_executor<Executor> { using other = self<detail::rebind_executor<stream_type, Executor>::other, TlsContext, LoggerType>; };
    explicit ctor(const executor_type& ex, tls_context_type tls_context={}, logger_type logger={})
        : _impl{std::make_shared<client_service_type>(ex, std::move(tls_context), std::move(logger))}{}
    explicit ctor<ExecCtx>(ExecCtx& context, tls_context_type tls_context={}, logger_type logger={}) requires is_convertible_v<ExecCtx&, asio::execution_context&>
        : self{context.get_executor(), std::move(tls_context), std::move(logger)}{}
    ctor(self&&) noexcept = default; ~dtor() { if (_impl) _impl->cancel(); }
    self& operator=(self&& other) noecxept { _impl->cancel(); _impl=std::move(other._impl); return *this; }

    executor_type get_executor() const noexcept { return _impl->get_executor(); }
    decltype(auto) tls_context<Ctx>() requires !std::is_same_v<Ctx,std::monostate> { return _impl->tls_context(); }
    decltype(auto) async_run<Token=asio::default_completion_token<executor_type>::type>(Token&& token={})
    { return asio::async_initiate<Token,void(error_code)>(initiate_async_run(_impl), token); }
    void cancel() { auto impl=_impl; _impl = impl->dup(); impl->cancel(); }

    self& will(will will) { _impl->will(std::move(will)); return *this; }
    self& credentials(std::string client_id, std::string username="", std::string password="")
    { _impl->credentials(std::move(client_id), std::move(username), std::move(password)); return *this; }
    self& brokers(std::string hosts, uint16_t defualt_port=1883) { _impl->brokers(std::move(hosts), default_port); return *this; }
    self& authenticator<Auth>(Auth&& authenticator) { _impl->authenticator(std::forward<Auth>(authenticator)); return *this; }
    self& keep_alive(uint16_t seconds) { _impl->keep_alive(seconds); return *this; }
    self& connect_properties(connect_props pros) { _impl->connect_properties(std::move(props)); return *this; }
    self& connect_property<p>(std::integral_constant<prop::property_type, p> prop, value_type_t<p> value)
    { _impl->connect_property(prop, std::move(value)); return *this; }

    void re_authenticate() { re_auth_op{_impl}.perform(); }
    const auto& connack_property<p>(std::integral_constant<prop::property_type, p> prop) const { return _impl->connack_property(prop); }
    const connack_props connack_properties() const { return _impl->connack_properties(); }
    decltype(auto) async_publish<qos_type,Token=asio::default_completion_token<executor_type>::type>
        (std::string topic, std::string payload, retain_e retain, const publish_props& props, Token&& token={})
    { return asio::async_initiate<Token,on_publish_signature<qos_type>>(
        initiate_async_publish<qos_type>{_impl}, token, std::move(topic), std::move(payload), retain, props); }
    decltype(auto) async_subscribe<Token=asio::default_completion_token<executor_type>::type>
        (const std::vector<subscribe_topic>& topics, const subscribe_props& props, Token&& token={})
    { return asio::async_initiate<Token,void(error_code,std::vector<reason_code>,suback_props)>
        (initiate_async_subscribe{_impl}, token, topics, props); }
    decltype(auto) async_subscribe<Token=asio::default_completion_token<executor_type>::type>
        (const subscribe_topic& topic, const subscribe_props& props, Token&& token={})
    { return async_subscribe(std::vector{topic}, props, std::forward<Token>(token)); }
    decltype(auto) async_unsubscribe<Token=asio::default_completion_token<executor_type>::type>
        (const std::vector<std::string>& topics, const unsubscribe_props& props, Token&& token={})
    { return asio::async_initiate<Token,void(error_code,std::vector<reason_code>,unsuback_props)>
        (initiate_async_unsubscribe{_impl}, token, topics, props); }
    decltype(auto) async_unsubscribe<Token=asio::default_completion_token<executor_type>::type>
        (const std::string& topic, const unsubscribe_props& props, Token&& token={})
    { return async_unsubscribe(std::vector{topic}, props, std::forward<Token>(token)); }
    decltype(auto) async_receive<Token=asio::default_completion_token<executor_type>::type>(Token&& token={})
    { return _impl->async_channel_receive(std::forward<Token>(token)); }
    decltype(auto) async_disconnect<Token=asio::default_completion_token<executor_type>::type>
        (disconnect_rc_e reason_code, const disconnect_props& props, Token&& token={})
    { auto impl=_impl; _impl=impl.dup(); return _impl->async_terminal_disconnect({reason_code}, props, impl, std::forward<Token>(token)); }
    decltype(auto) async_disconnect<Token=asio::default_completion_token<executor_type>::type>(Token&& token={})
    { return async_disconnect(normal_disconnection, {}, std::forward<Token>(token)); }
};
```

------
### Dependency

#### Boost.ASIO

* `<boost/asio/**.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Beast

* `<boost/beast/**.hpp>`

#### Boost.Container

* `<boost/container/small_vector.hpp>`

#### Boost.Core

* `<boost/core/identity.hpp>`

#### Boost.Endian

* `<boost/endian/conversion.hpp>`

#### Boost.Random

* `<boost/random/linear_congruential.hpp>`
* `<boost/random/uniform_smallint.hpp>`

#### Boost.Range

* `<boost/range/iterator_range_core.hpp>`

#### Boost.SmartPtr

* `<boost/smart_ptr/allocate_unique.hpp>`

#### Boost.System

* `<boost/system/error_code.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

------
### Standard Facilities
